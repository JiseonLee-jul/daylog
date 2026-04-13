# GraphRAG ↔ LLM Wiki

## Relationship: 대비되는 지식 구조화 전략

둘 다 원시 문서 위에 구조를 구축하지만, 철학과 메커니즘이 근본적으로 다르다.

## Comparison

| 측면 | GraphRAG | LLM Wiki |
|------|----------|----------|
| 구조물 | 엔티티·관계 지식 그래프 | 마크다운 위키 (요약, 개념, 연결) |
| 구축 방식 | 자동 트리플 추출 | LLM + 인간 큐레이션 협업 |
| 질의 시 | 그래프 탐색 → 원시 청크로 회귀 | 위키 페이지 직접 참조 (컴파일 완료) |
| 축적성 | 그래프 확장 (노드/엣지 추가) | 위키 복리 축적 (교차 참조 강화) |
| 최적 규모 | 대규모 코퍼스 자동화 | 소·중규모 개인/팀 지식 관리 |
| 인프라 | 벡터 DB + 그래프 DB | 마크다운 파일 + git |
| 유지 관리 | Graph Pollution 리스크 | Lint로 건강 검진 |

## Complementary Potential

두 접근은 배타적이지 않다. GraphRAG로 대규모 코퍼스를 자동 인덱싱하고, 핵심 발견을 LLM Wiki로 큐레이션하는 하이브리드 워크플로가 가능하다.

## Sources

- [gist_github_com_karpathy-llm-wiki](../summaries/gist_github_com_karpathy-llm-wiki.md)
- [260409_graph-pollution-heterogeneous-corpus](../summaries/260409_graph-pollution-heterogeneous-corpus.md)
