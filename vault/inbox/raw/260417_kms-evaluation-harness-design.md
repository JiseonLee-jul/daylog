# KMS Evaluation Harness 설계서

> 작성일: 2026-04-17
> 성격: KMS 파이프라인 평가 체계 설계 — 아키텍처 + 요구사항 충족
> 배경: KMS 파이프라인 설계서(260414_kms-pipeline-full-design.md) 기반

---

# 1. 목적

**KMS 기반 Agent Platform** 사업의 첫 단계로 KMS를 설계 중이다. 이 시스템은 단번의 설계로 완성할 수 없다. 설계하고, 실 데이터에 부딪히고, 문제를 발견하고, 컴포넌트를 뜯어 고치는 과정이 반복될 수밖에 없다.

따라서 핵심은 **문제를 빠르게 판단하고, 원인을 분석하고, 개선을 검증하는 루프**를 체계적으로 갖추는 것이다. 이 루프가 개발 전반과 운영 이후까지 지속적으로 작동해야 하며, 그 중심에 **평가 라이프사이클(Evaluation Harness)**이 있다.

이 평가 라이프사이클은 다음 요구사항을 만족해야 한다.

## 1.1 요구사항

**R1. 컴포넌트 독립성** — 파이프라인 컴포넌트가 교체되어도 평가 코어(Metric, Group, Registry, Loader)는 수정하지 않는다. Evaluator는 컴포넌트 출력과 Metric 사이의 번역 계층으로, 유일하게 수정이 허용되는 지점이다.

**R2. 평가 범위: 파이프라인 + 그래프** — 데이터가 수집되는 파이프라인의 각 단계와, 최종 산출물인 Knowledge Graph의 품질을 모두 다룬다.

**R3. 인터페이스 변화에 대한 내성** — 컴포넌트 입출력 필드는 자주 바뀌지만, 컴포넌트 분해 구조와 데이터 흐름은 안정적이다. 평가 체계는 필드 변화에는 유연하되, 흐름 구조에 결합한다.

**R4. 분산 실행 대응** — 파이프라인은 Celery 기반으로 분산 실행된다. 평가 체계는 단일 프로세스와 분산 환경 모두에서 일관되게 동작해야 한다.

**R5. 장기 확장: 관측·감사 기반** — 단기적으로는 개발 도구, 장기적으로는 운영 관측 및 감사 대시보드의 기반이 된다.

**R6. 회귀 판정과 재현성** — 결정적 메트릭(ChrF++, Entity F1 등)은 단일 실행으로 회귀를 차단한다. 비결정적 메트릭(LLM-as-Judge 등)은 N회 반복 + 통계적 판정으로 추세를 감시한다. Comparator 컴포넌트가 이를 담당한다.

---

# 2. 전체 아키텍처

본 설계는 **논리 계층(수직)**과 **횡단 인프라(수평)** 두 축으로 구성된다. 수직축은 "무엇을 하는가", 수평축은 "어디서 어떻게 돌아가는가".

```
            수직축 (논리 계층)

      ┌──────────────────────────┐
      │ L3  Continuous Evaluation │  언제 평가하는가 (Local/CI/Nightly/Shadow)
      ├──────────────────────────┤
      │ L2  Evaluation Harness   │  무엇을 측정하는가 (Evaluator/Metric/Group)
      ├──────────────────────────┤
      │ L1  Pipeline             │  비즈니스 로직 (Component Base Class)
      └──────────────────────────┘
          ▲       ▲       ▲       ▲
          │       │       │       │       수평축 (횡단 인프라)
      ┌───┴───┬───┴───┬───┴───┬───┴───┐
      │ OTel  │Shared │Celery │ Trace │
      │Traces │Storage│Workers│Context│
      │+Metrics│      │       │       │
      ├───────┴───────┼───────┴───────┤
      │ L1 + L2 횡단   │   L1 전용     │
      └───────────────┴───────────────┘
```

### 수직축: 논리 계층

각 계층은 아래 계층에만 의존한다. 위 계층의 존재를 모른다.

- **L1 Pipeline** — 문서를 지식으로 변환하는 비즈니스 로직. Parser, Chunker, Summarizer, Embedder, EntityExtractor, GraphRegistrar 등의 컴포넌트로 구성.
- **L2 Evaluation Harness** — L1의 산출물(artifact)과 실행 흔적(span)을 읽어 품질을 측정. Evaluator, Metric, Group으로 구성되며, 평가 정의는 YAML 선언.
- **L3 Continuous Evaluation** — L2를 언제, 어떤 범위로 실행할지를 결정하는 운영 계층. Local(수동) / CI(PR) / Nightly(스케줄) / Shadow(프로덕션) 네 가지 모드.

### 수평축: 횡단 인프라

횡단 범위에 따라 두 그룹으로 나뉜다.

**L1 + L2 횡단:**
- **OpenTelemetry** — L1은 실행 흔적(Traces)을, L2는 평가 결과(Metrics)를 OTel로 방출. 벤더 중립 OTLP 프로토콜로 어떤 백엔드든 수신 가능.
- **Shared Storage** — L1이 artifact를 저장하고, L2가 이를 조회. 별도 평가 저장소 없음. (S3/MinIO)

