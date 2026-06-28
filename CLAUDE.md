# eenaie-second-brain — Claude Code 작동 규칙

cwd가 이 vault 안일 때 Claude Code 세션이 따르는 가이드.

이 파일은 **IDE 어시스턴트(Claude Code)용** 작동 매뉴얼이다. wiki 작업의 실제
규칙은 `3-Resources/wiki/SCHEMA.md`에 있다. 역할이 다르므로 SCHEMA 내용을 여기
중복하지 않는다 — SCHEMA를 **가리킬** 뿐이다.

---

## 1. Vault 정체성

- **무엇인가:** PARA 조직법 + LLM-Wiki 패턴을 결합한 개인 지식 vault.
- **두 영감:**
  - [PARA by Tiago Forte](https://fortelabs.com/blog/para/) — 디렉토리 조직법.
  - [Andrej Karpathy의 LLM-Wiki gist (2026-04)](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
    — RAG가 매번 재검색하는 대신, LLM이 markdown wiki를 점진적으로 컴파일.
- **핵심 차별점:** 단순 노트 모음이 아니라 **자동 ingest + 수동 편집이
  하이브리드로 돌아가는 살아있는 vault.** 두 영역이 다른 규칙으로 공존한다(§3).

## 2. 디렉토리 맵

```
eenaie-second-brain/
├── 0-Inbox/         라우팅 대기 (≤30일 TTL)
├── 1-Projects/      마감 있는 활성 작업
├── 2-Areas/         지속적 책임 영역
├── 3-Resources/     참고 자료
│   ├── media/screenshots/   비-개념 시각 참고
│   └── wiki/                ← LLM-Wiki (SCHEMA.md 적용)
│       ├── SCHEMA.md        운영 규칙
│       ├── index.md         durable 페이지 카탈로그
│       ├── log.md           append-only 감사 로그
│       ├── raw/             원문 보존 계층 (articles/papers/transcripts)
│       ├── entities/        사람·회사·제품·모델
│       ├── concepts/        개념·프레임워크·방법론
│       ├── comparisons/     비교 분석
│       └── queries/         저장할 가치가 있는 질의 결과
├── 4-Archive/       완료/폐기
├── Templates/       PARA 노트 스캐폴드 (순수 markdown)
├── .claude/commands/  slash command: capture·ingest·query·lint·save
└── CLAUDE.md        (이 파일)
```

PARA 흐름: 신규 항목은 `0-Inbox/`로 들어와 분류 후 `1-Projects/` 또는
`2-Areas/`로 이동, 완료되면 `4-Archive/`로 보관.

## 3. 영역(Territoriality) — 가장 중요한 규칙

vault는 규칙이 다른 **두 영역**으로 나뉜다.

| 영역 | 규칙 | 주인 |
|------|------|------|
| `3-Resources/wiki/**` | 엄격한 스펙 (SCHEMA.md) | LLM이 관리 |
| 그 외 모든 PARA 폴더 | 느슨, 사용자 편집 | 사람이 관리 |

콘텐츠를 편집·생성하기 전에:
1. 대상 파일이 어느 영역인지 식별.
2. 그 영역의 규칙 적용.
3. 경계를 넘으면(예: inbox 파일을 `wiki/raw/`로 라우팅) **목적지의 규칙**이
   목적지에서 적용된다.

## 4. 파일 편집 규칙

| 경로 | 규칙 |
|------|------|
| `wiki/SCHEMA.md` | 수정 시마다 `version` 올리고 Growth log 추가. 조용한 수정 금지. |
| `wiki/log.md` | Append-only. 과거 entry 수정/삭제 금지. 500개 초과 시 `log-YYYY.md`로 회전. |
| `wiki/index.md` | 페이지 추가/삭제 시마다 재빌드. 항상 현재 상태 반영. |
| `wiki/raw/**` | Immutable. ingest 후 내용 수정 금지. sha256 drift = lint 실패. |
| `wiki/{entities,concepts,comparisons,queries}/` | SCHEMA §7 정책대로 갱신: `## History` 섹션 + `log.md` 감사 한 줄. 주장을 조용히 삭제하지 않는다. |
| `Templates/*.md` | 라우팅 폴더의 미래 노트에 영향. 신중히 수정. |
| `1-Projects/*.md` | 자유 편집. 권장 파일명 `YYYY-MM <title>.md`. |
| 그 외 PARA 폴더 | 사용자 편집 영역. 스킬은 읽되 조용히 재작성하지 않는다. |

## 5. Slash Commands

`.claude/commands/`의 5개 워크플로 동사. SCHEMA를 운영 스펙으로 참조한다.

| 커맨드 | 역할 | 방향 |
|--------|------|------|
| `/capture` | URL/텍스트 → `wiki/raw/` 또는 `2-Areas/` 보존 | 입력 캡처 (ingest 아님) |
| `/ingest` | `raw/` backlog → durable wiki 페이지 승격 | 지식 통합 |
| `/query` | Brain-First wiki 우선 질의 응답 | 회상 |
| `/lint` | wiki health check (SCHEMA §9) | 점검 |
| `/save` | 명시적으로 요청된 세션 산출물 저장 | 보존 |

**capture는 raw/braindump 저장에서 끝난다.** durable wiki 승격은 별개 단계이고
`/ingest`가 담당한다 — 이 분리가 패턴의 핵심이다.

## 6. Git 컨벤션

- vault 루트의 단일 git repo. 의미 있는 변경마다 커밋.
- 커밋 메시지: `<area>: <명령형 구절>` (area ∈ `init / schema / ingest /
  config / capture / lint`).
  - 예: `schema: SCHEMA.md v1.0 → v1.1 — taxonomy 확장`
  - 예: `ingest: Karpathy LLM-Wiki gist — 1 entity + 1 concept`
- 커밋 단위: 하나의 논리적 행동 (한 ingest, 한 schema bump, 한 config 변경).

## 7. Don't List

명시적 경계. 각 항목은 알려진 실패 모드를 막는다.

- ❌ **내부 동료 PII를 `wiki/entities/`에 넣지 않는다.** 외부 공인(저자,
  벤더)은 OK. 내부 인물은 `2-Areas/people/`로. (SCHEMA §10 taxonomy 경계)
- ❌ **AI auto-link 플러그인을 켜지 않는다** (Smart Connections, Copilot for
  Obsidian 등). 비-SCHEMA frontmatter를 써서 taxonomy를 깬다.
- ❌ **SCHEMA §9 lint 없이 `wiki/` 파일을 일괄 rename 하지 않는다.** wikilink가
  조용히 깨진다.
- ❌ **ingest 후 `wiki/raw/**`를 수정하지 않는다.** sha256 drift로 lint 실패가
  영구화된다. 새로 fetch해서 페이지를 다시 derive하라.
- ❌ **첫 capture에서 braindump를 wiki 페이지로 자동 분류하지 않는다.** 단일
  소스 콘텐츠는 ≥ 2 소스 임계값(SCHEMA §8)을 만족하기 전까지 PARA에 머문다.

## 8. 의심스러울 때 (읽기 순서)

1. **`3-Resources/wiki/SCHEMA.md`** — wiki를 건드리는 작업이면.
2. **호출한 slash command의 본문** — 커맨드가 트리거했으면.
3. **사용자에게 묻는다** — 위로 해소 안 되면.

파괴적 행동을 절대 조용히 추측하지 않는다. 삭제/재작성 전에 append·comment·질문.
