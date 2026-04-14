# KMS 파이프라인 전체 흐름 — 원본 설계 (02_MASTER-PLAN.md)

> 작성일: 2026-04-14
> 성격: 원본 설계 문서의 파이프라인 전체 흐름 시각화 (비교용)
> 출처: c:/Users/ezis-jiseon/Downloads/AI_KMS_기획_0413/02_requirements/02_MASTER-PLAN.md

```
KMS 파이프라인 전체 흐름 — 원본 설계
═══════════════════════════════════════════════════════════════════

[외부 소스 이벤트]
  · Slack 메시지 + URL
  · 이메일 + 첨부파일들
  · Confluence 페이지
  · 파일 업로드
  · 실시간 신호 (flag, suggest)
  · ...

       │
       ▼
  (커넥터의 역할 — 원본 설계에서 명시 없음)
  (Stage 1이 "파이프라인 경로를 결정하는 능동적 라우터" 역할로 흡수)

       │
       │ kms_ingest() 호출
       │
       ▼

═══════════════ KMS 파이프라인 (원본 설계) ═══════════════

┌────────────────────────────────────────────────────────────────┐
│ Stage 1: Landing (접수 및 분류 + 라우팅)                          │
│                                                                │
│ · 원본 파일 저장                                                 │
│   /raw/{source_id}/original.*                                   │
│                                                                │
│ · SHA256 계산 (중복 감지)                                         │
│                                                                │
│ · 파일 형식 감지                                                 │
│                                                                │
│ · 복합 입력 분해                                                 │
│   "메시지 + 논문 URL" → 별도 doc_node 두 개                       │
│   URL 정규화 + SHA256으로 이미 등록된 문서는 엣지만 추가              │
│                                                                │
│ · source_profile 6축 Phase A 추론 (형식 기반)                     │
│   volatility / density / scale / lifespan /                     │
│   provenance / analytical_depth                                  │
│                                                                │
│ · 4경로 분기 결정                                                │
│   ┌──────────────────────────────────────────┐                 │
│   │ source_type + doc_type 조합으로 판단        │                 │
│   │                                             │                 │
│   │ full path       PDF, URL, Slack 등          │                 │
│   │ lightweight     Agent Report (마크다운)      │                 │
│   │ fast            연구노트 (micro, 동기)        │                 │
│   │ signal          flag, suggest (Stage 1 미경유)│                 │
│   └──────────────────────────────────────────┘                 │
│                                                                │
│ · RDB source_record INSERT                                      │
│   ┌──────────────────────────────────────────┐                 │
│   │ source_id, file 메타                       │                 │
│   │ source_profile 6축                          │                 │
│   │ 거버넌스 필드 (ingested_by, owner, domain,   │                 │
│   │   sensitivity, ingestion_method,           │                 │
│   │   acl_source, acl_reference)                │                 │
│   │ status: stage1_complete                     │                 │
│   └──────────────────────────────────────────┘                 │
│                                                                │
│ 산출물: /raw/, source_record                                     │
└─────────────────────┬──────────────────────────────────────────┘
                      │
                      │ 경로별로 분기하여 Stage 2 진입
                      │
       ┌──────────────┼──────────────┬──────────────┐
       ▼              ▼              ▼              ▼
   full path    lightweight      fast path      signal path
                                                 (Stage 1 미경유,
                                                  기존 노드 수정)
       │              │              │
       ▼              ▼              ▼
┌────────────────────────────────────────────────────────────────┐
│ Stage 2: Transform (파싱 + 요약 + 청킹 + 임베딩 + Graph 등록)     │
│                                                                │
│ analytical_depth에 따라 경로 A or B:                             │
│                                                                │
│ ┌─ 경로 A: 얕은 파싱 (reference)                                 │
│ │    · Tier 2 텍스트 레이어 추출                                  │
│ │    · /transformed/{source_id}/content.md 생성                  │
│ │    · L0 요약 (적응형 100~500토큰, density+scale 기반)            │
│ │    · L0 임베딩                                                 │
│ │    · flat chunk (5~30개, parent-child 없음)                     │
│ │    · doc_node 등록 (Graph DB)                                  │
│ │    · chunk_node 등록 (flat)                                    │
│ │                                                              │
│ ┌─ 경로 B: 깊은 파싱 (standard 이상)                              │
│ │    · 파서 선택 + Tier 결정                                      │
│ │      (Tier 2 기본, 텍스트 레이어 없으면 Tier 3~4 승격)            │
│ │      (density + scale + 텍스트 레이어 유무로 판단)                │
│ │    · Docling/Whisper/구조화 파서로 마크다운 변환                  │
│ │    · /transformed/{source_id}/content.md                        │
│ │    · YAML 프런트매터 (6축 포함, 메타데이터 전부)                    │
│ │    · Phase B 프로파일 재조정 (L0 보고 내용 기반 조정)               │
│ │    · L0 요약 (적응형) + L1 요약 (원본 크기 비례)                    │
│ │    · 구조적 청킹 (heading 기반)                                  │
│ │      또는 시맨틱 청킹 (heading < 3개, 경량 LLM)                    │
│ │    · parent-child 트리 생성                                     │
│ │    · 기본 임베딩 생성 (L0/L1, chunk, parent_chunk)                │
│ │    · doc_node + chunk_node + parent_chunk_node 등록              │
│ │                                                              │
│ · chunks, embeddings, summaries 테이블 INSERT                    │
│ · status: stage2_complete                                       │
│                                                                │
│ ★ Stage 2 완료 = 시스템 등록 완료 = 검색 가능 상태                   │
│                                                                │
│ 산출물: /transformed/, chunks, embeddings, doc_node, chunk_node  │
└─────────────────────┬──────────────────────────────────────────┘
                      │
                      │ 모든 문서가 Stage 3로 진입
                      │
                      ▼
┌────────────────────────────────────────────────────────────────┐
│ Stage 3: Enrich (후처리 — 4개 층위)                              │
│                                                                │
│ analytical_depth에 따라 어느 층위까지 진행할지 결정                  │
│                                                                │
│                                                                │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ 3-a 형식 검증 (모든 문서 공통, LLM 없음)                    │   │
│ │                                                           │   │
│ │ · frontmatter schema 검증                                  │   │
│ │ · 필수 필드 존재 확인                                        │   │
│ │ · source_id ↔ doc_node 참조 무결성                           │   │
│ │ · chunk_node → parent_chunk_node 트리 일관성                  │   │
│ │ · ACL 스코프 유효성                                          │   │
│ │ · SHA256 해시 중복 재확인                                     │   │
│ │ · 상대 경로 링크 유효성                                       │   │
│ │                                                           │   │
│ │ 산출물: validation_log, 실패 시 validation_error 플래그       │   │
│ │ status: stage3a_complete                                    │   │
│ └───────────────────────┬──────────────────────────────────┘   │
│                         │                                      │
│     reference는 여기서 완결 (Stage 3-b 이상 진행 안 함)              │
│                         │ standard 이상                          │
│                         ▼                                      │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ 3-b 의미 심화 (standard 이상, LLM 사용)                     │   │
│ │                                                           │   │
│ │ · LLM으로 entity 추출                                       │   │
│ │   (기술 용어, 인물, 조직, 결정 등)                             │   │
│ │ · 기존 Graph entity와 매칭/병합                               │   │
│ │ · relationship edge 생성                                    │   │
│ │   (mentions, part_of, supersedes, contradicts 등)           │   │
│ │ · chunk_node에 mentions 엣지 연결                             │   │
│ │                                                           │   │
│ │ 산출물: entity_node, relationship edge                      │   │
│ │ status: stage3b_complete                                    │   │
│ └───────────────────────┬──────────────────────────────────┘   │
│                         │                                      │
│     standard는 여기서 완결                                         │
│                         │ deep 이상                              │
│                         ▼                                      │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ 3-c 합성 (deep 이상, 배치 — 일일)                             │   │
│ │                                                           │   │
│ │ · synthesis_node 생성                                       │   │
│ │   - 8가지 비용 제어 전략 적용:                                 │   │
│ │     1) 동시출현 게이트                                        │   │
│ │     2) 질의 기반 생성                                         │   │
│ │     3) 중심성 기반 우선순위                                     │   │
│ │     4) 계층적 집계                                            │   │
│ │     5) 증분 처리                                              │   │
│ │     6) 일일 예산 상한                                          │   │
│ │     7) 부실 노드 보관                                          │   │
│ │     8) 근거 검증                                              │   │
│ │ · consolidation (중복/모순/진화 분류)                          │   │
│ │   - 유사도 0.95 이상: 자동 병합                                │   │
│ │   - 0.70~0.95: 사람에게 확인 요청                              │   │
│ │ · connection inference (1-hop 관계 탐색)                      │   │
│ │ · stale 노드 감지 및 archive                                   │   │
│ │                                                           │   │
│ │ 산출물: synthesis_node, 노드 속성 업데이트 (freshness 등)      │   │
│ │ status: stage3c_complete                                    │   │
│ └───────────────────────┬──────────────────────────────────┘   │
│                         │                                      │
│     deep은 여기서 완결                                             │
│                         │ critical만                             │
│                         ▼                                      │
│ ┌──────────────────────────────────────────────────────────┐   │
│ │ 3-d 사람 검증 (critical만)                                  │   │
│ │                                                           │   │
│ │ · 전문가 검증 큐 등록                                        │   │
│ │ · 검토 결과 수집 → trust_score 갱신                          │   │
│ │ · 승인/반려/보류 상태 기록                                     │   │
│ │ · verified: true 플래그 설정                                 │   │
│ │                                                           │   │
│ │ 산출물: 검증 기록, 갱신된 trust_score                          │   │
│ │ status: stage3d_complete                                    │   │
│ └──────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────┘


─────────────────────────────────────────────────────────────────
                      파이프라인 밖 시스템 (원본 설계 기준)
─────────────────────────────────────────────────────────────────

  자동화된 기억 관리 (형성/통합/망각/관계추론/합성):
    · 형성(Formation): 새 문서 7단계 전처리 (Stage 2~3에 통합)
    · 통합(Consolidation): 벡터 유사도 기반 중복/모순 탐지 (Stage 3-c)
    · 망각(Forgetting): freshness 감쇠 → stale → archive (Stage 3-c)
    · 관계 추론(Connection Inference): 1-hop 탐색 (Stage 3-c)
    · 합성(Synthesis): 일일 배치 (Stage 3-c)

  Daily Digest (KMS Agent):
    · 새 소스 집계, entity 변동, article 상태 변화
    · 갱신 추천, 승격 후보
    · 운영자 대시보드로 노출

  수요 기반 심화 트리거:
    · 에이전트가 원본 fetch → 심화 제안
    · 사용자 명시 요청 "자세히 등록해줘"
    · 검색 결과 N회(기본 3) 이상 포함
    · 관리자 일괄 지정

  검증 가능한 삭제 (Verifiable Forgetting):
    · SHA256 해시 체인 추적
    · DEK 파기
    · 삭제 증명서 발급

  불변 감사 로그:
    · 해시 체인
    · SIEM 통합

  사서 에이전트 (Knowledge Curation):
    · topic_node article 작성 (Phase 2)
    · 큐레이션된 지식 우선 제공


─────────────────────────────────────────────────────────────────
                      커넥터/분해는 Stage 1 내부에서
─────────────────────────────────────────────────────────────────

원본 설계는 "커넥터" 개념을 Stage 1에 흡수:

  Stage 1의 책임:
    1. 접수 (원본 저장, SHA256)
    2. 분류 (6축 프로파일 Phase A 추론)
    3. 라우팅 (4경로 분기)
    4. 복합 입력 분해 (예: "메시지 + URL" 분해)

  즉, KMS 밖의 커넥터와 KMS 안의 Stage 1이 분리되지 않음.


─────────────────────────────────────────────────────────────────
                      원본 설계의 특징 정리
─────────────────────────────────────────────────────────────────

1. Stage 1이 매우 무거움
   접수 + 분류(6축) + 라우팅(4경로) + 분해

2. Stage 2가 depth에 따라 경로 A/B로 분기
   Tier, 청킹, L0/L1 생성 여부가 depth로 결정

3. Stage 3가 4개 층위로 세분화
   3-a(검증) / 3-b(entity) / 3-c(합성+배치) / 3-d(사람 검증)
   각 층위가 하나의 공정처럼 다뤄짐

4. Graph 등록이 Stage 2에서
   doc_node + chunk_node가 Stage 2 완료 시점에 그래프에 진입
   이 시점부터 검색 가능

5. 배치 작업(synthesis)이 Stage 3-c에 포함
   파이프라인 단계와 배치 작업이 같은 Stage에 섞여 있음

6. frontmatter에 6축 프로파일 전체 포함
   Agent가 문서를 받으면 6축을 모두 볼 수 있음

7. critical depth가 별도 값
   사람 검증 필수 여부가 depth 값에 섞임
```
