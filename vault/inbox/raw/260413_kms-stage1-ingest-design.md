# Stage 1: 보관 (Ingest) — 최종 설계

> 작성일: 2026-04-13
> 성격: KMS 파이프라인 Stage 1 설계 확정본. 원본 설계 문서(02_MASTER-PLAN.md)의 비판적 회고를 거쳐 재구성.

## 책임

원본을 있는 그대로 받아서 보관한다. 파일 내용을 읽지 않는다.

## 안 하는 것

- 파일 내용을 읽지 않음
- 형식을 판단하지 않음 (Stage 2 파서의 책임)
- 프로파일을 추론하지 않음
- 경로를 분기하지 않음
- 복합 입력을 분해하지 않음 (커넥터의 책임)

## 저장소

- Object Storage: `/ingests/{ingest_id}/{파일명}`
- RDB: source_record

## source_record 필드 (18개)

### 파일 식별 (5개)

| 필드 | 설명 |
|------|------|
| source_id | PK. 시스템 생성. |
| parent_id | 같이 들어온 문서의 부모. null이면 최상위. |
| raw_path | /ingests/{ingest_id}/{파일명} |
| file_name | 원본 파일명 |
| file_ext | 확장자 (.pdf, .md, .xlsx ...) |

### 무결성 (2개)

| 필드 | 설명 |
|------|------|
| sha256 | 중복 감지 + 무결성 검증 |
| file_size | 바이트 |

### 감사 (4개)

| 필드 | 설명 |
|------|------|
| actor_id | 행위 주체 식별자 ("jiseon", "slack-connector") |
| ingested_at | KMS에 들어온 시각 |
| source_type | "confluence", "slack", "filesystem" 등 |
| source_location | "#dev-ops", "Engineering/Architecture" 등 |

### 권한 (3개)

| 필드 | 설명 |
|------|------|
| owner | 수정/삭제 권한 책임자 |
| domain | 읽기 범위 (수평: 어느 팀) |
| sensitivity | 읽기 범위 (수직: public, internal, confidential, restricted) |

### 처리 제어 (1개)

| 필드 | 설명 |
|------|------|
| analytical_depth | reference, standard, deep. 호출자 지정 > 소스 매핑 > 기본값(reference) |

### 상태 (2개)

| 필드 | 설명 |
|------|------|
| status | active, archived, deleted |
| progress | pending, processing, completed, failed |

## analytical_depth 결정 규칙

우선순위 체인 (위에서 아래로, 먼저 걸리면 끝):

1. 호출자 명시 지정
2. source_location 매핑 규칙
3. sensitivity 매핑 규칙
4. domain 매핑 규칙
5. file_ext 매핑 규칙
6. 시스템 기본값 (reference)

## 권한 자동 유도 규칙

- owner = 호출자 지정 || 커넥터가 넘긴 author || actor_id
- domain = 호출자 지정 || source_location 매핑 || "default"
- sensitivity = 호출자 지정 || domain 정책 || "internal"

## Object Storage 구조

```
/ingests/{ingest_id}/
  {원본 파일명}
  {원본 파일명}
  ...
```

같이 들어온 파일을 같은 폴더에 평면으로 저장. 본체/첨부 관계는 source_record.parent_id가 소유. 폴더 구조에 계층을 두지 않음.

## doc_node 시간/상태 메타데이터

Stage 2 이후 doc_node에 부여되는 시간/상태 필드:

| 필드 | 타입 | 의미 |
|------|------|------|
| created_at | timestamp | 생성 시각 (불변) |
| updated_at | timestamp | 마지막 수정 시각 |
| status | enum | active, archived, deleted |
| progress | enum | pending, processing, completed, failed |

freshness, is_stale 등은 저장하지 않고 쿼리 시 계산.

## 원본 설계와의 차이

### 제거된 것
- 6축 프로파일 Phase A/B 추론
- 4경로 분기 (full/lightweight/fast/signal)
- 4 Tier (파서 내부로 캡슐화)
- 복합 입력 분해 (커넥터의 책임)
- source_profile (volatility, density, scale, lifespan, provenance)
- created_at (ingested_at과 중복)
- ingestion_method (source_type과 중복 가능성)
- error_msg

### 추가된 것
- parent_id (묶음 관계)
- source_type, source_location (출처 추적)
- status/progress 분리 (lifecycle과 pipeline 분리)
- 권한/감사 필드 체계화 (8차원 감사 프레임워크 기반)

### 변경된 것
- ingested_by + connector_id → actor_id 하나로 통합
- 6축 → analytical_depth 1개로 축소 (나머지는 메타데이터)
- status (stage1_complete/...) → status/progress 2필드로 단순화
- Stage 1 책임 축소: 접수+분류+라우팅+분해 → 접수만

## 설계 근거

### 업계 원칙
- "Classify by behavior, not by attribute" (Beauchemin)
- "One axis, one knob" (Tunkelang)
- 라우팅 축 3개 이하 (Dagster/Schrock)
- 접수는 하류를 모른다 (Kleppmann)
- 메타데이터 분류는 변환 단계에서 (프레임워크 공통)

### 감사 메타데이터 MECE 검증 (5W1H + Integrity + Outcome)
- Identity: actor_id
- Action: INGEST (고정)
- Temporality: ingested_at
- Origin: source_type, source_location
- Purpose: (source_type으로 유추)
- Mechanism: actor_id (겸용)
- Integrity: sha256
- Outcome: progress

### 권한 메타데이터 MECE 검증
- 누가 책임지나: owner
- 어느 조직 범위인가: domain
- 얼마나 민감한가: sensitivity
- 세 질문이 겹치지 않고, 합치면 접근 가능 여부를 완전히 결정