**L1 전용:**
- **Task Queue** — 파이프라인 워커의 분산 실행 (Celery + Broker). 평가 워커는 Celery에 종속되지 않는다.
- **Trace Propagation** — 파이프라인 워커 간 프로세스 경계에서 trace를 연결 (CeleryInstrumentor). 평가 워커는 별도 trace로 실행되므로 해당 없음.

---

# 3. 아키텍처 상세

2장의 3계층 구조를 구성요소 수준으로 펼친 전체 흐름.

```
═══════════════════════════════════════════════════════════════════
 L1  Pipeline (파이프라인 워커)
═══════════════════════════════════════════════════════════════════

 [Seed Corpus]
      │
      ▼
 Stage 1 (Ingest) ───▶ Stage 2 (Parse) ───▶ Stage 3 (Index)
 ┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
 │ FileStore     │     │ Parser       │     │ PreValidator     │
 │ DedupChecker  │     │ Chunker      │     │ Summarizer  ─┐   │
 │ DepthResolver │     │              │     │ Embedder     ├병렬│
 └──────┬───────┘     └──────┬───────┘     │ EntityExt.  ─┘   │
        │                    │              │ PostValidator     │
        │                    │              │ GraphRegistrar    │
        │                    │              └──────┬───────────┘
        │                    │                     │
        ▼                    ▼                     ▼

     각 Component의 __call__ 실행 시, 두 경로가 동시에 갈라짐:

              ┌────────────────┴────────────────┐
              │ ① artifact 경로                  │ ② span 경로
              ▼                                  ▼
 ┌──────────────────────────┐    ┌──────────────────────────┐
 │  Artifact 저장             │    │  OTel Traces 방출         │
 │  (파이프라인 기존 저장소)    │    │  (Base Class가 자동 계측)  │
 │                          │    │                          │
 │  S3/NFS:   body.md       │    │  artifact 경로를           │
 │  Postgres: chunks        │    │  span 속성에 기록          │
 │  GraphDB:  nodes, edges  │    │                          │
 └────────────┬─────────────┘    └────────────┬─────────────┘
              │                                │
              │                           OTLP ──→ 백엔드
              │                                    (벤더 무관)
              │                                │
              └──────────────┬─────────────────┘
                             │
                     파이프라인 워커 완료
                     (여기서 L1 프로세스 끝)
                             │
─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
                             │ 비동기 트리거
                             │ (task chain / 스케줄 / 수동)
                             ▼
═══════════════════════════════════════════════════════════════════
 L2  Evaluation Harness (평가 워커, 별도 프로세스)
═══════════════════════════════════════════════════════════════════

   ① artifact              ② span                golden/
   S3/DB/GraphDB에서        OTLP 백엔드 API로       (레퍼런스)
   ID로 조회                trace_id로 조회
        │                        │                    │
        ▼                        ▼                    ▼
   ArtifactLoader           APISpanLoader        GoldenLoader
        │                        │                    │
        └────────────────────────┼────────────────────┘
                                 │
                                 ▼
                         Evaluator (YAML + Registry)
                         ┌───────┴────────┐
                         ▼                ▼
                   결과물 품질         실행 품질
                   ParserEval         LatencyEval
                   ChunkerEval        CostEval
                   EntityEval
                   GraphEval
                         └───────┬────────┘
                                 │
                                 ▼
                         Group 집계 (YAML)
                                 │
                                 ▼
                         OTel Metrics 방출  ──→ OTLP ──→ 백엔드
                         (평가 결과도 벤더 중립)

═══════════════════════════════════════════════════════════════════
 L3  Continuous Evaluation
═══════════════════════════════════════════════════════════════════

       L2 평가 워커를 언제, 어떤 범위로 트리거하는가

      Local ───── CI ───── Nightly ───── Shadow
      (수동)     (PR)     (스케줄)     (프로덕션 샘플링
                  │                     → 골든셋 성장)
                  │
               회귀 감지 시
               배포 차단
```

**Observe First, Evaluate Async**: L1 파이프라인 워커는 artifact 저장 + OTel Traces 방출만 하고 즉시 완료한다. L2 평가 워커는 **별도 프로세스**에서 비동기로 실행되며, artifact와 span을 ID 기반 API 조회로 수집한다.
**두 경로의 분기**: Component의 `__call__`이 실행되면 ① 결과물은 기존 저장소에, ② 실행 흔적은 OTel Traces로 **동시에** 나간다.
**OTel의 벤더 중립 원칙**: L1은 Traces를, L2는 Metrics를 OTel로 방출한다. 둘 다 OTLP로 내보내므로, 수신 백엔드를 자유롭게 선택·교체할 수 있다.
**OTLP 백엔드의 3가지 역할**: ① L1 trace의 싱크(sink) ② L2가 span을 조회하는 소스(source) ③ L2 평가 결과의 싱크(sink).

---

# 4. L1: 파이프라인

## 4.1 컴포넌트 맵

모든 컴포넌트는 **Component Base Class**를 상속한다. `__call__`이 OTel 계측을 은닉하고, 서브클래스는 `_run()`만 구현한다. LLM 호출 컴포넌트(*)는 `LLMComponent`를 상속하여 `gen_ai.*` 속성을 자동 기록.

