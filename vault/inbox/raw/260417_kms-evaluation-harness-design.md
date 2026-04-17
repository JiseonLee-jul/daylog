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

**R1. 컴포넌트 독립성** — 파이프라인 컴포넌트가 교체되거나 구현이 바뀌어도 평가 코드는 수정하지 않는다. 문제 컴포넌트를 쉽게 갈아끼울 수 있어야 한다.

**R2. 평가 범위: 파이프라인 + 그래프** — 데이터가 수집되는 파이프라인의 각 단계와, 최종 산출물인 Knowledge Graph의 품질을 모두 다룬다.

**R3. 인터페이스 변화에 대한 내성** — 컴포넌트 입출력 필드는 자주 바뀌지만, 컴포넌트 분해 구조와 데이터 흐름은 안정적이다. 평가 체계는 필드 변화에는 유연하되, 흐름 구조에 결합한다.

**R4. 분산 실행 대응** — 파이프라인은 Celery 기반으로 분산 실행된다. 평가 체계는 단일 프로세스와 분산 환경 모두에서 일관되게 동작해야 한다.

**R5. 장기 확장: 관측·감사 기반** — 단기적으로는 개발 도구, 장기적으로는 운영 관측 및 감사 대시보드의 기반이 된다.

---

# 2. 전체 아키텍처

본 설계는 **논리 계층(수직)**과 **분산 인프라(수평)** 두 축으로 구성된다. 수직축은 "무엇을 하는가", 수평축은 "어디서 어떻게 돌아가는가".

```
              수직축 (논리 계층)

        ┌──────────────────────────┐
        │ L4  Continuous Evaluation │  언제 평가하는가 (Local/CI/Nightly/Shadow)
        ├──────────────────────────┤
        │ L3  Evaluation Harness   │  무엇을 측정하는가 (Evaluator/Metric/Group)
        ├──────────────────────────┤
        │ L2  OpenTelemetry        │  실행 흔적을 남기는 인프라 (Span/Exporter)
        ├──────────────────────────┤
        │ L1  Pipeline             │  비즈니스 로직 (Component Base Class)
        └──────────────────────────┘
            ▲       ▲       ▲
            │       │       │         수평축 (분산 인프라)
     ┌──────┴───┬───┴───┬───┴──────┐
     │  Celery  │Shared │  Trace   │  모든 계층을 관통하는
     │  Workers │Storage│  Context │  횡단 관심사
     └──────────┴───────┴──────────┘
```

### 수직축: 논리 계층

각 계층은 아래 계층에만 의존한다. 위 계층의 존재를 모른다.

- **L1 Pipeline** — 문서를 지식으로 변환하는 비즈니스 로직. Parser, Chunker, Summarizer, Embedder, EntityExtractor, GraphRegistrar 등의 컴포넌트로 구성.
- **L2 OpenTelemetry** — 파이프라인 실행 흔적(span)을 표준화된 형태로 방출하는 관측 인프라. 평가·모니터링·비용추적 등 모든 응용의 공통 토대.
- **L3 Evaluation Harness** — L1의 산출물(artifact)과 L2의 실행 흔적(span)을 읽어 품질을 측정. Evaluator, Metric, Group으로 구성되며, 평가 정의는 YAML 선언.
- **L4 Continuous Evaluation** — L3를 언제, 어떤 범위로 실행할지를 결정하는 운영 계층. Local(수동) / CI(PR) / Nightly(스케줄) / Shadow(프로덕션) 네 가지 모드.

### 수평축: 분산 인프라

특정 계층에 속하지 않고, 4개 계층 전체를 관통하는 횡단 관심사.

- **Task Queue** — 파이프라인의 분산 실행 (Celery + Broker)
- **Shared Storage** — 분산 워커 간 artifact 공유 (S3/MinIO)
- **Trace Propagation** — 프로세스 경계를 넘는 trace 연결 (CeleryInstrumentor)

---

