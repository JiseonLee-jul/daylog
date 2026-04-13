# Knowledge Compilation ↔ Hybrid RAG

## Relationship: 사전 컴파일 vs 질의 시 하이브리드 검색

Knowledge Compilation과 Hybrid RAG는 "어떻게 지식을 효과적으로 접근하느냐"에 대한 서로 다른 답변이다.

## Contrast

- **Knowledge Compilation**: 소스 수집 시점에 지식을 추출·종합·구조화하여 위키에 상주시킴. 질의 시 이미 컴파일된 위키를 참조. 인프라 최소(마크다운 + git).
- **Hybrid RAG**: 복수의 검색 방식·인덱스 구조를 조합하여 질의 시점의 검색 품질을 극대화. BM25 + Dense Vector, 구조화 + 비구조화, 멀티그래뉼러 인덱싱 등.

## Key Insight

Knowledge Compilation은 "검색을 최소화"하는 방향이고, Hybrid RAG는 "검색을 최적화"하는 방향. 전자는 소·중규모에서 인프라 없이 작동하고, 후자는 대규모에서 정밀한 검색이 필요할 때 강점을 발휘한다.

## Sources

- [gist_github_com_karpathy-llm-wiki](../summaries/gist_github_com_karpathy-llm-wiki.md)
