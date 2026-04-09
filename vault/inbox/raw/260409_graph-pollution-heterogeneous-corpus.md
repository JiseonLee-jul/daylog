# Graph Pollution 문제 정의와 해결 전략 고찰

*작성일: 2026-04-09*
*맥락: LightRAG 기반 사내 지식베이스 구축 중, 이기종 문서(법령·회의록·이메일·연구자료 등)를 단일 그래프에 임베딩할 때 발생하는 품질 저하 문제를 정의하고 해결책을 탐색*

---

## 1. 문제 정의

### 1.1 배경 — Heterogeneous Corpus 문제의 인과 흐름

조직의 지식베이스는 본질적으로 **이기종 코퍼스(heterogeneous corpus)**다. 법령·매뉴얼 같은 고구조·고밀도 문서부터 회의록·채팅 로그 같은 저구조·저밀도 문서까지 성격이 전혀 다른 자료들이 한 시스템에 모인다.

```
Heterogeneous Corpus (원인·상황)
        │ 문서별 정보 밀도가 본질적으로 다름
        ▼
Knowledge Density Variance (속성)
        │ 이 특성을 무시하고 균일하게 인덱싱하면
        ▼
Granularity Mismatch (문제)
        │ 결과로 나타남
        ▼
Graph Pollution (증상)
```

- **Heterogeneous Corpus:** 성격이 다른 문서들이 한 KB에 공존하는 상황 자체
- **Knowledge Density Variance:** 그 코퍼스의 측정 가능한 속성 — 단위 토큰당 담긴 정보량이 크게 다름
- **Granularity Mismatch:** 밀도 편차를 무시하고 동일한 입도로 처리했을 때의 구조적 불일치
- **Graph Pollution:** 저밀도·노이즈 문서의 엔티티·관계가 그래프 인덱스에 유입되어 검색·추론 품질을 저하시키는 현상

### 1.2 Graph Pollution의 엄밀한 정의

**"지식 그래프 또는 그래프 기반 RAG 인덱스에서, 검색 대상이 될 가치가 없거나 추론을 왜곡하는 저품질 노드·엣지가 축적되어 검색·추론 품질을 구조적으로 저하시키는 현상."**

네 가지 양상:

**양상 1: Noise Entity Injection** — 저밀도 문서(잡담, 언쟁)에서 LLM OpenIE가 무의미한 엔티티·트리플을 추출해 그래프에 삽입.

**양상 2: Relation Overfitting to Trivia** — 중요 엔티티와 사소한 엔티티 사이에 의미 없는 관계가 형성되어 멀티홉 검색 시 잘못된 경로 생성.

**양상 3: Entity Inflation** — 동일 실체의 중복 등록, 과도하게 일반적인 개념의 노드화 ("benchmark", "methodology" 등이 엔티티가 되는 Microsoft GraphRAG 실전 보고).

**양상 4: Hub Node Distortion** — 자주 언급되지만 의미 없는 단어가 과연결 허브가 되어 PageRank나 multi-hop 탐색을 왜곡.

### 1.3 피해

- 검색 정밀도 저하
- LLM 컨텍스트 오염 → hallucination
- 멀티홉 추론 오도
- 인덱싱 비용 낭비
- 저장소 팽창
- 디버깅·유지보수 난이도 증가

### 1.4 발생 지점

```
문서 → 청킹 → 엔티티추출 → 관계추출 → 그래프통합
        ①        ②           ③          ④
```

① 청킹: 잡음·핵심 구간을 같은 단위로 묶음
② 엔티티: LLM이 잡음에서도 "뭐라도" 추출
③ 관계: 약한 근거로 관계 추정
④ 통합: 중복 병합(alias resolution) 부실

---

## 2. 해결 전략 — 네 개의 축

업계 해법은 파이프라인의 **어느 단계에 개입하는가**로 4축에 배치된다.

```
┌─────────┐  ┌──────────┐  ┌─────────┐  ┌────────┐  ┌────────┐
│ Source  │─▶│ Pre-proc │─▶│ Chunking│─▶│ Index  │─▶│ Query  │
└─────────┘  └──────────┘  └─────────┘  └────────┘  └────────┘
                  ▲             ▲           ▲           ▲
                [축 C]        [축 A]      [축 B]      [축 D]
                Saliency     Chunking    Graph       Routing
                Denoising    전략        계층화      아키텍처
```

### 2.1 축 A — Chunking 전략

**철학:** "자르는 방식을 문서마다 다르게"

