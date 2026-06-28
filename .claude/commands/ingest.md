---
description: wiki/raw/ backlog를 durable entity/concept/comparison/query 페이지로 승격
argument-hint: [raw slug 또는 all — 비우면 backlog 전체]
allowed-tools: Read, Write, Edit, Bash(ls:*), Bash(date:*), Glob, Grep
---

당신은 이 vault의 **ingest** 워커다. `wiki/raw/`의 보존된 소스를 읽고, 임계값을
만족하는 지식을 `entities/`, `concepts/`, `comparisons/`, `queries/`의 durable
markdown 페이지로 승격한다. URL을 fetch하지 않는다(그건 `/capture`). 운영 규칙은
`3-Resources/wiki/SCHEMA.md`가 단일 진실 공급원이다.

## 대상

$ARGUMENTS  (비어 있으면 `raw/` 전체 backlog를 대상으로)

## 시작 전 필수

1. `3-Resources/wiki/SCHEMA.md`를 읽는다 — 특히 §3.1(페이지 frontmatter),
   §4(For future Claude 프리앰블), §5(인용), §7(갱신 정책), §8(임계값·파일명·
   상호참조).
2. `3-Resources/wiki/index.md`를 읽어 현재 페이지 목록을 파악.

## 절차 (SCHEMA §8 "벌크 ingest 순서")

1. **전부 읽기.** 대상 raw 파일을 모두 읽는다.
2. **식별.** 소스들에 걸친 entity/concept를 뽑는다. 새 페이지는
   **≥ 2개 소스에 등장**하거나 **한 소스의 핵심 주제**일 때만 만든다(§8).
   스쳐가는 언급·사소한 디테일·§2 분류 밖 주제는 만들지 않는다.
3. **기존 페이지 확인.** index에서 후보가 이미 있는지 한 번에 확인.
4. **생성/갱신 (한 배치):**
   - 신규 페이지: §3.1 frontmatter + §4 프리앰블 + 본문. 모든 주장에 §5 출처
     (frontmatter `sources:` 필수, 필요 시 인라인 인용). 파일명은 §8
     `소문자-하이픈.md`. 다른 wiki 페이지로 향하는 `[[wikilink]]` **≥ 2개**
     (전체 페이지 < 10이면 부트스트랩 예외로 ≥ 1).
   - 기존 페이지 갱신: §7 Moderate 정책. 충돌이면 재작성 + `## History` 보존,
     절대 조용히 삭제하지 않는다. 보충이면 섹션 추가만.
   - 페이지 ≤ 200줄 목표 (§8 크기 규칙).
5. **index.md 갱신** (한 번). 추가/변경된 페이지를 카탈로그에 반영, 통계 갱신.
6. **log.md에 기록** (배치당 한 줄, append-only). 형식:
   `## [<오늘> HH:MM] ingest | <page-slug>.md | <raw-slug> | <요약>`
   (오늘 날짜는 `date +%F`).
7. **raw는 immutable.** 본문/frontmatter를 수정하지 않는다(§3.2).

## 프라이버시 (SCHEMA §10)

내부 동료 PII는 wiki에 절대 넣지 않는다. 외부 공인만 `entities/` 대상.
내부/외부가 불확실하면 사용자에게 묻는다.

## 보고

생성·갱신된 페이지 목록, 만들지 않은 항목과 이유(임계값 미달 등), index/log
갱신 여부.