```
 Stage 1 (보관) ────────▶ Stage 2 (읽기) ────────▶ Stage 3 (이해)
┌──────────────┐        ┌──────────────┐        ┌──────────────────┐
│              │        │              │        │                  │
│ FileStore     │        │ Parser       │        │ PreValidator     │
│  원본 파일 저장 │        │  파일→마크다운  │        │  입력 무결성 검증  │
│              │        │              │        │   │              │
│ DedupChecker  │        │ Chunker      │        │   ▼              │
│  SHA256 중복   │        │  마크다운→청크  │        │ Summarizer*  ─┐  │
│              │        │              │        │ Embedder      ├병렬│
│ DepthResolver │        │              │        │ EntityExt.*  ─┘  │
│  depth/review │        │              │        │   │              │
│              │        │              │        │   ▼              │
│              │        │              │        │ PostValidator    │
│              │        │              │        │  출력 정합성 검증  │
│              │        │              │        │   │              │
│              │        │              │        │   ▼              │
│              │        │              │        │ GraphRegistrar   │
│              │        │              │        │  그래프 DB 등록    │
│              │        │              │        │                  │
└──────────────┘        └──────────────┘        └──────────────────┘
```

**안정적인 것**: 이 컴포넌트 분해와 흐름 방향. 새 Stage가 추가되지 않는 한 유지.
**변동 가능한 것**: 각 컴포넌트의 입출력 필드 정의. 구현하면서 바뀜.

## 4.2 Component Base Class

모든 파이프라인 컴포넌트가 상속하는 베이스. **OTel 계측을 은닉**하여 서브클래스에 OTel 코드가 없게 한다.

```python
class Component(ABC, Generic[TInput, TOutput]):
    component_name: str
    component_version: str = "unknown"

    def __call__(self, input: TInput) -> TOutput:
        with tracer.start_as_current_span(self.component_name) as span:
            span.set_attribute("kms.component", self.component_name)
            span.set_attribute("kms.component.version", self.component_version)
            self._set_input_attributes(span, input)
            result = self._run(input)
            self._set_output_attributes(span, result)
            return result

    @abstractmethod
    def _run(self, input: TInput) -> TOutput: ...
```

### Component vs LLMComponent

LLM 호출이 있는 컴포넌트는 `LLMComponent`를 상속한다. 차이는 **span에 기록되는 속성**과 그로 인해 **평가 가능한 차원**이 달라지는 것이다.

```
Component (계측 + 에러처리)
  ├── Parser, Chunker, Embedder, Validators, GraphRegistrar
  │
  └── LLMComponent (Component 상속 + GenAI 계측 추가)
        ├── Summarizer
        └── EntityExtractor
```

| | Component | LLMComponent |
|---|---|---|
| **span 속성** | `kms.*` | `kms.*` + `gen_ai.*` |
| **추가 기록** | — | 모델명, 토큰 수, 비용 |
| **OTLP 수신 도구** | 일반 span으로 표시 | `gen_ai.*`를 인식하여 LLM 호출로 해석 |
| **평가 가능 차원** | 지연시간, 성공/실패 | + LLM 비용, 토큰 사용량 |

LLMComponent는 `_llm_call()` 래퍼를 제공하여 LLM 호출마다 **자식 span**을 자동 생성한다. Summarizer가 L0/L1 요약을 각각 호출하면 자식 span이 2개 붙는다.

## 4.3 컴포넌트별 입출력 계약

필드 정의는 변동 가능하므로, **각 컴포넌트가 무엇을 받아 무엇을 만들고 어디에 남기는가** 수준만 확정한다.

**Stage 1 (보관)**
- **FileStore** — 원본 파일을 받아 그대로 저장한다. SHA256을 계산한다. → S3
- **DedupChecker** — SHA256으로 이미 처리된 파일인지 판별한다. → PostgreSQL 조회
- **DepthResolver** — 메타데이터와 규칙으로 processing_depth, requires_review를 결정한다. 파일 내용을 읽지 않는다.

**Stage 2 (읽기)**
- **Parser** — 원본 파일을 마크다운으로 변환한다. 파일 형식에 따라 파서를 선택한다 (Docling, Whisper 등). → S3 (body.md)
- **Chunker** — 마크다운을 의미 단위 청크로 분할한다. 문서 타입·구조·크기에 따라 전략을 결정한다. → PostgreSQL (chunks)

**Stage 3 (이해)**
- **PreValidator** — 청크 구조, FK 정합성 등 입력 무결성을 검증한다. 비싼 연산 전 관문. 데이터를 수정하지 않는다.
- **Summarizer*** — 마크다운으로부터 L0/L1 요약을 생성한다. LLM 호출. → Graph DB
- **Embedder** — 텍스트를 벡터로 변환한다. → PostgreSQL (chunks.embedding)
- **EntityExtractor*** — 마크다운과 청크로부터 entity와 relation을 추출한다. LLM 호출. → Graph DB
- **PostValidator** — 임베딩 차원, entity 참조 유효성 등 출력 정합성을 검증한다. 등록 전 관문.
- **GraphRegistrar** — 위 산출물 전체를 그래프 DB에 트랜잭션 단위로 등록한다. → Graph DB

