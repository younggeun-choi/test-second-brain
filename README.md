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
├── .claude/commands/  slash command 5종
└── CLAUDE.md        Claude Code 작동 규칙
```

PARA 흐름: `0-Inbox/` → 분류 → `1-Projects/` 또는 `2-Areas/` → 완료 시 `4-Archive/`.

## Slash Commands

Claude Code에서 cwd를 이 vault로 두고 바로 쓸 수 있다.

| 커맨드 | 하는 일 |
|--------|---------|
| `/capture <URL\|텍스트>` | URL은 `wiki/raw/`에 원문 보존(sha256 dedup), 텍스트는 `2-Areas/braindumps/`에. **ingest 아님.** |
| `/ingest [slug\|all]` | `raw/` backlog를 SCHEMA 임계값(≥2 소스 등)에 맞춰 durable wiki 페이지로 승격. index·log 갱신. |
| `/query <질문>` | Brain-First 프로토콜로 wiki를 먼저 읽고 인용과 함께 답변. `?wiki`/`?nowiki` 오버라이드. |
| `/lint [범위]` | 링크·frontmatter·index/log·raw drift·페이지 크기 점검 (SCHEMA §9). |
| `/save <설명>` | 세션의 결정·산출물을 PARA/wiki 규칙에 맞는 위치에 보존. |

워크플로의 핵심은 **capture와 ingest의 분리**다. capture는 원문 보존까지만,
durable 지식 승격은 ingest가 별도로 한다.

## 시작하기

1. (선택) 이 repo를 본인 vault 이름으로 fork/복제.
2. Claude Code를 vault 루트에서 연다 (`CLAUDE.md`가 자동 로드됨).
3. `3-Resources/wiki/SCHEMA.md` §2 태그 분류 체계를 본인 관심사로 교체.
4. `/capture <URL>`로 소스를 모으고, 쌓이면 `/ingest`로 승격, `/query`로 회상.
5. (선택) [Obsidian](https://obsidian.md/)으로 같은 폴더를 열면 `[[wikilink]]`
   그래프를 시각적으로 탐색할 수 있다.

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
