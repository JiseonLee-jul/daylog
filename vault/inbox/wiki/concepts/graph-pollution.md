# Graph Pollution

## Definition

지식 그래프 또는 그래프 기반 RAG 인덱스에서, 검색 대상이 될 가치가 없거나 추론을 왜곡하는 저품질 노드·엣지가 축적되어 검색·추론 품질을 구조적으로 저하시키는 현상.

## Causal Chain

Heterogeneous Corpus → Knowledge Density Variance → Granularity Mismatch → **Graph Pollution**

이기종 문서 공존이라는 원인 상황에서 정보 밀도 편차가 생기고, 이를 무시한 균일 처리가 Granularity Mismatch를 만들며, 그 증상으로 Graph Pollution이 나타난다.

## Four Manifestations

1. **Noise Entity Injection** — 저밀도 문서(잡담·언쟁)에서 LLM OpenIE가 무의미한 엔티티·트리플 추출
2. **Relation Overfitting to Trivia** — 중요·사소 엔티티 사이 의미 없는 관계 형성, 멀티홉 경로 오염
3. **Entity Inflation** — 동일 실체 중복 등록, 과도하게 일반적인 개념의 노드화 (Microsoft GraphRAG의 "benchmark", "methodology" 사례)
4. **Hub Node Distortion** — 자주 등장하는 불용 단어가 과연결 허브가 되어 PageRank·multi-hop을 왜곡

## Origin Points in Pipeline

- 청킹 단계: 잡음·핵심 구간을 같은 단위로 묶음
- 엔티티 추출 단계: LLM이 잡음에서도 "뭐라도" 추출
- 관계 추출 단계: 약한 근거로 관계 추정
- 그래프 통합 단계: alias resolution 부실로 중복 병합 실패

## Consequences

- 검색 정밀도 저하 (false positive 증가)
- LLM 컨텍스트 오염 → hallucination
- 멀티홉 추론 오도
- 인덱싱 LLM 비용 낭비
- 저장소 팽창
- 디버깅·유지보수 난이도 증가

## Mitigation Strategies

네 개의 개입 축으로 분류:
- **축 A (Chunking):** 경계 정리로 잡음-핵심 혼재 방지
- **축 B (Graph 계층화):** 구조적 희석 또는 물리적 격리
- **축 C (Saliency/Denoising):** 인덱싱 전 노이즈 제거 (가장 직접적)
- **축 D (Routing):** 인덱스 물리 분리로 오염 격리

## Key Insight

단일 균일 처리는 근본적으로 실패한다. 모든 효과적 해법은 "문서를 차등 처리"라는 공통 원리에 기반하며, 차이는 "어느 단계에서 차등하느냐"에 있다.

## Related Concepts

- [heterogeneous-corpus](heterogeneous-corpus.md)
- [graphrag](graphrag.md)
- [ket-rag](ket-rag.md)
- [raptor](raptor.md)
- [chunking-strategy](chunking-strategy.md)
- [saliency-denoising](saliency-denoising.md)
- [rag-routing](rag-routing.md)
