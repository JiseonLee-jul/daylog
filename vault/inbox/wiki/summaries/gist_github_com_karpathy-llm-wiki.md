---
source: raw/gist_github_com_karpathy-llm-wiki.md
date_compiled: 2026-04-13
key_topics: [LLM Wiki, Personal Knowledge Base, Knowledge Compilation, RAG Alternative, Memex, Obsidian]
---

# LLM Wiki — Andrej Karpathy

## Summary

Andrej Karpathy가 제안하는 "LLM 위키" 패턴은 기존 RAG 방식의 근본적 한계에 대한 대안이다. 기존 RAG(NotebookLM, ChatGPT 파일 업로드 등)는 매 질의마다 원시 문서에서 관련 청크를 검색해 답변을 생성하므로 지식이 축적되지 않는다. 반면 LLM 위키 패턴에서는 LLM이 **지속적인 위키를 점진적으로 구축하고 유지**한다 — 새로운 소스가 추가될 때마다 읽고, 핵심 정보를 추출하고, 기존 위키에 통합하여 교차 참조를 업데이트하고 모순을 표시한다. 지식은 한 번 컴파일되고 최신 상태로 유지되며, 매번 재도출되지 않는다.

아키텍처는 3개 레이어로 구성된다: (1) **원시 소스** — 불변의 소스 문서 컬렉션 (진실의 원천), (2) **위키** — LLM이 전적으로 소유하는 마크다운 파일 디렉토리 (요약, 개체 페이지, 개념 페이지, 비교, 종합), (3) **스키마** — LLM에게 위키 구조, 규칙, 워크플로를 알려주는 설정 문서 (CLAUDE.md 또는 AGENTS.md). 핵심 작업은 세 가지: Ingest(소스 수집 및 위키 통합), Query(위키 대상 질의 및 답변 재정리), Lint(주기적 건강 검진 — 모순, 고아 페이지, 누락 교차 참조 탐지).

Karpathy는 이 패턴을 Vannevar Bush의 Memex(1945)와 정신적으로 연결짓는다 — 문서 간 연상적 경로를 가진 개인 큐레이션 지식 저장소. Bush가 해결하지 못한 "누가 유지 관리를 하느냐"의 문제를 LLM이 해결한다. 인간은 소스 큐레이션, 분석 지시, 질문에 집중하고, LLM은 요약, 교차 참조, 정리, 기록 관리 등 모든 고된 작업을 담당한다. 실제 워크플로는 한쪽에 LLM 에이전트, 다른 쪽에 Obsidian을 열어두고 LLM이 편집하면 실시간으로 결과를 탐색하는 방식이다.

적용 분야는 광범위하다: 개인 자기 계발 추적, 연구 논문 종합, 독서 동반자 위키, 비즈니스 내부 위키(Slack/회의록/고객 통화 기반), 경쟁 분석, 여행 계획 등. 이 문서는 의도적으로 추상적이며 특정 구현이 아닌 패턴을 전달하는 것이 목적이다.

## Key Insights

- **위키는 복리로 쌓이는 산출물**: 소스 추가와 질문마다 풍부해지며, 좋은 답변도 새 페이지로 재정리 가능
- **index.md + log.md 이중 탐색**: 콘텐츠 지향 인덱스 + 시간순 로그로 ~100개 소스 규모에서 임베딩 기반 RAG 인프라 불필요
- **LLM이 유지 관리 비용을 제로에 가깝게**: 인간이 위키를 포기하는 이유(유지 부담 > 가치)를 제거
- **선택적 CLI 도구**: 규모가 커지면 qmd 같은 로컬 검색 엔진(BM25/벡터 하이브리드 + LLM 재순위화) 추가 가능

## Related Concepts

- [llm-wiki](../concepts/llm-wiki.md)
- [personal-knowledge-base](../concepts/personal-knowledge-base.md)
- [knowledge-compilation](../concepts/knowledge-compilation.md)
- [memex](../concepts/memex.md)
- [graphrag](../concepts/graphrag.md)
- [agentic-architecture](../concepts/agentic-architecture.md)
