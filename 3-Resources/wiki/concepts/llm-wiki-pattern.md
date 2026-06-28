---
title: LLM-Wiki Pattern
created: 2026-06-28
updated: 2026-06-28
type: concept
tags: [ai-ml/llm, meta/methodology, meta/this-vault]
sources: []
ai-first: true
confidence: high
---

## For future Claude

이 vault가 기반으로 삼는 운영 패턴이자, 이 wiki의 자기참조 seed 페이지다.
핵심 주장: 시간이 지나며 축적되는 개인·연구·비즈니스 지식에 대해서는, LLM이
유지하는 markdown wiki가 기존 RAG보다 우월하다는 것. 사용자가 "이 vault는 왜
이렇게 구성됐나", "LLM-Wiki가 뭔가", "RAG와 어떻게 다른가"라고 물으면 이
페이지를 먼저 읽는다. 패턴 출처는 [[andrej-karpathy]]의 2026-04 gist다.

## 핵심 주장

기존 RAG는 매 query마다 raw 문서에서 지식을 다시 derive한다. LLM-Wiki 패턴은
대신 **지식을 한 번 persistent하고 서로 연결된 markdown wiki로 compile**하고,
새 출처가 들어올 때마다 그것을 최신 상태로 유지한다. query 대상은 raw 문서가
아니라 wiki다.

> The wiki is a persistent, compounding artifact. The cross-references are
> already there. The contradictions have already been flagged. The synthesis
> already reflects everything you've read.

## 3-layer 아키텍처

| Layer | 역할 | 가변성 |
|-------|------|--------|
| **Raw sources** | 기사·논문·transcript. 사용자가 curation. | Immutable. LLM은 읽기만. |
| **Wiki** | LLM이 생성한 entity / concept / comparison / query 페이지. 상호 링크. | LLM이 소유·유지. |
| **Schema** | 운영 규칙: 구조, 컨벤션, 워크플로. | 시간에 걸쳐 사용자 + LLM이 공동 진화. |

이 vault에서 그 3계층은 각각 `wiki/raw/`, `wiki/{entities,concepts,
comparisons,queries}/`, `wiki/SCHEMA.md`로 구현된다. 운영 워크플로는 5개
slash command — `/capture`(소스→raw), `/ingest`(raw→durable page),
`/query`(wiki-first 응답), `/lint`(health check), `/save`(명시적 저장)로
나뉜다.

## RAG와의 대비

- **RAG:** query time에 검색·재합성. 매번 처음부터. 모순은 매번 재발견.
- **LLM-Wiki:** ingest time에 합성·교차참조·모순 플래깅을 미리 해두고, query는
  이미 컴파일된 wiki를 읽기만 한다. 비용이 query→ingest로 이동하고, 지식이
  **복리로 누적**된다.

## 왜 PARA와 결합하는가

PARA(Projects / Areas / Resources / Archive)는 사람이 관리하는 행동 지향
조직법이다. LLM-Wiki는 LLM이 관리하는 지식 지향 KB다. 이 vault는 둘을 한 repo의
**두 영역**으로 공존시킨다(SCHEMA §1 Territoriality): `3-Resources/wiki/`는
엄격한 LLM 영역, 나머지 PARA는 느슨한 사람 영역. 두 규칙이 섞이지 않게 하는
것이 운영의 핵심이다.

## Related

- [[andrej-karpathy]] — 패턴 제안자 (entity stub은 첫 ingest 시 생성)
- SCHEMA.md §1 Territoriality, §6 Brain-First Protocol
