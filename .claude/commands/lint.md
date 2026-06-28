---
description: wiki health check — 링크·frontmatter·index/log·raw drift·페이지 크기 (SCHEMA §9)
argument-hint: [점검 범위 또는 비워서 전체]
allowed-tools: Read, Glob, Grep, Bash(shasum:*), Bash(ls:*)
---

당신은 이 vault의 **lint** 점검자다. `3-Resources/wiki/SCHEMA.md` §9 체크리스트를
`3-Resources/wiki/` 전체에 실행한다. 기본은 **리포트 모드** — 발견을 심각도별로
보고하되, 파괴적 수정은 사용자 승인 없이 하지 않는다.

## 범위

$ARGUMENTS  (비어 있으면 wiki 전체)

## 체크리스트 (SCHEMA §9)

| # | 검사 | 심각도 |
|---|------|--------|
| 1 | 고아 페이지 (inbound `[[링크]]` 없음) | warning |
| 2 | 깨진 wikilink (`[[Target]]`인데 파일 없음) | critical |
| 3 | index 완전성 (모든 페이지가 `index.md`에) | critical |
| 4 | frontmatter 검증 (필수 필드, 태그 ∈ SCHEMA §2) | critical |
| 5 | 오래된 내용 (`updated` > 90일 vs 더 새 소스) | warning |
| 6 | 충돌 (`contradictions:` 또는 `contested: true`) | info |
| 7 | confidence 감사 (`low` 또는 단일 소스에 없음) | warning |
| 8 | 소스 drift (`raw/*` 재해시, frontmatter `sha256` 비교) | warning |
| 9 | 페이지 크기 (콘텐츠 페이지 > 200 후보 / > 300 필수. SCHEMA·index·log·raw 면제) | warning / critical |
| 10 | 태그 감사 (모든 태그 ∈ SCHEMA §2) | warning |
| 11 | log 회전 (`log.md` > 500 entries) | info |

## 절차

1. `entities/ concepts/ comparisons/ queries/`의 모든 페이지를 나열·읽는다.
2. 각 검사를 수행. wikilink 그래프는 Grep으로 `[[...]]` 수집해 양방향 대조.
3. 소스 drift는 각 raw 파일 본문을 `shasum -a 256`로 재해시해 frontmatter
   `sha256`과 비교.
4. **결과를 심각도별로 보고** (critical → warning → info). 각 항목에 파일
   경로와 한 줄 진단.

## 수정 정책

- **안전한 자동 수정 OK (보고 후):** `index.md` 재빌드(누락 페이지 추가).
- **승인 필요:** frontmatter 수정, 페이지 분할, 링크 교정, rename. 제안만 하고
  사용자 확인을 받는다 (CLAUDE.md §7 — lint 없이 일괄 rename 금지).
- raw 파일은 절대 수정하지 않는다. drift는 보고만 (재-fetch 필요 표시).
