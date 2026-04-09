# Heterogeneous Corpus

## Definition

성격이 다른 문서들(법령·매뉴얼·회의록·이메일·로그·연구자료 등)이 한 지식베이스에 공존하는 상황 자체를 가리키는 IR 분야의 고전 용어. 조직의 실제 지식베이스는 본질적으로 이기종이다.

## Associated Property: Knowledge Density Variance

이기종 코퍼스의 측정 가능한 속성으로, 문서별 단위 토큰당 정보 밀도가 본질적으로 다르다. 법령 조문은 고밀도(항목 하나하나가 중요), 회의록은 저밀도(잡담·언쟁이 섞임)로 대비된다.

## Why Uniform Processing Fails

하나의 "올바른 입도"는 존재하지 않는다. 200p 법령과 1시간 회의록을 동일 청크 크기·동일 추출 정밀도로 처리하면:
- 법령은 조항의 섬세함 손실 (under-granular)
- 회의록은 잡담까지 그래프에 진입 (over-granular)
- 양쪽 모두 적정 입도 이탈

## Leads To

Knowledge Density Variance를 무시한 균일 처리는 **Granularity Mismatch**를 일으키고, 그 결과 **Graph Pollution**이 발생한다.

## Mitigation Principle

모든 효과적 해법은 "문서를 차등 처리한다"는 공통 원리에 기반한다. 차등의 기준은 문서 타입(메타데이터), 정보 밀도(PageRank/TF-IDF), 의미 클러스터(임베딩) 등이 될 수 있다.

## Related Concepts

- [graph-pollution](graph-pollution.md)
- [chunking-strategy](chunking-strategy.md)
- [rag-routing](rag-routing.md)
- [ket-rag](ket-rag.md)