# 3. 아키텍처 상세

2장의 4계층 구조를 구성요소 수준으로 펼친 전체 흐름.

```
═══════════════════════════════════════════════════════════════════
 L1  Pipeline
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
 ┌─────────────────────────────────────────────────────────────┐
 │  Artifact 저장 (파이프라인 기존 저장소, 별도 평가 저장소 없음)   │
 │                                                             │
 │  S3/NFS:      /ingests/*, /transformed/*/body.md            │
 │  PostgreSQL:  source_records, chunks (embedding 포함)       │
 │  Graph DB:    doc_nodes, entity_nodes, edges                │
 └──────────────────────────┬──────────────────────────────────┘
                            │
                            │ ①
                            │
═══════════════════════════════════════════════════════════════════
 L2  OpenTelemetry
═══════════════════════════════════════════════════════════════════

 Component Base Class의 __call__이 span 자동 방출
 (artifact 경로/ID는 span 속성에 기록)

                     TracerProvider
                          │
            ┌─────────┬───┴────┬──────────┐
            ▼         ▼        ▼          ▼
        Langfuse   MLflow    File      Console
        (LLM trace (run 이력  (spans    (로컬
         프롬프트)   비교)     .jsonl)   디버깅)
            │         │        │
            │         │  ② ┌───┘
            │         │    │
═══════════════════════════════════════════════════════════════════
 L3  Evaluation Harness
═══════════════════════════════════════════════════════════════════

   ① artifact          ② span            golden/
   (S3/DB/GraphDB)     (File/MLflow)      (레퍼런스)
        │                   │                 │
        ▼                   ▼                 ▼
   ArtifactLoader       SpanLoader       GoldenLoader
        │                   │                 │
        └───────────────────┼─────────────────┘
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
                  ┌── MLflow Run ──────┐
                  │  Traces  (L2에서)   │
                  │  Metrics (L3에서)   │ ◀── 두 경로가 같은 run에 합류
                  │  Config  (버전/설정) │
                  └────────────────────┘
                            │
═══════════════════════════════════════════════════════════════════
 L4  Continuous Evaluation
═══════════════════════════════════════════════════════════════════

              동일한 L3를 다른 트리거로 실행

      Local ───── CI ───── Nightly ───── Shadow
      (수동)     (PR)     (스케줄)     (프로덕션 샘플링
                  │                     → 골든셋 성장)
                  │
               회귀 감지 시
               배포 차단
```

**① artifact 경로**: L1이 기존 저장소에 저장한 산출물을 L3가 Loader를 통해 읽음.
**② span 경로**: L2가 방출한 실행 흔적을 L3가 Loader를 통해 읽음.
**MLflow Run**: ①②의 평가 결과와 L2의 trace 데이터가 동일한 run에 합류하여, 결과물 품질과 실행 품질을 한 곳에서 비교.

---

# 4. L1: 파이프라인

## 4.1 컴포넌트 맵

```
Stage 1 (Ingest)        Stage 2 (Parse)        Stage 3 (Index)
보관                     읽기                    이해

FileStore               Parser                 PreValidator
  원본 파일 저장            파일 → 마크다운 변환      입력 무결성 검증 (비싼 연산 전 관문)
                                                  │
DedupChecker            Chunker                  ▼
  SHA256 중복 감지          마크다운 → 청크 분할     Summarizer ─┐
                                                 Embedder    ├ 병렬
DepthResolver                                    EntityExt. ─┘
  depth/review 결정                                │
                                                  ▼
                                                PostValidator
                                                  출력 정합성 검증 (등록 전 관문)
                                                  │
                                                  ▼
                                                GraphRegistrar
                                                  그래프 DB 등록 (트랜잭션)
```

**안정적인 것**: 이 컴포넌트 분해와 흐름 방향. 새 Stage가 추가되지 않는 한 유지.
**변동 가능한 것**: 각 컴포넌트의 입출력 필드 정의. 구현하면서 바뀜.