| 하위 기법 | 메커니즘 | 비용 |
|---|---|---|
| A-1 룰 기반 | 확장자·MIME별 사전 정의 파서 (LlamaIndex `SimpleFileNodeParser`) | ⭐ |
| A-2 구조 파서 | 문서 내부 마커 감지 (Unstructured.io) | ⭐ |
| A-3 의미 기반 | 임베딩 유사도 급변점 (LangChain `SemanticChunker`) | ⭐⭐ |
| A-4 LLM 기반 | 프롬프트로 분할 지시 (Dense X Retrieval, Contextual Retrieval) | ⭐⭐⭐⭐ |

**대표 기법:** Dense X Retrieval (Proposition Indexing, EMNLP 2024), Anthropic Contextual Retrieval (2024, 실패율 67%↓), Late Chunking (Jina AI, 2024), Parent-Child Hierarchical.

**한계:** 경계만 정리할 뿐 노이즈 자체 제거는 못 함.

### 2.2 축 B — Graph 계층화

**철학:** "그래프 구조 자체를 다계층으로 또는 이중으로"

| 하위 기법 | 메커니즘 |
|---|---|
| B-1 Microsoft GraphRAG | Leiden 재귀 클러스터링 + 커뮤니티 요약 |
| B-2 LightRAG | 단일 그래프 + 쿼리 시점 dual-level 분기 |
| B-3 RAPTOR | 임베딩 기반 GMM clustering → LLM 요약 → 재귀 트리 |
| B-4 KET-RAG | 중요도 기반 풀 KG + 이분 그래프 이중 인덱스 |

### 2.3 축 C — Saliency/Denoising

**철학:** "들어가기 전에 걸러내거나 압축한다"

| 하위 기법 | 판단 주체 | 실행 단위 | hallucination |
|---|---|---|---|
| C-1 LLM 요약 | 대형 LLM | 문서 | 🔴 |
| C-2 LLMLingua | 소형 LM | 토큰 | 🟢 |
| C-3 Dialogue Act 분류 | classifier | 발언 | 🟢 |
| C-4 LLM 관련성 게이팅 | 대형 LLM | 청크 | 🟢 |
| C-5 추출형 요약 | BART 등 | 문장 | 🟢 |

**대표:** LLMLingua/LongLLMLingua (Microsoft), ChunkRAG, FILCO, QMSum locate-then-summarize, Proposition Indexing.

**회의록 특화 파이프라인:** Whisper 전사 → pyannote diarization → dialogue act 분류 → LLMLingua-2 압축 → proposition 추출 → 인덱싱.

### 2.4 축 D — Routing 아키텍처

**철학:** "아예 다른 방에 보관, 쿼리에 따라 라우팅"

| 하위 기법 | 라우팅 주체 |
|---|---|
| D-1 메타 필터 | 앱/사용자 (Azure AI Search) |
| D-2 LLM Selector | LlamaIndex RouterQueryEngine |
| D-3 쿼리 분해 | SubQuestionQueryEngine |
| D-4 Agentic RAG | 에이전트 루프 (Tool 등록) |
| D-5 Federated/Tiered | 물리적 분리 (Vectara, Glean) |

---

## 3. 4개 축 시스템 영향 비교

| 축 | 개입 단계 | 비용 | 난이도 | 오염 해결력 | LightRAG 변경 |
|---|---|---|---|---|---|
| A. Chunking | 청킹 | 낮음 | ★ | 🟡 부분 | 없음 |
| B. Graph 계층 | 인덱스 구조 | 높음 | ★★★ | 🟢 구조적 | 큼 |
| C. Saliency | 청킹 이전 | 높음 | ★★ | 🟢 가장 직접 | 전처리 추가 |
| D. Routing | 시스템 전체 | 중 | ★★ | 🟢 격리 | 매우 큼 |

**한 줄 차별점:**
- **A** — "같은 데이터, 다른 가위질": 저렴, 경계만 정리
- **B** — "그래프에 층을 쌓아 노이즈 희석": 이론적 우아함, 구현 난이도 최대
- **C** — "들어가기 전에 걸러낸다": 가장 직접적, 손실 리스크
- **D** — "다른 방에 보관": 물리적 격리, 크로스-타입 질의 난점

---

## 4. 축 B 심화 — GraphRAG가 오염을 줄이는 원리

### 4.1 계층이 비용을 줄이는 두 방식

**Top-down Traversal:** 루트에서 하위로 가지치기, 무관 가지 pruning, 검색 공간 로그 스케일 감소.

**Level Selection (GraphRAG global vs local, LightRAG dual-level):** 쿼리 추상도에 맞는 레벨만 검색.

### 4.2 계층이 노이즈를 줄이는 세 메커니즘

1. **요약 과정에서 물리적 제거** — LLM이 요약 작성 시 잡담 자연 생략
2. **추상 쿼리는 하위에 닿지 않음** — 노이즈가 있어도 조회되지 않으면 무해
3. **구체 쿼리도 좁은 가지만 탐색** — 다른 가지의 노이즈와 격리

