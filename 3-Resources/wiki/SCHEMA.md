---
version: 1.0
status: active
created: 2026-06-28
updated: 2026-06-28
---

# SCHEMA — LLM-Wiki 운영 규칙

이 vault의 `3-Resources/wiki/` 영역에 대한 단일 진실 공급원(single source
of truth). 이 문서는 개정 가능하다 — 수정할 때마다 `version`을 올리고 맨
아래 Growth log에 한 줄 추가한다. 조용한(silent) 수정 금지.

> 이 스타터는 Andrej Karpathy의 LLM-Wiki gist(2026-04)와 Tiago Forte의
> PARA 조직법을 결합한 패턴이다. 배경은 `concepts/llm-wiki-pattern.md` 참조.

---

## 1. 영역(Territoriality) — 가장 중요한 규칙

이 vault는 규칙이 다른 **두 영역**으로 나뉜다.

| 영역 | 규칙 | 주인 |
|------|------|------|
| `3-Resources/wiki/**` | 엄격한 SCHEMA 강제 | LLM이 관리 |
| 그 외 모든 PARA 폴더 | 느슨, 자유 편집 | 사람이 관리 |

LLM-Wiki 에이전트는 기본적으로 `3-Resources/wiki/` **안에서만** 동작한다.
사용자의 명시적 지시 없이 형제 위치(`0-Inbox/`, `1-Projects/`, `2-Areas/`,
`4-Archive/`, `3-Resources/` 루트)를 읽거나 쓰지 않는다.

경계를 넘는 작업(예: inbox 파일을 `wiki/raw/`로 라우팅)이 일어나는 순간,
**목적지의 규칙**이 적용된다.

---

## 2. 태그 분류 체계(Taxonomy)

상위 도메인 8개, 소문자-하이픈. 하위 태그는 슬래시 중첩
(`ai-ml/llm` — Obsidian 호환 nested tag).

```
ai-ml       → llm, agents, eval, alignment
software    → language, architecture, testing
infra       → cloud, k8s, db, observability
business    → strategy, ops, finance
science     → cs, physics, biology, neuroscience
design      → ui-ux, branding
philosophy  → ethics, epistemology
meta        → methodology, productivity, this-vault
```

> 이 8개는 **기본값**이다. 본인 관심사에 맞게 교체·확장하라. 단 아래 성장
> 규칙은 유지하기를 권한다.

**성장 규칙:** 새 태그는 처음 쓰기 **전에** 이 절에 추가한다. 새 하위 태그는
≥ 3개 페이지가 그 태그를 달 때 정당화된다. 1회용 태그는 노이즈 — 가장 가까운
기존 태그를 우선 사용. **상위 도메인은 9번째를 만들지 않는다.** 안 맞으면
하위 태그를 확장하라.

---

## 3. Frontmatter

### 3.1 Wiki 페이지 (entities, concepts, comparisons, queries)

```yaml
---
title: <페이지 제목>
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: entity | concept | comparison | query | summary
tags: [<상위/하위>, ...]
sources: [raw/articles/<file>.md, ...]
ai-first: true
confidence: high | medium | low | speculation   # 선택
contested: true                                   # 선택
contradictions: [<page-slug>, ...]                # 선택
---
```

필수: `title`, `created`, `updated`, `type`, `tags`, `sources`, `ai-first`.
`ai-first: true`는 이 페이지가 사람의 선형 독서가 아니라 **LLM 검색에
최적화**되어 있음을 선언한다.

### 3.2 Raw 소스 (`raw/articles/`, `raw/papers/`, `raw/transcripts/`)

```yaml
---
source_url: https://...
source_type: article | paper | transcript | video | image | book
ingested: YYYY-MM-DD
sha256: <본문 내용의 hex 해시>
source_published: YYYY-MM-DD            # 선택 — 원문 발행일을 알 때
language: ko | en | ja | zh | mixed | auto   # 원문 언어
---
```

`sha256`는 drift 탐지용 — 해시가 바뀐 경우에만 재-ingest. raw 본문은
**ingest 시점에 번역하지 않는다.** 원문 언어 그대로 보존한다. 번역·요약은
항상 `entities|concepts|queries/`의 파생 산출물로 만든다.

`book` 소스는 정식 웹 출처가 없을 수 있다 — `source_url`은 서지 페이지(출판사/
서점)를 가리키고, raw 본문은 보통 책 본문이 아니라 소유자의 독서 노트다.

---

## 4. "For future Claude" 프리앰블

모든 wiki 페이지는 `## For future Claude` 섹션으로 시작한다.

```markdown
## For future Claude

[최대 4문장. 이 페이지가 무엇인지. 왜 이 vault에 존재하는지. 미래의
에이전트가 반드시 가져가야 할 단 하나의 핵심 사실. 첫 문장이 가장 무겁다 —
에이전트가 첫 문장만 읽고 멈출 수 있으므로, 첫 문장만으로 관련성이
전달돼야 한다.]
```

