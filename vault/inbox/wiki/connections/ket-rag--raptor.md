# KET-RAG ↔ RAPTOR

## Relationship

**서로 다른 철학으로 계층화를 구현하는 두 축.** 둘 다 축 B(GraphRAG 계층화)에 속하며 Graph Pollution 완화를 목표로 하지만, 계층을 만드는 원리가 대조된다.

## Contrast

| 항목 | RAPTOR | KET-RAG |
|---|---|---|
| 계층 생성 원리 | 의미 기반 상향식 (GMM 클러스터링 → LLM 요약 → 재귀) | 중요도 기반 수평 분리 (PageRank) |
| 분리 기준 | 임베딩 유사도 | 문서 정보 밀도 |
| 이기종 대응 | 출처 무관 의미 재조직 | 출처·밀도별 물리 분리 |
| LLM 비용 | 모든 레벨 요약에 필요 | 핵심 문서에만 |
| 원본 구조 보존 | ❌ 해체됨 | ✅ Core KG는 원본 구조 유지 |

## Shared Insight

둘 다 "**검색 공간에 다양성을 주입한다**"는 공통 목표를 가진다. RAPTOR는 추상도 다양성을, KET-RAG는 표현력 다양성(풀 KG vs 이분 그래프)을 주입한다. 계층/분리 구조가 검색 탐색 경로가 아니라 **검색 공간의 구성 자체를 바꾸는 장치**라는 점이 공통.

## Combination Possibility

두 기법은 배타적이지 않다. 이기종 코퍼스에서:
- 법령·매뉴얼 → KET-RAG의 Core KG
- 회의록 대량 → RAPTOR 트리 (또는 이분 그래프)
- Agent가 쿼리에 따라 두 인덱스를 선택적으로 호출

이 조합은 "법령의 구조적 관계는 KG로, 회의록의 의미적 계층화는 RAPTOR로"라는 역할 분담이 가능하다.

## See Also

- [ket-rag](../concepts/ket-rag.md)
- [raptor](../concepts/raptor.md)