## 4.2 Component Base Class

모든 파이프라인 컴포넌트가 상속하는 베이스. **계측(L2)을 은닉**하여 서브클래스에 OTel 코드가 없게 한다.

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
            self._set_output_attributes(span, output)
            return result

    @abstractmethod
    def _run(self, input: TInput) -> TOutput: ...
```

LLM 호출이 있는 컴포넌트(Summarizer, EntityExtractor)는 `LLMComponent`를 상속하여 `gen_ai.*` 속성을 자동 기록.

```
Component (계측 + 에러처리)
  ├── Parser, Chunker, Embedder, Validators, GraphRegistrar
  │
  └── LLMComponent (GenAI 속성 추가)
        ├── Summarizer
        └── EntityExtractor
```

## 4.3 컴포넌트별 입출력 계약

필드는 변동 가능하므로, **각 컴포넌트가 무엇을 받아 무엇을 내보내는지**의 구조만 확정한다.

| 컴포넌트 | 입력 | 출력 | 저장소 |
|---|---|---|---|
| **FileStore** | file (bytes), metadata | raw_path, sha256, file_size | S3 |
| **DedupChecker** | sha256 | is_duplicate, existing_id | PostgreSQL 조회 |
| **DepthResolver** | metadata, rules | processing_depth, requires_review | 규칙 엔진 |
| **Parser** | raw_path, file_ext | body_md, parse_meta | S3 (body.md) |
| **Chunker** | body_md, parse_meta | chunks[] | PostgreSQL |
| **PreValidator** | source_record, chunks | is_valid, errors[] | — (read-only) |
| **Summarizer** | body_md, token_count, depth | l0_abstract, l1_overview | Graph DB |
| **Embedder** | texts[] | embeddings[] | PostgreSQL |
| **EntityExtractor** | body_md, chunks | entities[], relations[] | Graph DB |
| **PostValidator** | embeddings, entities, relations | is_valid, errors[] | — (read-only) |
| **GraphRegistrar** | 위 전부 | node_ids[], edge_ids[] | Graph DB |

## 4.4 Celery Task와의 관계

Component(로직)과 Celery task(오케스트레이션)는 분리한다. Task는 Component의 얇은 래퍼.

```python
# Component: 순수 로직
class Parser(Component[SourceRecord, ParseResult]):
    component_name = "parser"
    def _run(self, source_record):
        return docling.convert(source_record.raw_path)

# Task: 오케스트레이션만
@shared_task(bind=True, max_retries=3)
def parse_task(self, source_id: str):
    source_record = load_source_record(source_id)
    return _parser(source_record)
```

---

# 5. L2: 관측 인프라 (OpenTelemetry)

## 5.1 역할

OpenTelemetry는 **평가의 인프라 계층**이다. 파이프라인 실행 흔적(span)을 표준화된 형태로 방출하며, 그 위에 평가·모니터링·비용추적이 얹힌다. 관측 도구가 아니라 관측 도구들의 **공용 규격**이다.

```
            응용
   ┌────────┴────────┐
 평가     모니터링   비용추적
   └────────┬────────┘
          인프라
      [OpenTelemetry]
            │
       파이프라인 (L1)
```

## 5.2 계측 패턴

파이프라인 코드에 OTel 참조가 드러나지 않는다. 계측은 **Component Base Class의 `__call__`에 집중**되고, 서브클래스는 순수 로직만 구현한다 (4.2절 참조).

OTel 관련 코드의 위치:

```
kms/observability/
  telemetry.py     TracerProvider 초기화, Exporter 등록
  semconv.py       KMS 속성 규약 상수 정의

kms/pipeline/
  base.py          Component.__call__에서 span 생성 (유일한 계측 지점)