\* = LLMComponent 상속 (gen_ai.* 속성 자동 기록)

## 4.4 파이프라인 오케스트레이션

컴포넌트를 **어떤 순서로 호출하는가**를 정의하는 계층. 이 호출 구조가 OTel span 트리의 형태를 결정한다.

```python
def kms_ingest(source_record):
    with tracer.start_as_current_span("kms_ingest"):  # 최상위 span
        stored = file_store(source_record)
        if dedup_checker(stored).is_duplicate:
            return
        depth = depth_resolver(stored)

        parsed = parser(stored)
        chunks = chunker(parsed)

        pre_validator(chunks)

        # 병렬 실행
        summary = summarizer(chunks)
        embeddings = embedder(chunks)
        entities = entity_extractor(chunks)

        post_validator(summary, embeddings, entities)
        graph_registrar(summary, embeddings, entities)
```

이 코드가 만드는 **span 트리**:

```
kms_ingest                          ← 오케스트레이션이 만드는 최상위
  ├── file_store                    ← 각 Component.__call__이 만드는 자식
  ├── dedup_checker
  ├── depth_resolver
  ├── parser
  ├── chunker
  ├── pre_validator
  ├── summarizer                    ← LLMComponent
  │     ├── llm_call (L0)           ← _llm_call이 만드는 손자
  │     └── llm_call (L1)
  ├── embedder
  ├── entity_extractor              ← LLMComponent
  │     └── llm_call
  ├── post_validator
  └── graph_registrar
```

span 트리는 OTel이 강제하는 것이 아니라, **`start_as_current_span`의 중첩 위치**로 우리가 설계하는 것이다. 오케스트레이션 코드가 호출 순서를 바꾸면 트리도 바뀐다.

---

# 5. L2: 평가 체계 (Evaluation Harness)

## 5.1 구조

R1(컴포넌트 독립성)과 R3(인터페이스 변화 내성)을 만족하려면, 평가 정의가 코드에 박혀 있으면 안 된다. 컴포넌트가 바뀔 때마다, 메트릭을 추가할 때마다 코어 코드를 수정해야 한다면 요구사항에 반한다.

EleutherAI의 lm-evaluation-harness(12K+ stars)는 이 문제를 **"평가 정의 = 데이터(YAML) + 메트릭(plugin) + 집계(group)"의 3분리**로 해결했다. 60개 이상의 벤치마크를 코어 무수정으로 흡수할 수 있었던 이유가 이 구조다. 본 설계는 이 패턴을 차용하되, 평가 대상을 LLM 모델이 아닌 **KMS 파이프라인 컴포넌트**에 맞게 재설계한다.

```
kms/eval/
  api/
    evaluator.py          Evaluator 추상 클래스 + @register_evaluator
    metric.py             Metric 추상 클래스 + @register_metric
    registry.py           동적 등록 시스템
    group.py              집계 로직

  evaluators/             결과물 품질
    parser.py               @register_evaluator("parser")
    chunker.py
    entity_extractor.py
    graph_structure.py
                          실행 품질
    span_latency.py
    span_cost.py

  metrics/
    chrf_plus.py            @register_metric("chrf_plus")
    teds.py
    window_diff.py
    entity_f1.py
    isolated_node_ratio.py
    p95_latency.py
    token_cost.py

  loaders/
    artifact_loader.py      ArtifactLoader 추상 + S3/Local 구현
    span_loader.py          SpanLoader 추상 + InMemory/API 구현
    golden_loader.py        GoldenLoader (golden/ 디렉토리에서 읽기)

  preprocess/
    normalize_whitespace.py
    strip_frontmatter.py

  loggers/
    otel_logger.py          평가 결과를 OTel Metrics로 방출

  configs/
    evaluators/*.yaml       평가 정의
    groups/*.yaml            집계 정의

  golden/                   Seed Corpus + Golden Reference (입력 파일과 기대 출력물을 함께 관리)
```

## 5.2 5계층 책임 분리

하나의 평가가 실행될 때 거치는 5개 계층. 각 계층은 독립적으로 교체 가능.

```
① DATA          golden/, artifact(S3/DB), span(OTLP 백엔드/InMemory)
     │
     ▼
② PREPROCESS    artifact 정규화 (BOM, whitespace)
                golden 정규화 (frontmatter 제거)
     │
     ▼
③ EVALUATION    Evaluator가 어떤 Metric을 부를지, 데이터를 어떻게 가공해서 넘길지 결정
     │
     ▼
④ METRIC        실제 점수 계산 (artifact + golden → float)
     │
     ▼
⑤ AGGREGATION   파일별 점수 → evaluator 집계 → group 집계
                → OTel Metrics로 방출 (OTLP → 어디든)
```

## 5.3 YAML 평가 정의

YAML만으로 평가를 정의할 수 있는 이유는 **Registry 패턴** 때문이다. `@register_evaluator("parser")`와 `@register_metric("chrf_plus")`가 클래스를 이름으로 등록하고, YAML은 그 이름을 문자열로 참조한다. 실행 시 YAML을 읽고 → Registry에서 이름으로 클래스를 찾아 → 인스턴스화한다.

