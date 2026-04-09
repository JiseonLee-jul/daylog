# Graph Pollution ↔ Heterogeneous Corpus

## Relationship

**원인과 증상의 관계.** Heterogeneous Corpus는 원인 상황이고, Graph Pollution은 그것이 무처리 상태로 인덱싱될 때 나타나는 최종 증상이다. 인과 사슬: Heterogeneous Corpus → Knowledge Density Variance → Granularity Mismatch → Graph Pollution.

## Why Heterogeneous Corpus Causes Pollution

이기종 문서의 정보 밀도 편차를 균일 처리로 수용하려 하면 양방향 실패가 발생한다. 고밀도 문서(법령)는 under-granular로 섬세함을 잃고, 저밀도 문서(회의록)는 over-granular로 잡음이 그래프에 진입한다. 이 오염은 하나의 옳은 청크 크기가 존재하지 않기 때문에 파라미터 튜닝으로는 해결되지 않는다.

## Implication

Graph Pollution을 근본적으로 해결하려면 "이기종 코퍼스라는 상황 자체를 인정하고 차등 처리한다"는 원리를 받아들여야 한다. 네 개의 해결 축(Chunking, Graph 계층화, Saliency, Routing)은 모두 이 원리의 서로 다른 구현이다.

## See Also

- [graph-pollution](../concepts/graph-pollution.md)
- [heterogeneous-corpus](../concepts/heterogeneous-corpus.md)
