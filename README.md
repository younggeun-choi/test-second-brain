# eenaie-second-brain

> **PARA + LLM-Wiki를 Claude Code에서 바로 쓸 수 있는 second-brain 스타터.**
> [PARA](https://fortelabs.com/blog/para/) 조직법과
> [LLM-Wiki 패턴](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)을
> 결합한 개인 지식 vault의 최소 골격.

이 repo는 운영 중인 personal vault에서 **재사용 가능한 뼈대만** 추출한
것이다. 개인 식별 정보, 회사·Jira·cron·Discord 연동, 특정 capture 워커(영상
전사 등)는 모두 제거했다. fork해서 본인 vault로 키우면 된다.

---

## 핵심 아이디어

일반적인 RAG가 질문할 때마다 문서를 다시 검색·요약한다면, **LLM-Wiki는 소스를
한 번 `raw/`에 보존한 뒤 중요한 지식을 durable markdown 페이지로 점진적으로
컴파일**한다. query 대상은 raw가 아니라 이미 합성·교차참조된 wiki다. 지식이
복리로 누적된다.

여기에 PARA를 결합해, 한 repo 안에 **규칙이 다른 두 영역**을 공존시킨다.

| 영역 | 규칙 | 주인 |
|------|------|------|
| `3-Resources/wiki/**` | 엄격한 SCHEMA 강제 | LLM이 관리 |
| 그 외 모든 PARA 폴더 | 느슨, 자유 편집 | 사람이 관리 |

이 **영역 분리(Territoriality)** 가 이 패턴의 가장 중요한 규칙이다
(`3-Resources/wiki/SCHEMA.md` §1).

## 디렉토리

```
eenaie-second-brain/
├── 0-Inbox/         라우팅 대기 (≤30일 TTL)
├── 1-Projects/      마감 있는 활성 작업
├── 2-Areas/         지속적 책임 영역
├── 3-Resources/
│   ├── media/screenshots/
│   └── wiki/                ← LLM-Wiki (SCHEMA.md 적용)
│       ├── SCHEMA.md        운영 규칙 (단일 진실 공급원)
│       ├── index.md         durable 페이지 카탈로그
│       ├── log.md           append-only 감사 로그
│       ├── raw/             원문 보존 (articles/papers/transcripts)
│       ├── entities/        사람·회사·제품·모델
│       ├── concepts/        개념·프레임워크·방법론
│       ├── comparisons/     비교 분석
│       └── queries/         저장할 가치가 있는 질의 결과
├── 4-Archive/       완료/폐기
├── Templates/       PARA 노트 스캐폴드 (순수 markdown)
├── .claude/commands/  slash command 6종
└── CLAUDE.md        Claude Code 작동 규칙
```

PARA 흐름: `0-Inbox/` → 분류 → `1-Projects/` 또는 `2-Areas/` → 완료 시 `4-Archive/`.

## Slash Commands

Claude Code에서 cwd를 이 vault로 두고 바로 쓸 수 있다.

| 커맨드 | 하는 일 |
|--------|---------|
| `/capture <URL>` | 외부 URL/파일을 `wiki/raw/`에 원문 보존(sha256 dedup). **ingest 아님.** |
| `/braindump <생각>` | 내 생각·메모를 도메인 분류·테마·전략 인사이트·action item으로 구조화해 `2-Areas/braindumps/` 등에 저장. |
| `/ingest [slug\|all]` | `raw/` backlog를 SCHEMA 임계값(≥2 소스 등)에 맞춰 durable wiki 페이지로 승격. index·log 갱신. |
| `/query <질문>` | Brain-First 프로토콜로 wiki를 먼저 읽고 인용과 함께 답변. `?wiki`/`?nowiki` 오버라이드. |
| `/lint [범위]` | 링크·frontmatter·index/log·raw drift·페이지 크기 점검 (SCHEMA §9). |
| `/save <설명>` | 세션의 결정·산출물을 PARA/wiki 규칙에 맞는 위치에 보존. |

입력은 두 갈래다 — 외부 소스(URL/파일)는 `/capture`, 내 생각·메모는 `/braindump`.
워크플로의 핵심은 **포착(capture·braindump)과 지식화(ingest)의 분리**다. 포착은
원문 보존까지만, durable 지식 승격은 ingest가 별도로 한다.

## 시작하기

1. (선택) 이 repo를 본인 vault 이름으로 fork/복제.
2. Claude Code를 vault 루트에서 연다 (`CLAUDE.md`가 자동 로드됨).
3. `3-Resources/wiki/SCHEMA.md` §2 태그 분류 체계를 본인 관심사로 교체.
4. 아래 "사용법" 루프를 돈다 — `/capture` → `/ingest` → `/query`.
5. (선택) [Obsidian](https://obsidian.md/)으로 같은 폴더를 열면 `[[wikilink]]`
   그래프를 시각적으로 탐색할 수 있다.

## 사용법 — 전체 흐름 예시

핵심 멘탈 모델: **모으는 단계(capture)와 지식으로 만드는 단계(ingest)는
분리되어 있다.** 무엇이든 일단 `raw/`에 쌓아두고, 충분히 모였을 때 골라서
wiki 페이지로 승격한다. RAG처럼 매번 재검색하는 게 아니라, 한 번 컴파일된
wiki를 질의한다.

### 1단계 — 소스를 모은다 (`/capture`)

```
/capture https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
```

→ `3-Resources/wiki/raw/articles/2026-04-04-karpathy-llm-wiki.md` 생성.
SCHEMA §3.2 frontmatter(`source_url`, `sha256`, `language` 등)가 붙고, 본문은
**원문 언어 그대로** 보존된다. 같은 sha256이 이미 있으면 "이미 캡처됨"으로
끝난다(dedup). 이 단계는 아직 wiki 지식이 **아니다.**

생각·메모는 URL이 아니라 `/braindump`로 던진다 (도메인 분류·테마·인사이트까지
입혀준다):

```
/braindump LLM-Wiki가 RAG보다 나은 핵심은 비용을 query에서 ingest로 옮기는 것
```

→ `2-Areas/braindumps/personal/braindump-2026-06-28-HHMM-...md`. 도메인 분류,
주요 테마, action item이 함께 정리된다. 단일 생각은 자동으로 wiki 페이지가
되지 않고(SCHEMA §8 ≥2 소스 임계값), 재사용 가능한 개념이면 `wiki_bridge_status:
deferred`로 표시만 해둔다 — 나중에 외부 소스가 같은 개념을 뒷받침하면 `/ingest`가
승격한다.

### 2단계 — 지식으로 승격한다 (`/ingest`)

raw가 몇 개 쌓이면:

```
/ingest
```

→ `raw/` backlog를 전부 읽고, **≥ 2개 소스에 등장**하거나 **한 소스의 핵심
주제**인 것만 durable 페이지로 만든다. 예: `entities/andrej-karpathy.md`(외부
인물) 생성 + `concepts/llm-wiki-pattern.md` 보강. 그다음 `index.md`(카탈로그)와
`log.md`(감사 로그)를 자동 갱신한다. 임계값에 못 미친 소스는 "왜 안 만들었는지"
보고만 하고 raw에 남겨둔다.

> raw는 immutable이다. ingest는 raw를 읽기만 하고, 지식은 항상 별도 페이지로
> 파생한다(SCHEMA §3.2).

### 3단계 — 회상한다 (`/query`)

```
/query 우리가 LLM-Wiki 패턴을 왜 골랐지?
```

→ Brain-First 프로토콜(SCHEMA §6)이 `우리/왜` 트리거를 잡아 `index.md`를
검색하고 `concepts/llm-wiki-pattern.md`를 읽은 뒤, **출처를 인용하며** 답한다:
"`[[llm-wiki-pattern]]`에 따르면 비용을 query→ingest로 옮겨 지식이 복리로
누적되기 때문…". 빠른 답만 원하면 `?nowiki`, wiki를 강제로 읽히려면 `?wiki`.

### 4단계 — 점검·보존 (`/lint`, `/save`)

```
/lint                       # 깨진 링크·frontmatter·고아 페이지·raw drift 점검
/save 비교 분석은 queries/ 대신 comparisons/ 에 두기로 결정   # 결정을 적절한 위치에 보존
```

### 한눈에 보는 루프

```
  웹 URL ───/capture───▶ raw/ (원문, 불변) ──┐
                                              ├──/ingest──▶ entities·concepts·comparisons·queries/ (durable)
  생각 ──/braindump──▶ 2-Areas/braindumps/ ──┘                 │           ▲
                                                     /query ◀──┘           └── /lint (건강검진) · /save (결정 보존)
```

## 일상 리듬 (권장)

- **수시:** 흥미로운 URL은 `/capture`, 떠오른 생각은 `/braindump`. 즉시, 마찰 0.
- **주 1–2회:** `/ingest`로 backlog를 비우며 지식을 컴파일.
- **필요할 때:** `/query`로 과거 결정·전략·등록 개념을 회상.
- **월 1회:** `/lint`로 링크·frontmatter·drift 점검.
- 작업·프로젝트 메모는 wiki가 아니라 `1-Projects/`·`2-Areas/`에 직접 쓴다
  (영역 분리 — 사람이 관리하는 영역).

## 자주 묻는 것

- **capture와 ingest를 왜 나누나?** 모으는 건 마찰이 0이어야 자주 하게 되고,
  지식 승격은 신중해야 품질이 유지된다. 둘을 묶으면 둘 다 망가진다.
- **그냥 wiki에 바로 쓰면 안 되나?** 가능하지만 그러면 출처·중복·모순 관리가
  깨진다. raw를 거치면 sha256 dedup·provenance·drift 탐지가 공짜로 따라온다.
- **Obsidian이 꼭 필요한가?** 아니다. 전부 순수 markdown이라 Claude Code만으로
  완결된다. Obsidian은 `[[wikilink]]` 그래프 시각화용 선택 사항.
- **태그 8개가 내 관심사와 안 맞는다.** SCHEMA §2를 교체하라 — 기본값일 뿐이다.

## 더 읽기

| 문서 | 역할 |
|------|------|
| `CLAUDE.md` | Claude Code 작동 규칙 — territoriality, 파일 편집 규칙, Don't-list |
| `3-Resources/wiki/SCHEMA.md` | wiki 운영 spec — frontmatter, 분류, ingest 정책, lint (v1.0) |
| `3-Resources/wiki/concepts/llm-wiki-pattern.md` | 이 패턴 자체의 자기참조 설명 |

## 영감

- [Andrej Karpathy의 LLM-Wiki gist (2026-04)](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — 핵심 패턴.
- [PARA by Tiago Forte](https://fortelabs.com/blog/para/) — 디렉토리 조직법.

## License

- **패턴·디렉토리 구조·SCHEMA·slash command** — 자유롭게 차용·fork 가능.
- **개별 노트 콘텐츠** — 본인이 추가하는 노트의 저작권은 본인에게.