### 4.3 RAPTOR의 진짜 의미 — "트리는 저장 구조, 검색은 평면"

논문 실험: Tree Traversal보다 **Collapsed Tree**(모든 레벨 노드 평면화 후 유사도 검색)가 우수.

**핵심 통찰:**
> RAPTOR의 트리는 "탐색 구조"가 아니라 **"다양한 추상도의 요약 노드를 생성하기 위한 장치"**다.

트리 생성 과정(GMM 클러스터링 → LLM 요약 → 재귀)은 **검색 공간에 추상도 다양성을 주입**한다. Collapsed Tree는 그 주입된 공간에서 flat 검색을 수행할 뿐. **계층 생성이 없다면 Collapsed Tree는 평범한 벡터 검색과 동일.**

이 통찰은 모든 계층적 GraphRAG의 본질을 관통: **계층은 검색 알고리즘이 아니라 검색 공간의 다양성을 만든다.**

### 4.4 RAPTOR 저장 단계 상세

1. 청크 분할 (보통 100토큰)
2. 청크 임베딩 (SBERT/ada)
3. **UMAP 차원 축소** (global → local 2단계) — 고차원 거리 왜곡 방지
4. **GMM 클러스터링** — K-means 아닌 이유: soft assignment (한 청크가 여러 클러스터에 속함), BIC로 최적 클러스터 수 자동 결정
5. 클러스터별 LLM 요약 → Level 1 노드
6. 재귀
7. 모든 레벨 노드를 **평면 테이블로 벡터 DB 저장**: `node_id | level | text | embedding | children_ids | source_chunk_ids`

### 4.5 RAPTOR 검색 두 모드

| 모드 | 특징 | 논문 권장 |
|---|---|---|
| Tree Traversal | 가지치기, 저비용, 상위 오판 시 손실 | ✗ |
| Collapsed Tree | 전체 스캔, 쿼리 적응적, 더 정확 | ✅ |

Collapsed Tree는 쿼리별로 적절한 레벨의 노드가 자동 선택됨 — 추상 질의는 상위 요약, 구체 질의는 하위 원본.

### 4.6 KET-RAG 상세

**핵심 철학:** "모든 문서가 풀 KG의 표현력을 필요로 하진 않는다. 한정된 LLM 비용을 중요한 문서에만 집중."

**구조:**
```
[Core KG — 정밀 표현]          [Auxiliary 이분 그래프 — 경량 색인]
 법령, 매뉴얼                    회의록, 이메일, 로그
 LLM OpenIE 트리플 추출          TF-IDF 키워드 추출 (LLM 0회)
```

**풀 KG vs 이분 그래프 차이:**
- 풀 KG: 모든 엔티티가 1종류, 풍부한 관계 의미(founded_by, located_in…), LLM 필요
- 이분 그래프: 노드 2종류(chunk ↔ keyword), 관계는 "contains"뿐, 사실상 **inverted index의 그래프 표현**

**중요도 판정 (KET-RAG 논문 공식 방법):**
1. 전체 청크로 이분 그래프 먼저 생성 (LLM 0회)
2. 이분 그래프 위에서 **PageRank** 실행
3. 상위 K% 청크만 풀 KG 대상으로 선정
4. 선정된 청크에만 LLM OpenIE 적용

PageRank가 정보 밀도를 잘 잡는 이유: 많은 키워드와 연결 + 다른 고점수 청크와 키워드 공유 = 정보 밀도 높은 청크.

**대안 기준:** NER 엔티티 밀도, 문서 타입 메타데이터, 임베딩 클러스터 다양성, **하이브리드(메타룰 + PageRank)**.

**KET-RAG 키워드 추출이 LLM 없이 작동하는 이유:** TF-IDF, YAKE, KeyBERT, RAKE, 한국어는 형태소 분석(Mecab) + TF-IDF 조합. 논문 기본은 TF-IDF.

---

## 5. 공통 패턴 — "Graph as Index, Text as Content"

모든 GraphRAG 계열(Microsoft GraphRAG, LightRAG, RAPTOR, KET-RAG, HippoRAG)이 공유하는 아키텍처:

```
[1] 그래프 탐색 → 관련성 판단, 멀티홉 추론
[2] 원본 청크 fetch → 그래프 노드가 가리키는 원문
[3] LLM 답변 생성 → 원본 청크를 컨텍스트로
```

**핵심:** 그래프의 entity·relation은 LLM에 **직접 들어가지 않는다**. 그것들은 "어느 청크를 봐야 할지" 결정하는 **라우팅 정보**로만 쓰인다.