```

## 5.3 Semantic Convention

span에 기록하는 속성의 네임스페이스 규약. 모든 컴포넌트가 일관된 속성명을 사용해야 L3 Evaluator가 안정적으로 읽을 수 있다.

**`kms.*`**: 우리가 정의하는 애플리케이션 속성. OTel은 공식 네임스페이스와 충돌하지 않는 사용자 정의 속성을 허용한다.

**`gen_ai.*`**: OTel 공식 GenAI Semantic Convention. Langfuse 등 도구가 자동 인식.

**KMS 공통 (`kms.*`)**

| 속성 | 예시 | 용도 |
|---|---|---|
| `kms.component` | `"parser"` | 어떤 컴포넌트인가 |
| `kms.component.version` | `"docling-1.3"` | 어떤 구현/버전인가 |
| `kms.source_id` | `"src_001"` | 어떤 문서를 처리했는가 |
| `kms.artifact_path` | `"s3://...body.md"` | 산출물 위치 (큰 결과물은 경로만) |
| `kms.status` | `"completed"` | 성공/실패 |

**LLM 호출 (`gen_ai.*`)**

| 속성 | 예시 | 용도 |
|---|---|---|
| `gen_ai.system` | `"anthropic"` | LLM 제공자 |
| `gen_ai.request.model` | `"claude-haiku"` | 모델 |
| `gen_ai.usage.input_tokens` | `1200` | 입력 토큰 |
| `gen_ai.usage.output_tokens` | `350` | 출력 토큰 |

## 5.4 Exporter 구성

하나의 span을 **여러 백엔드에 동시 송출**한다. 각 Exporter의 역할이 다르다.

```
TracerProvider
  ├── Langfuse (OTLP)   ─── LLM 호출 trace, 프롬프트 버전 관리
  ├── MLflow  (OTLP)    ─── run 단위 trace 기록, 이력 비교
  ├── File    (.jsonl)   ─── L3 Evaluator의 SpanLoader 입력
  └── Console            ─── 로컬 개발 디버깅 (운영 시 비활성)
