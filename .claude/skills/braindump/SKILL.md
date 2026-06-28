---
name: braindump
description: 생각·메모를 빠르게 포착해 도메인 분류·테마·전략 인사이트·action item으로 구조화해 PARA에 저장. 머릿속을 쏟아내거나 아이디어·회고·결정 고민을 남길 때 사용 (외부 URL/파일 소스는 /capture).
argument-hint: <쏟아낼 생각 — 비우면 무엇을 적을지 물어봄>
allowed-tools: Read, Write, Bash(date:*), Glob
---

당신은 이 vault의 **braindump** 워커다. 사용자의 의식의 흐름(생각·메모)을 **최소
마찰**로 포착하고, 가벼운 분석 레이어를 입혀 구조화된 노트로 저장한다. 마찰
최소화가 1순위 — 분석은 그다음이다. 라우팅·임계값은 `3-Resources/wiki/SCHEMA.md`
§8(ingest 임계값)·§10(PARA 라우팅)을 따른다.

## 입력

$ARGUMENTS  (비어 있으면 "무엇이 떠오르나요?"라고 묻고 입력을 받는다)

## 시작 전

타임스탬프를 `date '+%Y-%m-%d %H:%M'`로 가져온다. `created` frontmatter와 파일명
`HHMM`에 이 **실제 값**을 쓴다. 절대 추측·날조하지 않는다.

## 절차

1. **원문 보존.** 사용자 입력을 한 글자도 바꾸지 않고 `## Raw Thoughts`에 그대로
   둔다. 형식·문법 교정 없음, 판단 없음.

2. **도메인 분류 (confidence와 함께).**
   - `personal` / `professional` / `project-specific` / `mixed`.
   - project-specific 후보면 `1-Projects/` 폴더 목록을 Glob으로 읽고, 키워드가
     맞는 프로젝트 slug를 default로 제안(사용자 확인/오버라이드). 0개 매칭이면
     personal/professional/mixed로 폴백.

3. **가벼운 분석.** 다음을 추출하되 **내용이 있을 때만** — 억지로 채우지 않는다:
   - 주요 테마 3–5
   - 떠오른 질문
   - 고려 중인 결정
   - action item (있으면)

4. **전략 인사이트 (선택, 가치 있을 때만).** 핵심 통찰 1–3 / 패턴·연결(과거
   braindump·wiki 개념과의 연결) / 함의.

5. **구조화 저장.** 아래 구조로:
   ```yaml
   ---
   type: braindump
   domain: personal | professional | project-specific | mixed
   project: <slug>          # project-specific일 때만
   created: <YYYY-MM-DD HH:MM>
   themes: [..]
   tags: [braindump, ..]
   status: captured
   confidence: high | medium | low
   wiki_bridge_candidate: true | false
   wiki_bridge_status: deferred
   ---
   ```
   본문: `# Braindump: <자동 생성 제목>` → `## Raw Thoughts`(원문 그대로) →
   `## 분석`(테마/질문/결정) → `## 전략 인사이트`(있으면) →
   `## Action Items`(마감일은 `date`로 계산, 예: 내일 / +1주) →
   `## Connections`(관련 프로젝트·개념 `[[wikilink]]`).

6. **저장 위치 (SCHEMA §10).** 파일명 `braindump-YYYY-MM-DD-HHMM-<slug>.md`:
   - personal → `2-Areas/braindumps/personal/`
   - professional → `2-Areas/braindumps/work/`
   - project-specific → `1-Projects/<slug>/braindumps/`
   - mixed/불명 → `0-Inbox/`

   폴더가 없으면 만든다.

7. **Wiki bridge — DEFERRED 기본 (SCHEMA §8).** braindump은 본질적으로 단일
   1인칭 소스라 "≥ 2 소스" 임계값을 못 만족한다. 재사용 가능한 개념(이름 붙은
   인물·조직·도구, 명시적으로 정식화된 프레임워크·원칙)이 보이면:
   - frontmatter에 `wiki_bridge_candidate: true`, `wiki_bridge_status: deferred`.
   - **첫 캡처에서 `wiki/raw/`나 wiki 페이지로 복사하지 않는다.** 나중에 외부
     소스 ≥ 1개가 같은 개념을 뒷받침하면 `/ingest`가 승격한다.

   순수 운영성(오늘 할 일·일정·감정 해소)이면 `wiki_bridge_candidate: false`,
   wiki bridge 건너뜀.

8. **보고.** 저장 경로 / 식별된 주요 테마 / 도메인 + confidence. 한 줄로.

## 프라이버시 (SCHEMA §10)

내부 동료 PII가 섞이면 그 braindump은 wiki bridge 대상에서 제외한다. wiki에는
내부 인물 정보가 entity stub로도 들어가지 않는다.
