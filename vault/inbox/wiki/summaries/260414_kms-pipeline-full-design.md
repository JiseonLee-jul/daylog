---
source: raw/260414_kms-pipeline-full-design.md
compiled: 2026-04-14
topics: [KMS, pipeline design, ingest, parse, understand, graph, metadata, chunking, LightRAG, GraphRAG]
---

# Summary: KMS 파이프라인 전체 설계 — Stage 1/2/3 최종

02_MASTER-PLAN.md 원본 설계의 비판적 회고를 거쳐 재구성한 KMS 데이터 처리 파이프라인의 3단계(보관/읽기/이해) 확정 설계. "관심사 분리", "최소 개념", "SSOT는 외부 원본(Federated Origin + Cached Derivatives)"을 핵심 원칙으로 삼고, Beauchemin, Tunkelang, Kleppmann, Dagster/Schrock 등 업계 전문가의 원칙을 근거로 원본 설계의 과잉 복잡성을 제거했다. 파이프라인은 Stage 1(보관) → Stage 2(읽기) → Stage 3(이해) → Graph 등록으로 단선화되고, synthesis·consolidation·stale 감지는 파이프라인 밖 배치 스케줄러로, 사람 검증·depth 승격은 이벤트 기반 워크플로우로 분리되었다.

Stage 1은 "원본을 있는 그대로 받아서 보관한다. 파일 내용을 읽지 않는다"는 단일 책임만 수행한다. source_record는 19개 필드로 구성되며 파일 식별(5) + 무결성(2) + 감사(4) + 권한(3) + 처리 제어(2) + 상태(2) 구조다. 260413 Stage 1 확정본 대비 가장 큰 차이는 처리 제어 필드를 analytical_depth 단일 필드에서 processing_depth(reference/standard/deep) + requires_review(true/false)로 분리한 점이다. critical depth를 별도 상태 축(review)으로 떼어내어 "depth는 처리 깊이, review는 별도 직교 축"이라는 MECE를 확보했다. 두 필드 모두 우선순위 체인(호출자 지정 > source_location > sensitivity > domain > file_ext > 기본값)으로 자동 결정된다.

Stage 2는 원본 파일을 마크다운으로 변환한다. depth를 모르고 파일 형식을 파서 내부에서 감지하여(.pdf → Docling+OCR, .docx → Docling, .mp3/.wav → Whisper, .csv/.json → 구조화 파서, .md → 통과) 적절한 도구로 처리한다. 출력 content.md의 frontmatter는 식별(2) + 출처(3) + 요약(2) + 지식 신호(2) + 콘텐츠 메타(1) + 상태(3) = 14개 필드로 고정됐으며, 요약·지식 신호 필드는 Stage 3에서 채워진다. 청킹은 depth와 독립이고 "문서 타입(코드/표/채팅/산문) + 구조 유무(heading 충분/부족) + 크기(<2K flat / ≥2K parent-child)" 3축으로 결정한다. 원본 설계의 "reference=flat, standard=parent-child" 가설은 학술/프로덕션/LightRAG 코드 모두에서 근거가 없다는 리서치 결과에 따라 폐기됐다.

Stage 3는 마크다운과 청크를 지식 객체로 변환하고 그래프에 등록한다. 순서는 검증(동기 per-doc, defense in depth) → 깊이별 처리(공통: L0 요약+임베딩 / standard↑: L1 요약+엔티티+관계) → 그래프 등록(doc_node, chunk_node, entity_node, edges를 트랜잭션 단위 커밋) → frontmatter 갱신(abstract/overview/tags/entities) → 후속 처리 큐 등록(deep → synthesis, requires_review → 사람 검증)이다. L0/L1 크기는 token_count 기반 적응형(<200 생성 안 함, 200~2K: 200토큰, 2K~20K: 350, >20K: 500)으로 결정된다. 임베딩은 MVP에서 단일 모델(온프레미스 EmbeddingGemma 300M, 클라우드 Gemini Embedding 2)을 쓰고, 목적별 확장은 Phase 2로 미룬다. 원본 설계의 4층위 3-a/b/c/d는 평면화되어 depth 분기로 대체되었고, synthesis는 파이프라인 밖으로 이동, 검증 시점은 그래프 등록 후 → 등록 전으로 당겨졌다.

그래프 구조는 Neo4j 스타일의 doc_node ← chunk_node → entity_node 3계층에 parent_chunk_node(parent-child 구조일 때), has_attachment 엣지(parent_id 구조), mentions 엣지(chunk → entity), relationship 엣지(entity ↔ entity)를 갖는다. 문서 노드는 메타데이터 단일 관리점, 출처 추적, 접근 제어 앵커, 컨텍스트 확장(chunk → sibling 탐색) 역할을 모두 수행한다. LightRAG이 entity 중심 그래프(doc/chunk 노드 없음)라는 점에서 어댑터 레이어가 필요하며, 구현 방식(어댑터/완전 커스텀/하이브리드)은 미결이다. MVP는 Stage 1/2/3 + processing_depth + requires_review + 3계층 노드 + 단일 임베딩 + 동기 검증 + LightRAG 어댑터까지로 한정하고, synthesis 생성, connection inference, 목적별 임베딩, 멀티모달, cross-doc 배치 검증, 큐레이션 기여(topic_article), depth 강등은 Phase 2 이후로 유예한다.

## Key Points

- 3단계 단선 파이프라인: Stage 1(보관) → Stage 2(읽기) → Stage 3(이해) → Graph
- 파이프라인 밖 2축: 배치 스케줄러(synthesis/consolidation/stale), 이벤트(사람 검증/depth 승격)
- Stage 1 처리 제어 필드를 analytical_depth 단일 필드 → processing_depth + requires_review 2개로 분리 (MECE 확보)
- Stage 2 청킹은 depth와 독립, 문서 타입/구조/크기 3축으로 결정 (원본 "depth별 차등 청킹" 가설 폐기)
- Stage 3는 검증을 그래프 등록 전으로 당기고 synthesis를 파이프라인 밖으로 분리, depth 분기로 4층위 평면화
- L0/L1 요약 크기는 token_count 기반 적응형으로 결정
- 그래프는 Neo4j 스타일 doc/chunk/entity 3계층 + parent-child + attachment + mentions + relationship
- SSOT는 외부 원본(Federated Origin + Cached Derivatives), KMS는 파생물 캐시
- 그래프 시간 모델 최소화: 2 timestamp + 1 status, 파생값(freshness 등)은 쿼리 시 계산
- synthesis는 비용 대비 이득이 FAQ/factual 질의에서 입증 안 되어 MVP 제외
- LightRAG은 entity 중심이라 Neo4j 스타일 doc/chunk 노드와 어댑터 필요

## Related Concepts

- [kms-pipeline-architecture](../concepts/kms-pipeline-architecture.md)
- [ingest-stage-design](../concepts/ingest-stage-design.md)
- [parse-stage-design](../concepts/parse-stage-design.md)
- [understand-stage-design](../concepts/understand-stage-design.md)
- [chunking-strategy](../concepts/chunking-strategy.md)
- [document-node-pattern](../concepts/document-node-pattern.md)
- [processing-depth](../concepts/processing-depth.md)
- [mece-metadata](../concepts/mece-metadata.md)
- [pipeline-responsibility-separation](../concepts/pipeline-responsibility-separation.md)
- [frontmatter-schema](../concepts/frontmatter-schema.md)
- [federated-origin-cached-derivatives](../concepts/federated-origin-cached-derivatives.md)
- [analytical-depth](../concepts/analytical-depth.md)