```

**Langfuse**는 `gen_ai.*` 속성이 있는 span을 LLM 호출로 인식하여 프롬프트·응답·비용을 구조화. **MLflow**는 모든 span을 run 단위로 묶어 이력 비교를 지원. **File**은 L3 Evaluator가 로컬에서 span을 읽기 위한 경로.

## 5.5 Artifact 크기 정책

span 속성에 직접 넣는 것과 경로만 기록하는 것을 구분한다.

| 데이터 | 크기 | span에 기록하는 것 |
|---|---|---|
| parse_meta, token_count 등 | 수 바이트 | 값 자체 |
| body_md, chunks 본문 | 수 KB ~ MB | **경로만** (`kms.artifact_path`) |
| embedding 벡터 | 수 MB | **경로만** (DB 참조) |

## 5.6 분산 환경에서의 Trace 연결

Celery 워커가 다른 프로세스에서 실행되면 trace가 끊긴다. `CeleryInstrumentor`가 **task 메시지 헤더에 trace context를 자동 삽입**하여 해결.

```python
# telemetry.py 초기화 시 한 줄
CeleryInstrumentor().instrument()
```

이후 Celery task 간 span이 자동으로 부모-자식 관계로 연결된다.

---

# 6. L3: 평가 체계 (Evaluation Harness)

## 6.1 구조

평가 정의를 **데이터(YAML) + 메트릭(plugin) + 집계(group)**로 3분리한다. EleutherAI lm-evaluation-harness의 패턴을 차용.

```
kms_eval/
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

  preprocess/
    normalize_whitespace.py
    strip_frontmatter.py

  configs/
    evaluators/*.yaml       평가 정의
    groups/*.yaml            집계 정의

  corpus/                   Seed Corpus (테스트 입력 파일)
  golden/                   Golden Reference (기대 출력물)
```

## 6.2 5계층 책임 분리

하나의 평가가 실행될 때 거치는 5개 계층. 각 계층은 독립적으로 교체 가능.

```
① DATA          corpus/, golden/, artifact(S3/DB), span(File/MLflow)
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
```

## 6.3 YAML 평가 정의

코드 수정 없이 YAML 한 장으로 새 평가를 추가한다.

```yaml
# configs/evaluators/parser_pdf.yaml
evaluator: parser
target_component: Parser
corpus_glob: corpus/pdf_*.pdf
golden_dir: golden/
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

## 6.4 Evaluator의 역할

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

## 6.5 Loader 추상화

Evaluator는 저장소 위치를 모른다. 추상화된 Loader를 통해 읽는다.

| Loader | 읽는 곳 | 용도 |
|---|---|---|
| **ArtifactLoader** | S3, PostgreSQL, Graph DB | 파이프라인 산출물 |
| **SpanLoader** | File (.jsonl), MLflow API | OTel 실행 흔적 |
| **GoldenLoader** | golden/ 디렉토리 | 기대 출력물 (사람이 작성) |

로컬 개발에서는 `LocalArtifactLoader`, 운영에서는 `S3ArtifactLoader`로 교체. Evaluator 코드는 무수정.

## 6.6 Golden Reference

Golden은 "예상 지표값"이 아니라 **기대하는 출력물 자체**다. 지표는 비교 결과로 계산된다.

```
corpus/pdf_report.pdf          ← 입력 파일
     │
     ▼ Pipeline 실행
     │
artifact (body_md)             ← 실제 출력

golden/pdf_report/expected.md  ← 기대 출력 (사람이 작성)

     ↓ artifact vs golden 비교

chrf_plus = 0.92               ← 계산된 지표값
```

## 6.7 확장 시 코드 변경 범위

| 변화 | 수정 위치 | 코어 수정 |
|---|---|---|
| 새 메트릭 추가 | `metrics/` 파일 + YAML 한 줄 | 없음 |
| 새 컴포넌트 평가 | `evaluators/` 파일 + YAML | 없음 |
| 메트릭 교체 | YAML metric_list 변경 | 없음 |
| 평가 조합 변경 | `groups/` YAML | 없음 |
| 골든 세트 추가 | `golden/` 디렉토리에 파일 추가 | 없음 |

---

# 7. L4 + 수평축: 운영과 분산

## 7.1 Continuous Evaluation: 4개 모드

동일한 L3 Harness를 **다른 트리거와 범위**로 실행한다.

| 모드 | 트리거 | 범위 | 목적 | 소요 시간 |
|---|---|---|---|---|
| **Local** | 개발자 수동 | 소규모 서브셋 | 개발 중 빠른 확인 | 수 초 ~ 수 분 |
| **CI** | PR/Push | 중간 (10~20 파일) | 회귀 차단 (배포 게이트) | 5~20분 |
| **Nightly** | cron 스케줄 | 전체 corpus + 비싼 메트릭 | drift 감지 | 30분 ~ 수 시간 |
| **Shadow** | 프로덕션 트래픽 | 실 데이터 샘플링 | 실환경 검증 + 골든셋 성장 | 상시 |

```bash
# Local
python -m kms_eval.run --evaluator parser_pdf

# CI (GitHub Actions)
python -m kms_eval.run --group ci_smoke
python -m kms_eval.check_regression --baseline main

# Nightly (cron)
python -m kms_eval.run --group full_pipeline

# Shadow
# 프로덕션 파이프라인에서 저품질 span 자동 수집 → 골든셋 후보 큐
```

**MVP에서의 우선순위**: Local만으로 시작 → CI 추가 → Nightly → Shadow 순으로 점진 확장.

## 7.2 Data Flywheel (Shadow 모드의 핵심)

Shadow가 가동되면 골든셋이 **자동으로 성장**하는 루프가 형성된다.

```
프로덕션 데이터 유입
     │
     ▼
파이프라인 실행 + L2 span 방출
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

## 7.3 분산 인프라 (수평축)

L4의 4개 모드가 분산 환경에서 동작하기 위해 필요한 횡단 인프라.

### Task Queue

```
Component (로직)  ←→  Celery Task (오케스트레이션)
     분리                  얇은 래퍼

파이프라인 실행은 Celery 워커에서 분산.
평가 실행(L3)은 별도 프로세스에서 일괄 수행.
```

### Shared Storage

분산 워커 간 artifact 공유. 별도 평가 저장소를 두지 않고 파이프라인 기존 저장소를 읽는다.

| 단계 | 저장소 |
|---|---|
| MVP | NFS (공유 파일시스템) |
| 운영 | S3/MinIO |

평가 실행 간 격리는 **run_id 접두사**로 해결. 덮어쓰기 방지.

### Trace Propagation

Celery 워커 간 trace 연결이 끊기는 문제를 `CeleryInstrumentor`가 해결. task 메시지 헤더에 trace context를 자동 삽입하여 **분산 환경에서도 하나의 trace로 연결**.

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
| **Loader 추상화** (6장) | Evaluator는 "source_id로 artifact를 달라"고 요청할 뿐, 그것이 S3에 있는지 로컬에 있는지 모른다. 컴포넌트가 저장 방식을 바꿔도 Loader 구현만 교체하면 Evaluator는 무수정. |
| **YAML 바인딩** (6장) | Evaluator와 컴포넌트의 연결이 코드가 아닌 설정 파일이다. 새 컴포넌트를 추가하면 YAML 한 장을 쓰면 된다. 기존 Evaluator 코드를 열 필요가 없다. |
| **Metric Registry** (6장) | `@register_metric`으로 등록된 Metric은 어떤 Evaluator에서든 이름으로 호출 가능. 새 메트릭을 추가해도 기존 메트릭이나 Evaluator를 건드리지 않는다. |

---

### R2. 평가 범위: 파이프라인 + 그래프

> 파이프라인 각 단계와 Knowledge Graph 품질을 모두 다룬다.

**가능한 이유**: Evaluator가 **두 종류의 입력**(artifact + span)을 받을 수 있고, 컴포넌트 단위로 분리되어 있어 파이프라인 중간 단계와 최종 그래프를 **같은 구조로** 평가할 수 있기 때문이다.

| 설계 결정 | 왜 이것이 범위 확장을 가능하게 하는가 |
|---|---|
| **컴포넌트별 Evaluator** (6장) | 파이프라인의 각 단계가 독립 컴포넌트이므로, 각각에 대응하는 Evaluator를 붙일 수 있다. Parser만 평가하거나 전체를 평가하거나, 조합이 자유롭다. |
| **두 차원 평가** (3장) | OTel이 실행 흔적(span)을 자동 방출하므로, Evaluator는 "결과물이 정확한가"(artifact)와 "과정이 건강한가"(span)를 하나의 run에서 동시에 측정할 수 있다. OTel이 없었다면 실행 품질은 별도 시스템을 구축해야 했다. |
| **Group 집계** (6장) | YAML로 Evaluator를 Stage 단위 / 전체 단위로 묶을 수 있다. 그래프 평가(GraphStructureEval)도 같은 Group 구조에 편입되므로 파이프라인 평가와 그래프 평가가 하나의 리포트에 합쳐진다. |

---

### R3. 인터페이스 변화에 대한 내성

> 필드 변화에는 유연하되, 흐름 구조에 결합한다.

**가능한 이유**: 평가 체계가 결합하는 지점이 **개별 필드가 아니라 컴포넌트 분해 구조**이기 때문이다. 필드가 바뀌면 영향을 받는 곳이 한 곳(Evaluator의 가공 로직)으로 국한된다.

| 설계 결정 | 왜 이것이 변화 내성을 주는가 |
|---|---|
| **Evaluator = 지휘자** (6장) | Evaluator는 "artifact에서 표만 뽑아서 TEDS에 넘긴다" 같은 가공을 담당한다. 필드가 바뀌면 이 가공 로직만 수정하면 된다. Metric(TEDS 계산) 자체는 "두 표를 받아서 점수를 낸다"이므로 필드 변화와 무관하다. |
| **Preprocess 분리** (6장) | 데이터 정규화(BOM 제거, frontmatter 제거)가 Evaluator/Metric 밖에 있다. 포맷이 바뀌면 preprocess 함수만 수정. Metric은 정규화된 데이터만 받으므로 영향 없다. |
| **YAML version 필드** (6장) | 평가 정의 자체가 바뀌면(메트릭 추가, 기준 변경) 버전을 올린다. 다른 버전의 결과끼리 비교하면 경고가 뜬다. 인터페이스 변경으로 인한 오탐 regression을 방지한다. |

---

### R4. 분산 실행 대응

> 단일 프로세스와 분산 환경 모두에서 일관 동작한다.

**가능한 이유**: 파이프라인 로직(Component)이 **실행 환경을 모르기** 때문이다. Component는 로컬에서 직접 호출해도, Celery 워커에서 호출해도 동일하게 동작한다. 환경 차이는 Component 바깥의 인프라 계층이 흡수한다.

| 설계 결정 | 왜 이것이 분산 대응을 가능하게 하는가 |
|---|---|
| **Component / Task 분리** (4장) | Component는 입력을 받아 출력을 반환하는 순수 로직이다. Celery task는 "어떤 워커에서 이 Component를 호출할지"만 결정하는 얇은 래퍼. 로컬 테스트에서는 task 없이 Component를 직접 호출하면 된다. |
| **CeleryInstrumentor** (5장) | Celery 메시지 헤더에 trace context를 자동 삽입한다. 워커가 다른 프로세스여도 span이 부모-자식으로 연결되므로, 분산 환경에서도 하나의 trace로 전체 파이프라인을 추적할 수 있다. |
| **Shared Storage** (7장) | 워커 A가 저장한 artifact를 워커 B(또는 Evaluator)가 읽을 수 있다. NFS나 S3를 쓰면 "어느 워커가 저장했는지"는 투명해진다. |
| **Loader 추상화** (6장) | 로컬에서는 `LocalArtifactLoader`, 분산에서는 `S3ArtifactLoader`. Evaluator 코드는 동일. 환경 차이가 Loader 한 곳에 캡슐화된다. |

---

### R5. 장기 확장: 관측·감사 기반

> 단기 개발 도구에서 장기 운영 관측성의 토대로 확장한다.

**가능한 이유**: OTel이 **인프라 계층**이므로, 인프라를 재구축하지 않고 그 위에 **응용만 추가**하면 되기 때문이다.

| 설계 결정 | 왜 이것이 장기 확장을 가능하게 하는가 |
|---|---|
| **OTel = 인프라 계층** (5장) | 파이프라인에 한 번 심은 계측은 평가·모니터링·비용추적·감사에 모두 재사용된다. 운영기에 모니터링 대시보드를 추가하려면 Exporter를 하나 더 연결하면 된다. 파이프라인 코드 재계측 불필요. |
| **4개 운영 모드** (7장) | Local → CI → Nightly → Shadow는 같은 Harness를 다른 트리거로 실행할 뿐이다. 개발기에 Local만 쓰다가 운영기에 Shadow를 켜도 평가 로직을 다시 작성하지 않는다. |
| **MLflow 중앙 저장** (3장) | 개발기의 실험 추적과 운영기의 이력 관리가 같은 MLflow에 누적된다. "6개월 전 파서 v1.0과 현재 v2.3의 메트릭 비교"가 자연스럽게 가능하다. |
| **Data Flywheel** (7장) | Shadow 모드에서 프로덕션 실패 사례가 골든셋으로 자동 환류한다. 평가 체계가 운영 데이터와 함께 성숙하므로, 시간이 갈수록 평가의 신뢰도가 높아진다. |
