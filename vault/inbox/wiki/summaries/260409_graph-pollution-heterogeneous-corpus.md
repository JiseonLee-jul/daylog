---
source: raw/260409_graph-pollution-heterogeneous-corpus.md
date_compiled: 2026-04-09
topics: [graph-pollution, heterogeneous-corpus, graphrag, lightrag, raptor, ket-rag, rag-architecture]
---

# Graph Pollution 문제 정의와 해결 전략 고찰

## 개요

이기종 문서(법령·회의록·이메일·연구자료)를 단일 GraphRAG 지식베이스로 통합할 때 발생하는 **Graph Pollution** 문제를 정의하고, 업계·학계의 해결책을 4개 축으로 분해해 비교 고찰한 정리문이다. LightRAG 기반 사내 지식베이스 구축 맥락에서 작성되었으며, 이론적 프레임부터 실전 점진 도입 전략까지 포괄한다.

## 문제 정의

**Heterogeneous Corpus → Knowledge Density Variance → Granularity Mismatch → Graph Pollution**이라는 인과 사슬로 문제를 정의한다. Graph Pollution이란 "지식 그래프 인덱스에서 저품질 노드·엣지가 축적되어 검색·추론 품질을 구조적으로 저하시키는 현상"이며, 네 가지 양상(Noise Entity Injection, Relation Overfitting to Trivia, Entity Inflation, Hub Node Distortion)으로 나타난다. 파이프라인 상 ① 청킹 ② 엔티티 추출 ③ 관계 추출 ④ 그래프 통합 네 지점 모두가 오염 원천이 될 수 있다.

## 네 개의 해결 축

파이프라인 개입 단계별로 네 축에 해법을 분류한다.

- **축 A (Chunking):** 문서 타입별 다른 자르기. 룰 기반 / 구조 파서 / 의미 기반 / LLM 프롬프트 기반. 대표: Dense X Retrieval, Anthropic Contextual Retrieval, Late Chunking.
- **축 B (Graph 계층화):** Microsoft GraphRAG(Leiden 클러스터링), LightRAG(dual-level), RAPTOR(GMM 재귀 요약), KET-RAG(풀 KG + 이분 그래프 이중 인덱스).
- **축 C (Saliency/Denoising):** 인덱싱 이전 정제. LLM 요약 / LLMLingua / Dialogue Act 분류 / 관련성 게이팅 / 추출 요약. 회의록 노이즈에 가장 직접적.
- **축 D (Routing):** 메타 필터 / LLM Selector / 쿼리 분해 / Agentic RAG / Federated. 오염을 물리적으로 격리.

## 축 B 심화 — 계층의 진짜 의미

**RAPTOR의 핵심 통찰:** 계층 구조는 "탐색 구조"가 아니라 **"다양한 추상도의 요약 노드를 생성하기 위한 장치"**다. Collapsed Tree 검색이 트리 순회를 하지 않더라도, 트리 생성 과정(GMM 클러스터링 → LLM 요약 → 재귀)이 남긴 요약 노드들이 검색 공간에 추상도 다양성을 주입한다는 사실 자체가 가치다. 계층 생성이 없다면 Collapsed Tree는 평범한 벡터 검색과 동일해진다.

**KET-RAG의 핵심 통찰:** 문제의 본질은 "무엇이 중요한가"가 아니라 **"한정된 LLM 비용을 어디에 집중할 것인가"**다. PageRank로 상위 청크를 선정해 풀 KG를 구축하고, 나머지는 TF-IDF 기반 이분 그래프(사실상 inverted index의 그래프 표현)로 처리한다. 풀 KG와 이분 그래프의 차이는 "의미 관계의 유무"이며, 이것이 LLM 호출 비용의 대칭을 결정한다.

## 공통 패턴 — Graph as Index, Text as Content

모든 GraphRAG 계열(Microsoft GraphRAG, LightRAG, RAPTOR, KET-RAG, HippoRAG)의 공통 아키텍처: 그래프 탐색으로 관련 청크를 찾고, 원본 청크를 fetch해, 그 원문을 LLM 컨텍스트로 답변 생성. **그래프의 entity·relation은 LLM에 직접 들어가지 않고 "어느 청크를 봐야 할지" 결정하는 라우팅 정보로만 쓰인다.** 따라서 LLM 비용은 답변 생성이 아닌 검색 품질에 대한 투자이며, KET-RAG의 영리함은 "모든 청크가 멀티홉 탐색 대상일 필요 없다"는 통찰에서 나온다.

## LightRAG 기반 실전 권장

기존 인프라를 유지하며 점진적으로 도입하는 ROI 기준 순서: **Phase 1** 메타 태그(D-1) + 구조 파서(A-2) → **Phase 2** 회의록 LLMLingua 압축(C-2) → **Phase 3** Proposition 추출(A-4 부분) → **Phase 4** KET-RAG 식 이중 인덱스 차용(B-4). 선택적 고도화로 회의록 전용 RAPTOR 트리 추가, 크로스 질의 증가 시 Agentic RAG(D-4)로 진화. 메타 태그 라우팅 + 저밀도 문서 전처리 조합이 비용 대비 효과가 가장 높다.

## Related Concepts

- [graph-pollution](../concepts/graph-pollution.md)
- [heterogeneous-corpus](../concepts/heterogeneous-corpus.md)
- [graphrag](../concepts/graphrag.md)
- [raptor](../concepts/raptor.md)
- [ket-rag](../concepts/ket-rag.md)
- [lightrag](../concepts/lightrag.md)
- [hybrid-rag](../concepts/hybrid-rag.md)
- [chunking-strategy](../concepts/chunking-strategy.md)
- [saliency-denoising](../concepts/saliency-denoising.md)
- [rag-routing](../concepts/rag-routing.md)
