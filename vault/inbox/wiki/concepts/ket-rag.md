# KET-RAG

## Definition

**Knowledge-Enhanced Text RAG** — "A Cost-Efficient Multi-Granular Indexing Framework for Graph-RAG" (arXiv:2502.09304, SIGKDD 2025). 동일 코퍼스에 **풀 Knowledge Graph**와 **이분 그래프** 두 인덱스를 병존시켜, 정보 밀도가 높은 핵심 문서에만 값비싼 LLM 추출을 집중시키는 하이브리드 인덱싱 프레임워크.

## Core Philosophy

> "모든 문서가 풀 KG의 표현력을 필요로 하진 않는다. 한정된 LLM 비용을 중요한 문서에만 집중한다."

본질은 "무엇이 중요한가"가 아니라 **"한정된 LLM 자원을 어디에 분배할 것인가"**의 문제.

## Dual Index Structure

```
[Core KG — 정밀 표현]          [Auxiliary 이분 그래프 — 경량 색인]
 법령, 매뉴얼, 핵심 문서         회의록, 이메일, 로그
 LLM OpenIE 트리플 추출          TF-IDF 키워드 추출 (LLM 0회)
 고비용 / 고품질                  저비용 / 러프 매칭
```

## Full KG vs Bipartite Graph

| 항목 | 풀 KG | 이분 그래프 |
|---|---|---|
| 노드 종류 | 모든 엔티티 (1종류, 자유 연결) | 2종류 (chunk / keyword) |
| 엣지 의미 | 풍부한 관계 (founded_by, in...) | contains 단일 |
| 연결 자유도 | 임의 ↔ 임의 | 종류 다른 노드끼리만 |
| 구축 방법 | LLM OpenIE | TF-IDF / BM25 |
| LLM 호출 | 청크당 1~3회 | **0회** |
| 표현력 | 멀티홉 추론 가능 | 연관성만 |

이분 그래프는 사실상 **inverted index의 그래프 표현**이며, 전통 IR의 검색 색인과 구조가 매우 유사하다.

## Importance Scoring (Official Method)

1. **모든 청크로 이분 그래프 먼저 생성** (LLM 0회)
2. **PageRank 실행** — 그래프 구조 기반 중요도 점수화
3. **상위 K% 청크만 풀 KG 대상으로 선정**
4. **선정된 청크에만 LLM OpenIE 적용**

PageRank가 정보 밀도를 잘 잡는 이유: 많은 키워드와 연결 + 다른 고점수 청크와 키워드 공유 = 통계적으로 정보 밀도 높은 청크.

## Alternative Importance Criteria

- NER 기반 엔티티 밀도
- 문서 타입 메타데이터 (사전 지식 직접 반영)
- LLM 사전 분류 (비싸지만 정확)
- 임베딩 클러스터 다양성
- **하이브리드: 메타룰(법령→무조건 Core) + PageRank(회의록→자동 승급)**

## Keyword Extraction Without LLM

이분 그래프 키워드 추출은 전통 NLP:
- **TF-IDF** (논문 기본)
- YAKE, RAKE, KeyBERT
- 한국어는 형태소 분석(Mecab/Okt) + TF-IDF

## Pollution Mitigation Mechanism

1. 저밀도 문서의 엔티티 추출을 **아예 안 함** → 잡음 트리플 미생성
2. 이분 그래프는 키워드 통계 기반이라 **언쟁·잡담의 영향이 희석됨**
3. 검색 경로 분리 — 정밀 질의는 Core KG, 탐색 질의는 Auxiliary
4. 고밀도·저밀도 문서가 **물리적으로 다른 자료구조에 격리**됨

## Reported Results

- GraphRAG 대비 인덱싱 비용 수십% 절감
- Multi-hop QA 품질 유사하거나 상회
- 혼합 코퍼스(위키 + 뉴스 + 대화) 시나리오에서 특히 강점

## Fit with Heterogeneous Corpus Problem

이기종 문서(법령·매뉴얼 vs 회의록·이메일)가 섞인 환경에 아키텍처가 직접 대응한다. 사용자가 직관적으로 원하는 "법령은 깊게, 회의록은 얕게"를 구조적으로 강제. 저밀도 문서를 실수로 깊게 파는 일 자체가 불가능한 설계.

## Related Concepts

- [graphrag](graphrag.md)
- [graph-pollution](graph-pollution.md)
- [hybrid-rag](hybrid-rag.md)
- [raptor](raptor.md)
- [heterogeneous-corpus](heterogeneous-corpus.md)