코드 수정 없이 YAML 한 장으로 새 평가를 추가한다.

```yaml
# configs/evaluators/parser_pdf.yaml
evaluator: parser
target_component: Parser
golden_glob: golden/pdf_*
artifact_preprocess: !function preprocess.normalize_whitespace
golden_preprocess: !function preprocess.strip_frontmatter
metric_list:
  - metric: chrf_plus
    aggregation: mean
    higher_is_better: true
  - metric: teds_table
    aggregation: mean
    higher_is_better: true
metadata:
  version: 1.0
```

```yaml
# configs/groups/stage2_parse.yaml
group: stage2_parse
evaluators:
  - parser_pdf
  - parser_xlsx
  - chunker_default
aggregate_metric_list:
  - metric: chrf_plus
    aggregation: mean
    weight_by_size: true
```

## 5.4 Evaluator의 역할

Evaluator는 **점수를 계산하지 않는다**. 각 Metric에 맞게 데이터를 가공하여 넘기는 **지휘자**다.

```python
@register_evaluator("parser")
class ParserEvaluator(Evaluator):
    def evaluate(self, artifact_md, golden_md, span):
        return {
            # 전체 텍스트 비교
            "chrf_plus": chrf_plus.compute(artifact_md, golden_md),

            # 표만 추출하여 비교
            "teds_table": teds_table.compute(
                extract_tables(artifact_md),
                extract_tables(golden_md)
            ),

            # span 기반 실행 품질
            "latency": p95_latency.compute(span),
        }
```

Metric은 재사용 가능 — `chrf_plus`를 다른 Evaluator가 불러서도 쓸 수 있다.

## 5.5 Loader 추상화

Evaluator는 저장소 위치를 모른다. 추상화된 Loader를 통해 읽는다.

| Loader | 읽는 곳 | 용도 |
|---|---|---|
| **ArtifactLoader** | S3, PostgreSQL, Graph DB | 파이프라인 산출물 (ID 기반 조회) |
| **SpanLoader** | OTLP 백엔드 API 또는 InMemory | 실행 흔적 (trace_id 기반 조회) |
| **GoldenLoader** | golden/ 디렉토리 | 기대 출력물 (사람이 작성) |

평가 워커는 파이프라인 워커와 **별도 프로세스**에서 실행되므로, SpanLoader는 실행 환경에 따라 구현이 달라진다:

| SpanLoader 구현 | 데이터 접근 | 사용 모드 |
|---|---|---|
| **InMemorySpanLoader** | 프로세스 내 메모리 | Local 전용 (단일 프로세스) |
| **APISpanLoader** | OTLP 백엔드 API (trace_id 기반) | CI / Nightly / Shadow |

Evaluator 코드는 어느 구현이든 무수정. Loader만 교체.

## 5.6 Golden Reference

Golden은 "예상 지표값"이 아니라 **기대하는 출력물 자체**다. 지표는 비교 결과로 계산된다.

```
golden/pdf_report/
  input.pdf                    ← 입력 파일 (Seed Corpus)
  expected.md                  ← 기대 출력 (사람이 작성)

     │
     ▼ input.pdf → Pipeline 실행 → artifact (body_md)
     ▼ artifact vs expected.md 비교
     
chrf_plus = 0.92               ← 계산된 지표값
```

동일 대상의 입력과 기대 출력을 한 디렉토리에서 관리한다.

## 5.7 확장 시 코드 변경 범위

| 변화 | 수정 위치 | 코어 수정 |
|---|---|---|
| 새 메트릭 추가 | `metrics/` 파일 + YAML 한 줄 | 없음 |
| 새 컴포넌트 평가 | `evaluators/` 파일 + YAML | 없음 |
| 메트릭 교체 | YAML metric_list 변경 | 없음 |
| 평가 조합 변경 | `groups/` YAML | 없음 |
| 골든 세트 추가 | `golden/` 디렉토리에 파일 추가 | 없음 |

---

# 6. L3: 운영 (Continuous Evaluation)

## 6.1 4개 모드

동일한 L2 Harness를 **다른 트리거와 범위**로 실행한다.

| 모드 | 트리거 | 범위 | 목적 | 소요 시간 |
|---|---|---|---|---|
| **Local** | 개발자 수동 | 소규모 서브셋 | 개발 중 빠른 확인 | 수 초 ~ 수 분 |
| **CI** | PR/Push | 중간 (10~20 파일) | 회귀 차단 (배포 게이트) | 5~20분 |
| **Nightly** | cron 스케줄 | 전체 corpus + 비싼 메트릭 | drift 감지 | 30분 ~ 수 시간 |
| **Shadow** | 프로덕션 트래픽 | 실 데이터 샘플링 | 실환경 검증 + 골든셋 성장 | 상시 |

```bash
# Local
python -m kms.eval.run --evaluator parser_pdf

# CI (GitHub Actions)
python -m kms.eval.run --group ci_smoke
python -m kms_eval.check_regression --baseline main

# Nightly (cron)
python -m kms.eval.run --group full_pipeline

# Shadow
# 프로덕션 파이프라인에서 저품질 span 자동 수집 → 골든셋 후보 큐
```

