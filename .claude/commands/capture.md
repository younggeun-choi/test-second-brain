---
description: URL 또는 텍스트를 wiki/raw/ 또는 2-Areas/braindumps/ 에 원문 보존 (ingest 아님)
argument-hint: <URL 또는 캡처할 텍스트>
allowed-tools: Read, Write, Bash(shasum:*), Bash(date:*), WebFetch
---

당신은 이 vault의 **capture** 워커다. 입력을 분류해 원문을 보존하는 데까지만
한다. durable wiki 페이지로 승격하지 **않는다** — 그건 `/ingest`의 일이다.
규칙은 `3-Resources/wiki/SCHEMA.md` §3.2(raw frontmatter), §8(파일명),
§10(PARA 라우팅)을 따른다.

## 입력

$ARGUMENTS

## 절차

1. **분류한다.**
   - 입력이 URL이면 → 소스 캡처. (arxiv.org → `raw/papers/`, youtube/threads/x
     같은 영상·SNS → `raw/transcripts/`, 그 외 일반 웹 → `raw/articles/`)
   - 입력이 자유 텍스트(생각·메모)면 → `2-Areas/braindumps/`에 저장.
     폴더가 없으면 만든다.

2. **URL인 경우:**
   - `WebFetch`로 본문을 가져와 읽을 수 있는 markdown으로 추출(보일러플레이트
     제거, 본문·제목·발행일 보존).
   - 원문 언어를 감지(`ko|en|ja|zh|mixed|auto`). **본문은 번역하지 않는다.**
   - 본문 내용의 sha256을 계산: 본문을 임시로 쓴 뒤
     `shasum -a 256 <file>` 실행, 또는 추출 텍스트를 직접 해시.
   - 파일명 `YYYY-MM-DD-slug.md` (날짜 = 원문 발행일, 모르면 `unknown-slug`).
     오늘 날짜는 `date +%F`로 확인.
   - SCHEMA §3.2 frontmatter를 붙여 해당 `raw/<type>/` 폴더에 저장:
     ```yaml
     ---
     source_url: <원본 URL>
     source_type: article | paper | transcript | video | image | book
     ingested: <오늘>
     sha256: <hex>
     source_published: <발행일 또는 생략>
     language: <감지 언어>
     ---
     ```
   - **dedup:** 저장 전, 같은 sha256을 가진 raw 파일이 이미 있는지 확인.
     있으면 새로 쓰지 말고 "이미 캡처됨"이라고 보고.

3. **자유 텍스트인 경우:**
   - `2-Areas/braindumps/YYYY-MM-DD-<slug>.md`에 저장. 가벼운 frontmatter
     (`created`, `type: braindump`, `tags`)와 원문을 보존.
   - **첫 캡처를 wiki 페이지로 자동 분류하지 않는다** (CLAUDE.md §7,
     SCHEMA §8 ≥2 소스 임계값).

4. **보고한다.** 저장 경로, source_type, language, sha256 앞 8자리, 그리고
   "이건 raw 보존이고 durable 지식 승격은 `/ingest`로"라는 다음 단계 안내.

URL과 설명 텍스트가 함께 오면, URL은 raw로·코멘트는 braindump으로 각각 저장한다.
