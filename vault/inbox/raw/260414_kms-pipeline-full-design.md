# KMS 파이프라인 전체 설계 — Stage 1/2/3 최종

> 작성일: 2026-04-14
> 성격: KMS 데이터 처리 파이프라인 3단계 설계 확정본
> 배경: 02_MASTER-PLAN.md 원본 설계의 비판적 회고를 거쳐 재구성

## 설계 원칙

- 합리성 기반: 업계 표준/원칙을 근거로 판단
- 관심사 분리: 접수/변환/제공의 명확한 책임 분리 (Beauchemin, Kleppmann 원칙)
- 최소 개념: 개념 수를 늘리기보다 각 개념이 명확한 역할을 갖도록
- SSOT는 외부 원본: KMS는 파생물 캐시 (Federated Origin + Cached Derivatives)

## 파이프라인 개요

```
Stage 1 (보관) → Stage 2 (읽기) → Stage 3 (이해) → Graph 등록

파이프라인 밖:
  · 배치 스케줄러: synthesis, consolidation, stale 감지
  · 이벤트 기반: 사람 검증, depth 승격
```

---

## Stage 1: 보관 (Ingest)

### 책임
원본을 있는 그대로 받아서 보관한다. 파일 내용을 읽지 않는다.

### 저장소
- Object Storage: `/ingests/{ingest_id}/{파일명}`
- RDB: source_record

### source_record 필드 (19개)

**파일 식별 (5개)**
- source_id: PK
- parent_id: 같이 들어온 문서의 부모. null이면 최상위.
- raw_path
- file_name
- file_ext

**무결성 (2개)**
- sha256
- file_size

**감사 (4개)**
- actor_id: "jiseon" | "slack-connector" | ...
- ingested_at
- source_type: "confluence" | "slack" | ...
- source_location: "#dev-ops" | "Engineering/Database" | ...

**권한 (3개)**
- owner
- domain
- sensitivity: public | internal | confidential | restricted

**처리 제어 (2개)**
- processing_depth: reference | standard | deep
- requires_review: true | false

**상태 (2개)**
- status: active | archived | deleted
- progress: pending | processing | completed | failed

### processing_depth 결정 규칙 (우선순위 체인)
1. 호출자 명시 지정
2. source_location 매핑
3. sensitivity 매핑
4. domain 매핑
5. file_ext 매핑
6. 기본값: reference