**MVP에서의 우선순위**: Local만으로 시작 → CI 추가 → Nightly → Shadow 순으로 점진 확장.

## 6.2 Data Flywheel (Shadow 모드의 핵심)

Shadow가 가동되면 골든셋이 **자동으로 성장**하는 루프가 형성된다.

```
프로덕션 데이터 유입
     │
     ▼
파이프라인 실행 + OTel Traces 방출
     │
     ▼
저품질 감지 (LLM-judge score < 임계값)
     │
     ▼
골든셋 후보 큐에 자동 등록
     │
     ▼
사람 검토 → 골든셋 편입
     │
     ▼
다음 CI/Nightly 평가에 반영
```

---

# 7. 수평축: 횡단 인프라

특정 계층에 속하지 않고, L1·L2·L3 전체를 관통하는 공유 인프라.

## 7.1 OpenTelemetry

OTel은 **벤더 중립 관측 규격**이다. 관측 도구가 아니라 관측 도구들의 **공용 프로토콜(OTLP)**이며, L1과 L2 모두가 사용한다.

**L1이 방출하는 것: Traces (span)**
- 파이프라인 실행 흔적 (소요시간, 상태, artifact 경로)
- Component Base Class의 `__call__`에서 자동 생성

**L2가 방출하는 것: Metrics (평가 결과)**
- 평가 점수 (ChrF++ = 0.91, Entity F1 = 0.78 등)
- Group 집계 후 OTel Metrics로 방출

```
L1 Pipeline  ──→ OTel Traces  ──→ OTLP ──→ 어디든
L2 Evaluator ──→ OTel Metrics ──→ OTLP ──→ 어디든
                                    │
                                    ▼
                           Langfuse, MLflow,
                           Prometheus, Grafana,
                           또는 미래의 어떤 도구든
```

**수신 백엔드는 선택사항이다.** Langfuse, MLflow는 현재 권장 도구일 뿐, OTLP를 지원하는 어떤 도구든 교체 가능.

### Semantic Convention

span/metric에 기록하는 속성의 네임스페이스 규약. **OTel 공식 속성을 우선 사용하고, 공식에 없는 개념만 커스텀(`kms.*`)으로 정의한다.** 공식 속성명을 쓰면 수신 도구(Langfuse, Grafana 등)가 자동 해석하지만, 커스텀 이름은 raw key-value로만 표시된다.

**공통 속성 (모든 span에 기록)**

| 속성 | 출처 | 예시 | 용도 |
|---|---|---|---|
| `service.version` | OTel 공식 | `"a3f2c1"` (git hash) | 파이프라인 전체 버전 |
| `deployment.environment.name` | OTel 공식 | `"production"` | prod/staging/dev 환경 분리 |
| `user.id` | OTel 공식 | `"jiseon"`, `"slack-connector"` | 누가 트리거했는가 |
| `session.id` | OTel 공식 | `"ingest-batch-20260417"` | 세션/배치 단위 분석 |
| `error.type` | OTel 공식 | `"TimeoutError"` | 실패 시 오류 유형 |
| `kms.component` | 커스텀 | `"parser"`, `"chunker"` | 어떤 컴포넌트인가 |
| `kms.component.version` | 커스텀 | `"docling-1.3"` | 컴포넌트 구현/버전 |
| `kms.artifact_path` | 커스텀 | `"s3://...body.md"` | 산출물 저장 경로 |
| `kms.status` | 커스텀 | `"completed"`, `"skipped"` | 처리 상태 (성공 포함) |

**LLM 호출 속성 (LLMComponent span에만 추가)**

| 속성 | 출처 | 예시 | 용도 |
|---|---|---|---|
| `gen_ai.system` | OTel 공식 | `"anthropic"` | LLM 제공자 |
| `gen_ai.request.model` | OTel 공식 | `"claude-haiku"` | 요청 모델명 |
| `gen_ai.operation.name` | OTel 공식 | `"chat"` | 작업 종류 (chat/embed) |
| `gen_ai.request.temperature` | OTel 공식 | `0.7` | 재현성 추적 |
| `gen_ai.usage.input_tokens` | OTel 공식 | `1200` | 입력 토큰 → 비용 계산 |
| `gen_ai.usage.output_tokens` | OTel 공식 | `350` | 출력 토큰 → 비용 계산 |
| `gen_ai.data_source.id` | OTel 공식 | `"src_001"` | 처리 대상 소스 식별 |

### OTel 관련 코드의 위치

observability는 플랫폼 공유 인프라에 위치하며, KMS의 pipeline과 eval이 이를 참조한다.

```
platform/shared/observability/       # 플랫폼 공유 인프라
  telemetry.py                       # TracerProvider + MeterProvider 초기화
  semconv.py                         # 속성 규약 상수 정의

kms/pipeline/
  base.py                            # Component.__call__에서 span 생성 (L1 계측 지점)

kms/eval/
  loggers/
    otel_logger.py                   # 평가 결과를 OTel Metrics로 방출 (L2 계측 지점)
```

## 7.2 Celery (L1 파이프라인 분산 실행)