목표: 미래의 에이전트가 본문을 읽지 않고도 ~10초 안에 관련성을 판단. 본문이
"무엇"이라면, 프리앰블은 "왜 이게 이 vault에서 중요한가"이다.

---

## 5. 인용 & 출처(Provenance)

세 계층을 함께 쓴다.

1. **Frontmatter `sources:`** — 페이지 전체의 최상위 출처. 페이지가 파생된
   모든 raw 파일을 나열한다. (필수)

2. **인라인 인용** — 날짜가 있거나 논쟁적이거나 confidence가 다른 특정
   주장에:
   ```
   Karpathy가 2026-04에 LLM-Wiki 패턴을 제안했다
   [Source: [[raw/articles/2026-04-04-karpathy-llm-wiki.md]] | 2026-04-04 | confidence: high].
   ```

3. **합성 마커(synthesis marker)** — ≥ 3개 소스를 합친 문단에:
   ```
   세 독립 출처가 모두 rewrite-on-ingest 패턴을 기술한다.
   ^[raw/articles/a.md, raw/articles/b.md, raw/transcripts/c.md]
   ```

원칙: `sources` frontmatter는 항상 필수. 단일 주장의 검증 가능성이 중요할
때 인라인 인용. 서사가 여러 raw에 기댈 때 합성 마커.

---

## 6. Brain-First Knowledge Protocol

답하기 전에 에이전트는 vault 페이지를 먼저 읽을지 결정한다. 4계층을 순서대로
적용한다.

### Layer 1 — 필수 트리거 (vault 읽기 REQUIRED)

사용자 메시지에 다음이 포함되면:

**한국어:** `우리`, `내`, `저희`, `회사`, `팀`, `지난번`, `이전`, `전에`,
`예전`, `작년`, `결정`, `방향`, `전략`, `왜 그렇게`, `정해졌`, `정한`

**English:** `we`, `our`, `my`, `the team`, `the company`, `last`,
`previously`, `before`, `decided`, `strategy`

**추가:** vault에 등록된 고유명사(= `entities/`, `concepts/`, `index.md`의
페이지 제목).

**행동:**
1. `index.md`에서 후보 페이지를 검색.
2. 후보 1–3개를 읽는다 (하드 캡: 5개, 토큰 비용 제한).
3. 응답에 참조한 페이지를 인용: "`[[entity-X]]`에 따르면…".

### Layer 2 — 가벼운 스캔 (에이전트 판단)

Layer 1이 발동하지 않았고 응답이 trivial하지 않다면, `index.md`를 한 번
훑는다. 헤딩이 사용자 키워드와 맞으면 그 페이지를 전체 읽기로 확장. 소프트 룰.

### Layer 3 — 스킵 도메인 (트리거가 맞아도 vault 읽기 안 함)

- 일반 코딩/디버깅
- 외부 뉴스/시사 (대신 웹 검색)
- 사소한 계산/번역
- 잡담/인사

Layer 1 false positive 필터. 예: "우리 React 쓸까?"는 `우리`에 걸리지만 일반
기술 질문이라 vault 읽기 없음.

### Layer 4 — 사용자 오버라이드

- `?wiki` 또는 `/recall` → 트리거 무관하게 Layer 1 강제.
- `?nowiki` 또는 `/quick` → 모든 wiki 읽기 생략, latency 우선.

### 범위 제한

이 프로토콜은 `wiki/`에만 적용된다. `2-Areas/`·`1-Projects/` 검색은 사용자의
명시적 지시가 있어야 한다.

---

## 7. 갱신 / 재작성 정책

새 소스가 기존 페이지와 충돌하는 정보를 담으면 **Moderate 정책**을 따른다.

1. **날짜 비교.** 더 최근 날짜의 주장이 일반적으로 우선.
2. **기존 페이지에서 주장을 재작성.**
3. **옛 주장은 페이지 맨 아래 `## History` 섹션에 보존:**
   ```
   ## History
   - 2026-04-12 → 2026-05-08: 가격 $X → $Y 변경
     (source: [[raw/articles/vendor-update-2026-05.md]])
   ```
4. **Frontmatter 표시:** 충돌이 여러 페이지에 걸치면
   `contradictions: [<other-page-slug>]`.
5. **`log.md`에 감사 기록** (갱신 1건당 한 줄).

새 소스가 **보충**(충돌 아님)이면: 새 섹션/불릿으로 추가만. `## History`
불필요.

충돌이 **진짜이고 날짜가 모호**하면: 두 입장을 나란히 보존, `contested: true`
표시, log에 기록.

**하드 룰:** 주장을 절대 조용히 삭제·재작성하지 않는다. 페이지는 현재의
진실이고, `log.md`는 감사 흔적이다.

---

## 8. Ingest 임계값 & 페이지 관리

