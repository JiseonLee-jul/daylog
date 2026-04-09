# LightRAG

## Definition

Simple and Fast Retrieval-Augmented Generation (arXiv:2410.05779, 2024). 단일 지식 그래프를 구축하되, 쿼리 시점에 **dual-level retrieval**로 두 경로를 분리하는 경량 GraphRAG 변종.

## Dual-Level Retrieval

- **Low-level retrieval** — 특정 엔티티와 직접 관계 (정의, 속성, 작성자 등 사실 질의)
- **High-level retrieval** — 복수 엔티티에 걸친 주제·개념 집계 질의
- **Hybrid mode** — 두 결과 혼합

## Characteristics vs Other GraphRAG

- **Microsoft GraphRAG**와 달리 커뮤니티 계층 없음 (단일 그래프)
- 입도 차이 처리를 **인덱스 구조가 아닌 쿼리 시점 라우팅**으로 수행
- 경량·고속이 장점, 그러나 인덱싱 단계의 오염 격리는 없음

## Limitations for Heterogeneous Corpus

단일 그래프에 이기종 문서가 모두 진입하므로 **Graph Pollution 방지 메커니즘이 인덱스 레벨에 없다**. 쿼리 분기만으로는 저밀도 문서의 잡음 트리플이 그래프에 축적되는 문제를 해결하지 못함.

## Enhancement Path

LightRAG 코어를 유지하면서 오염을 줄이는 현실적 경로:
- 전처리(축 C)로 회의록 노이즈 제거 후 인덱싱
- 메타 태그(축 D-1) 부여로 쿼리 시 필터링
- KET-RAG 아이디어 차용: 법령만 Core, 저밀도는 별도 인덱스로 분리

## Related Concepts

- [graphrag](graphrag.md)
- [graph-pollution](graph-pollution.md)
- [ket-rag](ket-rag.md)
- [rag-routing](rag-routing.md)