### Component / Task 분리

Component는 **실행 환경을 모르는 순수 로직**이고, Celery task는 **"어떤 워커에서 실행할지"만 결정하는 얇은 래퍼**다. 이 분리 덕분에 같은 코드가 로컬과 분산 모두에서 동작한다.

```python
# Component: 순수 로직 (Celery를 모름)
class Parser(Component[SourceRecord, ParseResult]):
    component_name = "parser"
    def _run(self, source_record):
        return docling.convert(source_record.raw_path)

# Celery Task: 오케스트레이션만
@shared_task(bind=True, max_retries=3)
def parse_task(self, source_id: str):
    source_record = load_source_record(source_id)
    return _parser(source_record)
```

```
로컬:   parser(source_record)           ← Component 직접 호출
분산:   parse_task.delay(source_id)      ← Celery가 워커 배정, 내부에서 같은 Component 호출
```

Celery는 **L1 파이프라인 워커의 분산 실행 인프라**다. 평가 워커(L2)는 Celery에 종속되지 않으며, 단순 스크립트/CI job/cron 등 어떤 방식으로든 실행 가능하다.

### Trace Propagation (L1 내부 문제)

파이프라인 워커 간 프로세스 경계에서 trace가 끊기는 문제. `CeleryInstrumentor`가 task 메시지 헤더에 trace context를 자동 삽입하여 해결한다.

```python
# platform/shared/observability/telemetry.py 초기화 시 한 줄
CeleryInstrumentor().instrument()
```

평가 워커는 파이프라인 trace를 이어받지 않는다. 과거 trace를 **API로 조회**할 뿐이므로 Trace Propagation과 무관하다.

## 7.3 Shared Storage

분산 워커 간 artifact 공유. 별도 평가 저장소를 두지 않고 파이프라인 기존 저장소를 읽는다.

| 단계 | 저장소 |
|---|---|
| MVP | NFS (공유 파일시스템) |
| 운영 | S3/MinIO |

평가 실행 간 격리는 **run_id 접두사**로 해결. 덮어쓰기 방지.

---

# 8. 요구사항 충족

1장에서 정의한 R1–R5를, 어떤 설계 결정으로 만족시키는가. 그리고 **왜 그것이 가능한가**.

---

### R1. 컴포넌트 독립성

> 컴포넌트가 교체되어도 평가 코드는 수정하지 않는다.

**가능한 이유**: 평가 코드가 컴포넌트의 **구현**을 보지 않고 **계약(인터페이스)**만 보기 때문이다. 이 계약이 안정적인 한, 구현이 무엇이든 평가는 동일하게 동작한다.

| 설계 결정 | 왜 이것이 독립성을 보장하는가 |
|---|---|
| **Component Base Class** (4장) | 모든 컴포넌트가 동일한 `__call__` → `_run()` 계약을 따른다. Evaluator는 이 계약의 출력만 본다. Parser를 Docling에서 Marker로 교체해도 `_run()`의 반환 타입이 같으면 Evaluator 입장에서 바뀐 게 없다. |
| **Loader 추상화** (5장) | Evaluator는 "source_id로 artifact를 달라"고 요청할 뿐, 그것이 S3에 있는지 로컬에 있는지 모른다. 컴포넌트가 저장 방식을 바꿔도 Loader 구현만 교체하면 Evaluator는 무수정. |
| **YAML 바인딩** (5장) | Evaluator와 컴포넌트의 연결이 코드가 아닌 설정 파일이다. 새 컴포넌트를 추가하면 YAML 한 장을 쓰면 된다. 기존 Evaluator 코드를 열 필요가 없다. |
| **Metric Registry** (5장) | `@register_metric`으로 등록된 Metric은 어떤 Evaluator에서든 이름으로 호출 가능. 새 메트릭을 추가해도 기존 메트릭이나 Evaluator를 건드리지 않는다. |

---

### R2. 평가 범위: 파이프라인 + 그래프

> 파이프라인 각 단계와 Knowledge Graph 품질을 모두 다룬다.

**가능한 이유**: Evaluator가 **두 종류의 입력**(artifact + span)을 받을 수 있고, 컴포넌트 단위로 분리되어 있어 파이프라인 중간 단계와 최종 그래프를 **같은 구조로** 평가할 수 있기 때문이다.

| 설계 결정 | 왜 이것이 범위 확장을 가능하게 하는가 |
|---|---|
| **컴포넌트별 Evaluator** (5장) | 파이프라인의 각 단계가 독립 컴포넌트이므로, 각각에 대응하는 Evaluator를 붙일 수 있다. Parser만 평가하거나 전체를 평가하거나, 조합이 자유롭다. |
| **두 차원 평가** (3장) | OTel이 실행 흔적(Traces)을 자동 방출하므로, Evaluator는 "결과물이 정확한가"(artifact)와 "과정이 건강한가"(span)를 하나의 run에서 동시에 측정할 수 있다. OTel이 없었다면 실행 품질은 별도 시스템을 구축해야 했다. |
| **Group 집계** (5장) | YAML로 Evaluator를 Stage 단위 / 전체 단위로 묶을 수 있다. 그래프 평가(GraphStructureEval)도 같은 Group 구조에 편입되므로 파이프라인 평가와 그래프 평가가 하나의 리포트에 합쳐진다. |

