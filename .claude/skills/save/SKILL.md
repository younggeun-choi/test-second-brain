---
name: save
description: 이번 세션의 결정·산출물·지식을 PARA 또는 wiki 규칙에 맞는 위치에 보존한다. raw capture(/capture)·ingest(/ingest)와 헷갈리지 않게, 세션에서 만들어진 가치 있는 결과물을 명시적으로 저장할 때 사용.
argument-hint: <저장할 결정/산출물 설명>
allowed-tools: Read, Write, Edit, Bash(date:*), Glob
---

당신은 이 vault의 **save** 워커다. 명시적으로 요청된 세션 지식·결정·산출물을
올바른 위치에 보존한다. 이건 raw capture(`/capture`)도, raw→durable 승격
(`/ingest`)도 아니다 — **지금 세션에서 만들어진 가치 있는 결과물을 잃지 않게
저장**하는 일이다.

## 저장 대상

$ARGUMENTS

## 라우팅 결정 (SCHEMA §10)

1. **재사용 가능한 개념적 가치가 있는가?**
   - 단일 산출물을 넘어 미래에 재참조될 프레임워크·인사이트·결정 근거 →
     `wiki/` 후보.
     - 외부 entity/개념이고 ≥ 2 소스 또는 핵심 주제면 `entities/`·`concepts/`.
     - 단일 출처의 질의 결과/결정 기록이면 `wiki/queries/`에 `type: query`로.
     - wiki에 쓸 때는 SCHEMA §3.1 frontmatter + §4 프리앰블 + §5 출처 +
       상호참조 규칙을 모두 지키고, `index.md`·`log.md`를 갱신한다.
   - 그렇지 않으면 (특정 프로젝트/영역에 묶인 작업 산출물) → **PARA**:
     - 작업 결정/회의록 → `2-Areas/<domain>/`
     - 프로젝트 산출물 → 해당 `1-Projects/` 노트의 `## Log`
     - 임시·미분류 → `0-Inbox/`

2. **프라이버시 (SCHEMA §10):** 내부 동료 PII는 wiki 금지 → `2-Areas/people/`.

3. **모호하면 사용자에게 목적지를 확인**한 뒤 저장한다.

## 절차

- 목적지를 정하고, 적절한 frontmatter(`created`/`updated`는 `date +%F`)와 함께
  저장한다.
- wiki에 저장한 경우에만 `index.md`/`log.md`를 갱신한다.
- 무엇을 어디에 왜 저장했는지 한 줄로 보고한다.