**이유:**
1. LLM은 자연어 원문을 트리플보다 잘 다룸 (뉘앙스·조건·부정 보존)
2. 그래프 추출은 손실 압축
3. 감사 가능성 (정확한 출처 추적)

**함의:**
- 그래프 구축의 가치는 "올바른 청크를 찾는 능력"에 있음
- LLM 비용은 답변 생성이 아닌 **검색 품질에 대한 투자**
- KET-RAG의 영리함: "모든 청크가 멀티홉 탐색 대상일 필요 없다"

**예외:** Pure Symbolic KGQA (Wikidata QA 등)는 트리플만으로 답변. 구조화 데이터 한정.

---

## 6. 하이브리드 RAG의 세 층위

"하이브리드 RAG"는 여러 층위로 쓰임:

| 층위 | 의미 | 대표 |
|---|---|---|
| 1. Retrieval 하이브리드 | BM25 + dense vector | Elasticsearch + 벡터 DB |
| 2. Knowledge Source 하이브리드 | 구조화 + 비구조화 | HybGRAG |
| 3. Index Structure 하이브리드 | 동일 데이터를 다른 자료구조로 중복 인덱싱 | **KET-RAG** |

KET-RAG는 층위 3의 명백한 사례로, "multi-granular hybrid indexing"이라 불린다.

---

## 7. 실전 권장 조합 (LightRAG 기반 점진 도입)

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  축 D    │─▶│  축 C    │─▶│  축 A    │─▶│  축 B    │
│타입라우터│  │회의록필터│  │타입별청킹│  │LightRAG  │
│(메타태그)│  │  /압축   │  │          │  │dual-level│
└──────────┘  └──────────┘  └──────────┘  └──────────┘
```

**ROI 기준 도입 순서:**

**Phase 1 (D-1 + A-2):** `doc_type`, `density_tier` 메타 태그 부여. 법령=Unstructured 조문 파서, 회의록=화자 턴, research=Markdown 헤딩. LLM 비용 0.

**Phase 2 (C-2):** 회의록만 LLMLingua-2로 토큰 압축. 소형 LM.

**Phase 3 (A-4 부분):** 회의록에 Proposition 추출 추가.

**Phase 4 (B-4 차용):** LightRAG 코어 유지, 법령·매뉴얼만 풀 KG로, 저밀도 문서는 경량 벡터 인덱스/이분 그래프로 분리.

**선택적 고도화:** 회의록 누적 시 RAPTOR 트리를 회의록 전용 인덱스로 추가. 크로스 질의 증가 시 Agentic RAG (D-4)로 진화.

---

## 8. 핵심 고찰 요약

**문제의 본질:** Graph Pollution은 "노이즈 제거" 문제가 아니라 **"이종 문서 간 정보 밀도 차이를 인덱스 구조가 어떻게 수용할 것인가"**의 문제. 균일 처리는 근본적으로 실패.

**해결의 본질:** 네 축 모두 "문서를 차등 처리한다"는 원리. 차이는 **어느 단계에서** 차등하느냐 (A: 자를 때 / B: 구조화할 때 / C: 정제할 때 / D: 저장할 때).

**GraphRAG 계열의 공통 진리:**
- 그래프는 **색인이지 내용이 아니다**
- 계층은 **탐색 구조가 아니라 검색 공간의 다양성 주입 장치다**
- LLM 비용은 **"어디에 쓰느냐"가 "얼마나 쓰느냐"보다 중요하다** (KET-RAG의 통찰)

**실무적 결론:** 완벽한 단일 해법은 없다. 네 축 조합 + 점진적 도입 + 기존 인프라(LightRAG) 유지가 현실적. **메타 태그 라우팅(D-1) + 저밀도 문서 전처리(C-2)**가 ROI 최대.

---

## 참고 문헌

- Edge, D. et al. "From Local to Global: A Graph RAG Approach" (Microsoft, 2024)
- Guo, Z. et al. "LightRAG" (arXiv:2410.05779, 2024)
- Sarthi, P. et al. "RAPTOR" (ICLR 2024, arXiv:2401.18059)
- "KET-RAG: A Cost-Efficient Multi-Granular Indexing Framework for Graph-RAG" (SIGKDD 2025, arXiv:2502.09304)
- Chen, T. et al. "Dense X Retrieval" (EMNLP 2024, arXiv:2312.06648)
- Jiang, H. et al. "LLMLingua" (EMNLP 2023), "LongLLMLingua" (ACL 2024)
- Gutiérrez, B. et al. "HippoRAG" (arXiv:2405.14831)
- Anthropic, "Introducing Contextual Retrieval" (2024)
- Günther, M. et al. "Late Chunking" (Jina AI, 2024)
- QMSum (NAACL 2021), MeetingBank, ChunkRAG (arXiv:2410.19572)
