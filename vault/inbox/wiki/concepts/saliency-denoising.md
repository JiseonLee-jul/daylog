# Saliency / Denoising (Pre-Index)

## Definition

인덱싱 이전 단계에서 저밀도·노이즈 콘텐츠를 물리적으로 제거·압축·필터링하는 기법군. Graph Pollution 해결 4개 축 중 축 C에 해당하며, "들어가기 전에 걸러낸다"가 철학. 회의록 잡담·언쟁 구간 제거에 가장 직접적.

## Execution Methods

### C-1: Summarization-before-Indexing

원문 전체를 LLM으로 요약 후 요약본을 청크로. Ragie.ai의 Summary Index + Chunk Index 이중 구성 패턴. 문서 단위, 대형 LLM 1회, hallucination 위험.

### C-2: LLMLingua / LongLLMLingua (Microsoft)

소형 LM(GPT-2-small, LLaMA-7B)으로 토큰별 perplexity 점수화 → 저중요 토큰 제거. 추출적 삭제이므로 hallucination 없음. NaturalQuestions에서 4x 압축 시 GPT-3.5 +21.4pt 보고. LLMLingua-2는 3-6배 빠름. LangChain·LlamaIndex 기본 통합.

### C-3: Dialogue Act Classification

발언을 greeting/question/decision/argue/info 등으로 분류 → "결정·정보·지시"만 유지. QMSum의 locate-then-summarize(NAACL 2021) 프레임워크. Classifier 추론 비용만, hallucination 없음.

### C-4: LLM Relevance Gating

각 청크에 LLM이 "의사결정·수치·지시 포함 여부" 점수 부여 → 임계값 미만 제거. ChunkRAG(arXiv:2410.19572), FILCO. 가장 유연하나 가장 비쌈.

### C-5: Extractive Summarization

BART 등 추출형 요약 모델로 문장 선택. 생성 아닌 추출이라 hallucination 없음. 구어체 불완전 문장 한계.

## Meeting Transcript Pipeline (Recommended)

1. Whisper 전사
2. pyannote diarization (화자 분리)
3. Dialogue Act 분류 (decision/info만 통과)
4. LLMLingua-2 토큰 압축
5. Proposition Indexing (atomic fact 추출)
6. 인덱싱

이 조합이 "1시간 회의 중 앞뒤 10분만 중요" 시나리오에 가장 직접 대응.

## Trade-offs

- 가장 직접적인 오염 방지 효과
- 필터 기준 변경 시 재인덱싱 비용
- 맥락상 중요한 반론·미묘한 발언 손실 가능
- 도메인별(기술/경영/법무) 필터 기준 튜닝 필요

## Related Concepts

- [graph-pollution](graph-pollution.md)
- [chunking-strategy](chunking-strategy.md)
- [heterogeneous-corpus](heterogeneous-corpus.md)