### requires_review 결정 규칙
1. 호출자 명시 지정
2. source_location 매핑 (/policies/** → true)
3. sensitivity 매핑 (confidential/restricted → true)
4. domain 매핑 (legal/governance → true)
5. 기본값: false

### 원본 설계와의 차이
- 6축 프로파일 Phase A/B 추론 → 제거 (파일 안 읽음)
- 4경로 분기 (full/lightweight/fast/signal) → 제거 (파서가 판단)
- 복합 입력 분해 → 커넥터 책임
- critical depth → requires_review 별도 필드로 분리
- Stage 1 책임: 접수 + 분류 + 라우팅 + 분해 → 접수만

---

## Stage 2: 읽기 (Read/Parse)

### 책임
원본 파일을 마크다운으로 변환한다. depth를 모른다.

### 저장소
- Object Storage: `/transformed/{source_id}/content.md`
- RDB: chunks 테이블

### 파서 책임
파일 형식을 파서 내부에서 감지하고 적절한 도구로 변환:
- .pdf → Docling (+ OCR 자동)
- .docx → Docling
- .mp3/.wav → Whisper STT
- .csv/.json → 구조화 파서
- .md → 그대로 통과
- 이미 마크다운 → 파싱 스킵

### 출력: content.md frontmatter (14개 필드)

**식별 (2개)**: source_id, title
**출처 (3개)**: source_type, source_location, ingested_at
**요약 (2개)**: abstract, overview (Stage 3에서 채워짐)
**지식 신호 (2개)**: tags, entities (Stage 3에서 채워짐)
**콘텐츠 메타 (1개)**: language
**상태 (3개)**: status, updated_at, review_status

### 청킹 전략

**결정 축: 문서 타입 + 구조 + 크기** (depth와 독립)

1. 콘텐츠 타입 기반
   - 코드 → AST 기반 청킹
   - 표 → 행 기반 (헤더 유지)
   - 채팅 → 스레드 단위
   - 산문 → 문장/의미 단위

2. 구조 유무 기반
   - heading 충분 → 구조적 청킹
   - heading 부족 → 시맨틱 청킹 (경량 LLM)

3. 크기 기반
   - < 2,000 토큰 → flat
   - ≥ 2,000 토큰 + 다단 구조 → parent-child

### 파싱 품질 (3차원 통일 + 형식별 세부 — 참고)
- Completeness (완전성)
- Fidelity (충실성)
- Structural Integrity (구조 보존)

### 원본 설계와의 차이
- depth별 청킹 차등 → 제거 (문서 특성으로 결정)
- 4 Tier 개념 → 파서 내부로 캡슐화
- Phase B 프로파일 재조정 → 제거

---

## Stage 3: 이해 (Understand)

### 책임
마크다운과 청크를 지식 객체로 변환하고 그래프에 등록한다.

### 입력
- content.md (frontmatter + body)
- chunks 테이블
- processing_depth
- requires_review

### 순서

**1. 검증 (동기, per-doc)**
- frontmatter schema 확인
- source_id ↔ chunks 참조 무결성
- chunk 트리 일관성
- ACL 스코프 매칭
- 상대 경로 링크 유효성
- Stage 2 검증과 겹치되, 그래프 등록 직전 최종 관문 (defense in depth)

**2. 깊이별 처리**

모든 depth 공통:
- L0 요약 생성 (token_count 기반 적응형)
- chunk 임베딩 생성
- L0 임베딩

standard 이상:
- L1 요약 생성
- L1 임베딩
- entity 추출 (LightRAG 기반)
- 관계 엣지 추출

**3. 그래프 등록**
- doc_node, chunk_node, entity_node, edges
- part_of 엣지 (doc ↔ chunk)
- mentions 엣지 (chunk ↔ entity)
- 트랜잭션 단위 커밋

**4. frontmatter 갱신**
- abstract, overview, tags, entities, updated_at

**5. 후속 처리 큐 등록**
- deep → synthesis 배치 큐
- requires_review=true → 사람 검증 큐

### L0/L1 크기 결정 (token_count 기반)

L0:
- < 200 토큰 → 생성 안 함 (본문=L0)
- 200~2,000 → 200
- 2,000~20,000 → 350 (기본)
- > 20,000 → 500

L1 (standard 이상만):
- < 2,000 → 생략
- 2,000~20,000 → 원본의 ~10%
- > 20,000 → 원본의 ~5% + 섹션별 L1

### 임베딩 모델 (MVP)
- 온프레미스: EmbeddingGemma 300M
- 클라우드: Gemini Embedding 2
- 단일 모델. 목적별 확장은 Phase 2.

### 원본 설계와의 차이
- Stage 3 내부 4층위 (3-a/b/c/d) → 평면화 (depth 분기로 대체)
- synthesis → 배치 스케줄러로 이동 (파이프라인 밖)
- critical depth → requires_review 별도 필드
- 검증 시점 → 그래프 등록 후 → 그래프 등록 전

---

## 파이프라인 밖

### 배치 스케줄러 (주기적)
- synthesis 생성 (deep 문서 대상, 일일/주간)
- consolidation (중복 entity 병합)
- connection inference (새 관계 추론)
- stale 감지 (lifespan 기반)
- cross-doc 검증 (고아 entity, ACL drift)

### 이벤트 기반
- 사람 검증 완료 → review_status=verified, trust 갱신
- depth 승격 요청 → Stage 3 재실행
- 원본 변경 감지 → Stage 2부터 재실행

### 승격 정책
- "심화 제안 먼저 → 확인 후 실행"
- 자동 승격 아님
- 트리거: 에이전트 원본 fetch, 사용자 요청, 검색 N회 이상, 관리자 지정

---

## 그래프 구조

```
[doc_node] ← 문서 단위 (L0, L1, 메타)
   │
   ├── [chunk_node] ← 본문 조각 (Graph DB에 chunks 테이블 PK 참조)
   │        │
   │        └── mentions → [entity_node]
   │
   ├── [parent_chunk_node] ← 섹션 단위 (parent-child 구조일 때)
   │        └── [chunk_node]
   │
   └── has_attachment → [doc_node] (parent_id 구조)

[entity_node] ←→ [entity_node] (relationship edge)
```

**문서 노드 패턴 (Neo4j 스타일)**:
- 메타데이터 단일 관리점 (domain, sensitivity, owner)
- 출처 추적
- 접근 제어 앵커
- 컨텍스트 확장 (chunk → 같은 doc의 sibling 탐색)

---

## 설계 근거

### 업계 원칙 (리서치 결과)
- "Classify by behavior, not by attribute" — Beauchemin
- "One axis, one knob" — Tunkelang
- 라우팅 축 3개 이하 — Dagster/Schrock
- 접수는 하류를 모른다 — Kleppmann
- Federated Origin + Cached Derivatives — 설계 문서 07 ADR

### 청킹 전략 차등의 실증 근거
리서치 결과: depth(문서 중요도)별 청킹 차등은 근거 없음. 학술 논문, 프로덕션 시스템, LightRAG 코드 모두 문서 타입/구조/크기로 결정. 원본 설계의 "reference=flat, standard=parent-child"는 검증되지 않은 설계 가설.

### 그래프 시간 모델의 최소화
리서치 결과: LightRAG은 2 timestamp + 1 status만 사용. GraphRAG은 timestamp 0개. "저장 vs 계산" 원칙에 따라 freshness, is_stale 등 파생값은 저장하지 않고 쿼리 시 계산.

### synthesis의 비용 대비 이득
리서치 결과: Microsoft GraphRAG 커뮤니티 리포트는 1,000 문서당 $50~160, 개방형 thematic 질의에서만 품질 이득. FAQ/factual 질의에서는 입증 안 됨. MVP에서 생략 권장.

### LightRAG 통합 주의점
LightRAG은 entity 중심 그래프. doc_node/chunk_node 없음. 우리 설계(Neo4j 스타일)와 다르므로 어댑터 레이어 필요. 구현 방식은 추후 결정.

---

## MVP 범위 정리

### 포함
- Stage 1/2/3 전체 파이프라인
- processing_depth (reference/standard/deep)
- requires_review
- 문서 노드 + 청크 노드 + entity 노드
- 기본 임베딩 1개
- 동기 검증 (per-doc)
- LightRAG 어댑터 (entity/관계 추출)

### 제외 (Phase 2 이후)
- synthesis 생성
- connection inference
- 목적별 임베딩 모델 확장
- 이미지/멀티모달 임베딩
- cross-doc 배치 검증
- Wiki/큐레이션 기여 (topic_article)
- depth 강등

---

## 미결정 사안 (추후 결정)

1. LightRAG 통합 방식 (A: 어댑터, B: 완전 커스텀, C: 하이브리드)
2. 파싱 품질 점수 구체 공식 (3차원 통일 + 형식별 세부는 참고)
3. entity 추출 프롬프트 구조
4. 승격 트리거 임계값 구체화
5. 배치 실행 주기 (일일 vs 주간)
