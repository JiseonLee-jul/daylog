# KMS 파이프라인 전체 흐름 — 커넥터부터 검색까지

> 작성일: 2026-04-14
> 성격: 입력 분해 → 파이프라인 → 그래프 등록 전체 흐름 시각화

```
KMS 파이프라인 전체 흐름
═══════════════════════════════════════════════════════════════════

[외부 소스 이벤트]
  · Slack 메시지 + URL
  · 이메일 + 첨부파일들
  · Confluence 페이지 수정
  · 파일 업로드
  · ...

       │
       ▼
┌────────────────────────────────────────────────────────────────┐
│ 커넥터 / 어댑터 (KMS 밖)                                         │
│                                                                │
│ 책임: 외부 이벤트를 KMS가 이해하는 단위로 분해                      │
│                                                                │
│ · 소스 시스템의 API/형식을 이해                                    │
│ · 컨테이너 포맷 분해:                                             │
│   - 이메일 → 본문 + 첨부N개                                       │
│   - Slack 메시지 → 텍스트 + 첨부/공유 링크                          │
│   - ZIP/TAR → 내부 파일들                                         │
│   - 이메일 스레드 → 각 이메일                                      │
│ · 관계 정보 수집 (parent_id 힌트)                                  │
│ · 메타데이터 준비:                                                 │
│   - actor_id (누가), source_type, source_location                │
│   - owner, domain, sensitivity 제안                               │
│                                                                │
│ 각 분해된 파일마다 kms_ingest() 호출                              │
│                                                                │
│ 예시 (이메일 1통 + 첨부 2개):                                     │
│   kms_ingest(file=message.eml, parent_id=null)                  │
│   kms_ingest(file=report.pdf, parent_id={이메일 source_id})       │
│   kms_ingest(file=data.xlsx, parent_id={이메일 source_id})        │
└──────────────────────────┬─────────────────────────────────────┘
                           │
                           │ kms_ingest(file, parent_id, metadata)
                           │ (분해된 개별 파일 N회 호출)
                           │
       ┌───────────────────┼───────────────────┐
       ▼                   ▼                   ▼
   [파일 1]            [파일 2]            [파일 3]
   (본체)              (첨부)              (첨부)
       │                   │                   │
       │  (각 파일이 독립 파이프라인)              │
       ▼                   ▼                   ▼

═══════════════ KMS 파이프라인 (파일 1건 기준) ═══════════════

┌────────────────────────────────────────────────────────────────┐
│ Stage 1: Ingest (수집)                                          │
│                                                                │
│ · 원본 파일 저장                                                 │
│   /ingests/{ingest_id}/{파일명}                                  │
│   (같이 들어온 파일은 동일 ingest_id 폴더)                         │
│                                                                │
│ · SHA256 계산 (중복 감지)                                         │
│                                                                │
│ · source_record INSERT                                          │
│   ┌──────────────────────────────────────────┐                 │
│   │ source_id (새로 생성)                      │                 │
│   │ ingest_id (커넥터가 넘겨줌)                 │                 │
│   │ parent_id (커넥터가 넘겨줌, null 가능)      │                 │
│   │ raw_path, file_name, file_ext             │                 │
│   │ sha256, file_size                          │                 │
│   │ actor_id, ingested_at                      │                 │
│   │ source_type, source_location               │                 │
│   │ owner, domain, sensitivity                 │                 │
│   │ processing_depth, requires_review          │                 │
│   │ status=active, progress=pending            │                 │
│   └──────────────────────────────────────────┘                 │
└─────────────────────────┬──────────────────────────────────────┘
                          │
                          │ source_record 생성 완료
                          │
                          ▼
┌────────────────────────────────────────────────────────────────┐
│ Stage 2: Parse (변환)                                           │
│                                                                │
│ · 파서 선택 (file_ext 기반)                                      │
│   .pdf → Docling / .docx → Docling / .mp3 → Whisper             │
│   .csv/.json → 구조화 파서 / .md → 통과                           │
│                                                                │
│ · 마크다운 변환 → body.md (임시)                                   │
│   /transformed/{ingest_id}/{source_id}/body.md                   │
│                                                                │
│ · 청킹 (문서 타입/구조/크기 기반)                                   │
│   - 코드 → AST / 표 → 행 / 채팅 → 스레드 / 산문 → 문장             │
│   - heading 충분 → 구조적 / 부족 → 시맨틱 (경량 LLM)                 │
│   - < 2K 토큰 → flat / ≥ 2K + 다단 → parent-child                 │
│                                                                │
│ · chunks 테이블 INSERT                                          │
│   ┌──────────────────────────────────────────┐                 │
│   │ chunk_id, source_id, order_index          │                 │
│   │ text, token_count, char_offset_*          │                 │
│   │ parent_chunk_id, chunk_type               │                 │
│   │ embedding=NULL (Index에서 채움)            │                 │
│   └──────────────────────────────────────────┘                 │
│                                                                │
│ · progress = processing                                         │
└─────────────────────────┬──────────────────────────────────────┘
                          │
                          │ body.md + chunks 준비 완료
                          │
                          ▼
┌────────────────────────────────────────────────────────────────┐
│ Stage 3: Index (인덱싱)                                         │
│                                                                │
│ ┌─ 1. 검증 (동기, per-doc)                                       │
│ │    · frontmatter schema, FK, chunk 트리                       │
│ │    · ACL 매칭, 링크 유효성                                       │
│ │    실패 → 그래프 미등록, progress=failed                          │
│ │                                                              │
│ ▼                                                              │
│ ┌─ 2. 깊이별 처리                                                 │
│ │    모든 depth: L0 생성, chunk 임베딩, L0 임베딩                    │
│ │    standard+:   L1 생성, L1 임베딩                              │
│ │                 entity 추출, 관계 엣지 (LightRAG)                 │
│ │    chunks 테이블 UPDATE embedding=?                              │
│ │                                                              │
│ ▼                                                              │
│ ┌─ 3. 그래프 등록 (트랜잭션)                                        │
│ │    ┌──────────────────────────────────────┐                   │
│ │    │ doc_nodes INSERT                       │                   │
│ │    │   doc_id, source_id, title             │                   │
│ │    │   l0_abstract, l0_embedding            │                   │
│ │    │   l1_overview, l1_embedding            │                   │
│ │    │   tags, language, content_path         │                   │
│ │    │   review_status (pending if required)  │                   │
│ │    └──────────────────────────────────────┘                   │
│ │    ┌──────────────────────────────────────┐                   │
│ │    │ entity_nodes INSERT (standard+)        │                   │
│ │    │   canonical_name, entity_type, desc    │                   │
│ │    │   (어댑터: LightRAG 결과 변환)           │                   │
│ │    └──────────────────────────────────────┘                   │
│ │    ┌──────────────────────────────────────┐                   │
│ │    │ edges INSERT                            │                   │
│ │    │   part_of (doc ↔ chunk)                 │                   │
│ │    │   mentions (chunk ↔ entity)             │                   │
│ │    │   related_to (entity ↔ entity)          │                   │
│ │    │   has_attachment (parent_id 있을 때)     │                   │
│ │    └──────────────────────────────────────┘                   │
│ │                                                              │
│ ▼                                                              │
│ ┌─ 4. content.md 최종 저장                                        │
│ │    /transformed/{ingest_id}/{source_id}/content.md             │
│ │    frontmatter (Ingest+Parse+Index 메타 전부) + body            │
│ │    body.md 삭제 (옵션)                                           │
│ │                                                              │
│ ▼                                                              │
│ ┌─ 5. 후속 처리 큐 등록                                            │
│      deep → synthesis 배치 큐 (Phase 2)                           │
│      requires_review=true → 사람 검증 큐                           │
│                                                                │
│ · progress = completed                                          │
└─────────────────────────┬──────────────────────────────────────┘
                          │
                          ▼
                 ★ 검색 가능 상태


─────────────────────────────────────────────────────────────────
                      파이프라인 밖 시스템
─────────────────────────────────────────────────────────────────

  배치 스케줄러 (Celery beat)
    일일: stale 감지 (freshness 계산)
    주간: consolidation (entity 병합)
    [Phase 2]: synthesis, connection inference, cross-doc 검증

  이벤트 핸들러 (Celery 이벤트 워커)
    review_completed    → review_status 갱신
    promotion_requested → Stage 3 재실행
    document_archived   → status=archived
    document_deleted    → cascade 삭제

  승인 큐 API
    · 사람 검증 큐 (requires_review=true)
    · 승격 승인 큐

  검색 서비스 (FastAPI + MCP)
    · kms_search, kms_get, kms_graph
    · DB 조회만 (파이프라인과 독립)


─────────────────────────────────────────────────────────────────
                      커넥터 예시 — 이메일 + 첨부
─────────────────────────────────────────────────────────────────

입력 이벤트:
  From: jiseon@company.com
  Subject: 간식 카탈로그 공유
  Body: "새 업체 찾았어요. 참고해주세요."
  Attachments: [catalog.pdf, pricing.xlsx]


이메일 커넥터 동작:
  ingest_id = uuid()  (이 이벤트 전체의 ID)
  
  분해:
    1) 이메일 본문 텍스트
       kms_ingest(
         content="새 업체 찾았어요...",
         file_ext="eml",
         ingest_id=ingest_id,
         parent_id=null,
         metadata={...}
       ) → source_id=src_001
    
    2) catalog.pdf 첨부
       kms_ingest(
         file=catalog.pdf,
         ingest_id=ingest_id,
         parent_id=src_001,
         metadata={...}
       ) → source_id=src_002
    
    3) pricing.xlsx 첨부
       kms_ingest(
         file=pricing.xlsx,
         ingest_id=ingest_id,
         parent_id=src_001,
         metadata={...}
       ) → source_id=src_003


저장 결과:

  /ingests/{ingest_id}/
    message.eml
    catalog.pdf
    pricing.xlsx

  /transformed/{ingest_id}/
    {src_001}/content.md    (이메일 본문)
    {src_002}/content.md    (PDF 카탈로그)
    {src_003}/content.md    (Excel 가격표)

  source_records:
    src_001 (parent_id=null)
    src_002 (parent_id=src_001)
    src_003 (parent_id=src_001)

  Graph:
    doc_node(src_001) ──has_attachment──> doc_node(src_002)
                      ──has_attachment──> doc_node(src_003)
    
    각 doc_node 하위:
      chunk_nodes
      entity_node mentions


─────────────────────────────────────────────────────────────────
                      커넥터 예시 — Slack 메시지 + URL
─────────────────────────────────────────────────────────────────

입력 이벤트:
  Channel: #dev-ops
  User: admin
  Message: "PostgreSQL vacuum 관련 좋은 글. https://blog.example.com/vacuum"


Slack 커넥터 동작:
  ingest_id = uuid()
  
  분해:
    1) 메시지 텍스트 (URL 포함)
       kms_ingest(
         content="PostgreSQL vacuum 관련 좋은 글...",
         ingest_id=ingest_id,
         parent_id=null,
         source_type="slack",
         source_location="#dev-ops"
       ) → src_010
    
    2) URL 콘텐츠 (별도 fetch)
       kms_ingest(
         url="https://blog.example.com/vacuum",
         ingest_id=ingest_id,
         parent_id=src_010,       ← 메시지가 parent
         source_type="web"
       ) → src_011
    
  커넥터가 URL을 자동 fetch하고 별도 source_record로 등록.
  "메시지가 URL을 공유했다"는 관계는 has_attachment (또는 references) 엣지로.


─────────────────────────────────────────────────────────────────
                      핵심 원칙 정리
─────────────────────────────────────────────────────────────────

1. 입력 분해는 커넥터의 책임 (KMS 밖)
   · 커넥터가 외부 이벤트를 KMS가 이해하는 단위로 분해
   · KMS는 분해된 개별 파일/데이터만 받음

2. 처리 원자 단위 = 파일/데이터 (source_record 1개)
   · 각 파일이 독립적으로 Ingest → Parse → Index 거침
   · 실패 격리, 병렬 처리, 재시도가 자연스러움

3. 쓰레드/그룹 개념 = 메타데이터
   · ingest_id로 같이 들어온 것 식별
   · parent_id로 본체-첨부 관계 표현
   · has_attachment 엣지로 그래프에 표현

4. KMS는 "왜 같이 왔는지"를 모름
   · 커넥터만 소스 시스템의 의미를 알고 있음
   · KMS는 "이 파일들은 parent_id로 연결된다"만 알면 됨
```