---

### R3. 인터페이스 변화에 대한 내성

> 필드 변화에는 유연하되, 흐름 구조에 결합한다.

**가능한 이유**: 평가 체계가 결합하는 지점이 **개별 필드가 아니라 컴포넌트 분해 구조**이기 때문이다. 필드가 바뀌면 영향을 받는 곳이 한 곳(Evaluator의 가공 로직)으로 국한된다.

| 설계 결정 | 왜 이것이 변화 내성을 주는가 |
|---|---|
| **Evaluator = 지휘자** (5장) | Evaluator는 "artifact에서 표만 뽑아서 TEDS에 넘긴다" 같은 가공을 담당한다. 필드가 바뀌면 이 가공 로직만 수정하면 된다. Metric(TEDS 계산) 자체는 "두 표를 받아서 점수를 낸다"이므로 필드 변화와 무관하다. |
| **Preprocess 분리** (5장) | 데이터 정규화(BOM 제거, frontmatter 제거)가 Evaluator/Metric 밖에 있다. 포맷이 바뀌면 preprocess 함수만 수정. Metric은 정규화된 데이터만 받으므로 영향 없다. |
| **YAML version 필드** (5장) | 평가 정의 자체가 바뀌면(메트릭 추가, 기준 변경) 버전을 올린다. 다른 버전의 결과끼리 비교하면 경고가 뜬다. 인터페이스 변경으로 인한 오탐 regression을 방지한다. |

---

### R4. 분산 실행 대응

> 단일 프로세스와 분산 환경 모두에서 일관 동작한다.

**가능한 이유**: 파이프라인 워커(L1)와 평가 워커(L2)가 **애초에 별도 프로세스**이며, 각각이 실행 환경을 모르는 구조이기 때문이다.

**L1 파이프라인 — Celery로 분산, Component는 환경을 모름:**

| 설계 결정 | 왜 이것이 분산 대응을 가능하게 하는가 |
|---|---|
| **Component / Task 분리** (7장) | Component는 입력을 받아 출력을 반환하는 순수 로직이다. Celery task는 "어떤 워커에서 이 Component를 호출할지"만 결정하는 얇은 래퍼. 로컬에서는 task 없이 Component를 직접 호출하면 된다. |
| **CeleryInstrumentor** (7장) | 파이프라인 워커 간 프로세스 경계에서 trace context를 자동 전파한다. 분산 환경에서도 하나의 trace로 전체 파이프라인을 추적할 수 있다. 이 문제는 L1 내부에 한정된다. |

**L2 평가 — Celery와 무관, API 조회로 데이터 접근:**

| 설계 결정 | 왜 이것이 분산 대응을 가능하게 하는가 |
|---|---|
| **Observe First, Evaluate Async** (3장) | 파이프라인이 artifact + traces를 남기고 완료하면, 평가 워커가 별도 프로세스에서 비동기로 실행된다. 파이프라인 trace를 이어받는 것이 아니라 과거 trace를 API로 조회할 뿐이므로 Trace Propagation 문제가 발생하지 않는다. |
| **Loader 추상화** (5장) | 로컬에서는 `InMemorySpanLoader`, 분산에서는 `APISpanLoader`. Evaluator 코드는 동일. 환경 차이가 Loader 한 곳에 캡슐화된다. |
| **Shared Storage** (7장) | 파이프라인 워커가 저장한 artifact를 평가 워커가 S3/DB에서 읽는다. "어느 워커가 저장했는지"는 투명하다. |

---

### R5. 장기 확장: 관측·감사 기반

> 단기 개발 도구에서 장기 운영 관측성의 토대로 확장한다.

**가능한 이유**: OTel이 **횡단 인프라**이므로, L1과 L2 모두의 출력을 벤더 중립적으로 내보낸다. 인프라를 재구축하지 않고 수신 백엔드만 추가하면 새로운 응용이 가능하다.

| 설계 결정 | 왜 이것이 장기 확장을 가능하게 하는가 |
|---|---|
| **OTel = 횡단 인프라** (7장) | L1의 Traces와 L2의 Metrics가 모두 OTLP로 나간다. 운영기에 Grafana 대시보드를 추가하려면 OTLP 수신 설정만 하면 된다. 파이프라인 재계측도, 평가 코드 수정도 불필요. |
| **벤더 중립** (7장) | Langfuse/MLflow는 현재 선택일 뿐이다. OTLP를 지원하는 어떤 도구든 교체·추가 가능. 도구에 종속되지 않는다. |
| **4개 운영 모드** (6장) | Local → CI → Nightly → Shadow는 같은 Harness를 다른 트리거로 실행할 뿐이다. 개발기에 Local만 쓰다가 운영기에 Shadow를 켜도 평가 로직을 다시 작성하지 않는다. |
| **Data Flywheel** (6장) | Shadow 모드에서 프로덕션 실패 사례가 골든셋으로 자동 환류한다. 평가 체계가 운영 데이터와 함께 성숙하므로, 시간이 갈수록 평가의 신뢰도가 높아진다. |