### 새 페이지를 만드는 시점

- 어떤 entity/concept가 **≥ 2개의 ingest된 소스**에 등장, 또는
- **하나의 소스의 핵심 주제**일 때 (그것에 관한 논문/기사).

만들지 않는 경우: 스쳐 지나가는 언급, 사소한 구현 디테일, §2 분류 밖 주제.

### 파일명

`소문자-하이픈만.md`. 공백·언더스코어·대문자 금지.

- ✅ `transformer-architecture.md`, `oh-my-hotel.md`
- ❌ `Transformer Architecture.md`, `oh_my_hotel.md`

**Raw 파일**은 날짜 접두사를 추가: `YYYY-MM-DD-slug.md` (원문 발행일).
탐지 불가 시 `unknown-` 사용.

- ✅ `raw/articles/2026-04-04-karpathy-llm-wiki.md`
- ✅ `raw/papers/unknown-attention-is-all-you-need.md`

### 페이지 크기

- 목표: ≤ 200줄.
- 200–300: 분할 후보 검토 (lint warning).
- > 300: 분할 필수 (lint failure). 섹션을 별도 페이지로 추출, `[[wikilink]]`로 연결.

### 상호 참조

모든 페이지는 **다른 wiki 페이지로 향하는 `[[wikilink]]` ≥ 2개**를 포함한다
(`entities/`, `concepts/`, `comparisons/`, `queries/`). `raw/*` 링크는 카운트
안 됨 — 그건 `sources:` frontmatter로 따로 추적.

**부트스트랩 예외:** 전체 wiki 페이지 수가 < 10일 때 최소값은 ≥ 1로 내려간다.
페이지가 10개를 넘으면 예외는 자동 해제.

### 벌크 ingest 순서

여러 소스를 한 번에 처리할 때: 전부 읽기 → 소스들에 걸친 entity/concept 식별
→ 기존 페이지 한 번 확인 → 한 배치로 생성/갱신 → `index.md` 한 번 갱신 →
`log.md`에 배치 기록 한 줄.

---

## 9. Lint 체크리스트

`/lint` (또는 수동 점검)으로:

| # | 검사 | 심각도 |
|---|------|--------|
| 1 | 고아 페이지 (inbound `[[링크]]` 없음) | warning |
| 2 | 깨진 wikilink (`[[Target]]`인데 파일 없음) | critical |
| 3 | index 완전성 (모든 페이지가 `index.md`에) | critical |
| 4 | Frontmatter 검증 (필수 필드, 태그 ∈ §2) | critical |
| 5 | 오래된 내용 (`updated` > 90일 vs 더 새 소스) | warning |
| 6 | 충돌 (`contradictions:` 또는 `contested: true`) | info |
| 7 | confidence 감사 (`low` 또는 단일 소스에 없음) | warning |
| 8 | 소스 drift (`raw/*` 재해시, `sha256` 비교) | warning |
| 9 | 페이지 크기 — wiki 콘텐츠만 (`entities/concepts/comparisons/queries/`): > 200 분할후보, > 300 필수. SCHEMA/index/log/raw 면제. | warning / critical |
| 10 | 태그 감사 (모든 태그 ∈ §2) | warning |
| 11 | log 회전 (`log.md` > 500 entries → `log-YYYY.md`) | info |

---

## 10. PARA ↔ wiki 라우팅

새 콘텐츠의 기본 위치:

| 콘텐츠 유형 | 위치 | 비고 |
|---|---|---|
| 회의록·결정 로그 (작업) | `2-Areas/<domain>/` | wiki 아님 |
| 회의에서 나온 재사용 가능 인사이트/프레임워크 | `wiki/concepts/` | + raw transcript를 `wiki/raw/transcripts/`에 |
| 외부 인물/조직/도구 | `wiki/entities/` | 저자, 벤더, 공인 |
| 임시 노트 (≤ 30일 TTL) | `0-Inbox/` | |

### 모호할 때

단일 산출물을 넘어 **재사용 가능한 개념적 가치**가 있을 때만 wiki로. 아니면
PARA Areas/Projects가 기본.

### 프라이버시 경계

- 내부 동료의 개인정보(PII)는 wiki에 절대 넣지 않는다 (entity stub로도). 내부
  인물은 `2-Areas/people/`로.
- 외부 공인(저자, 벤더, 컨퍼런스 연사)은 wiki 대상.
- 내부/외부가 불확실하면 → 사용자에게 묻는다.

---

## Growth log

- **2026-06-28 v1.0** — eenaie-second-brain 스타터 초기 작성. second-brain-vault
  SCHEMA v1.4에서 개인/Hermes 결합부(Korean Ingest Policy, Jira/cron 특수 케이스,
  4-hat taxonomy)를 제거하고 generic·한국어 골격만 추출. 10개 절 +
  taxonomy 기본값 8도메인.
