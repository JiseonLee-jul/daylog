---
source: raw/260413_kms-stage1-ingest-design.md
compiled: 2026-04-13
topics: [KMS, pipeline design, ingest, metadata, access control, audit, MECE]
---

# KMS Stage 1: 보관(Ingest) 설계 확정본

원본 설계 문서(02_MASTER-PLAN.md)의 비판적 회고를 거쳐 재구성된 KMS 파이프라인 Stage 1 설계. 업계 표준(Airflow, Dagster, Databricks Medallion)과 전문가 원칙(Beauchemin "Classify by behavior", Tunkelang "One axis one knob", Kleppmann "접수는 하류를 모른다")을 근거로, 원본 설계의 과잉 복잡성을 제거하고 책임을 단일화했다.

핵심 변경은 세 가지다. 첫째, 원본의 6축 프로파일 + 4경로 분기 + 4 Tier를 analytical_depth 단일 라우팅 축으로 축소했다. "파이프라인 행동을 실제로 바꾸는 것만 라우팅 축"이라는 원칙을 적용하면, 6축 중 처리 방법을 바꾸는 것은 analytical_depth 하나뿐이고, 나머지 5축(volatility, density, scale, lifespan, provenance)은 메타데이터로 저장만 하면 된다. 4경로와 4 Tier는 "어떻게 읽을 것인가"라는 하나의 결정을 세 개로 쪼개놓은 것이므로 파서 내부로 캡슐화했다.

둘째, 감사 메타데이터를 5W1H + Integrity + Outcome 8차원 프레임워크로 체계화하고, 권한 메타데이터를 owner/domain/sensitivity 3축 MECE 구조로 정립했다. actor_id 하나로 user/agent/connector/system을 통합하고, 권한 자동 유도 규칙(호출자 지정 > 커넥터 매핑 > 기본값)을 정의했다.

셋째, 시간/상태 모델을 최소화했다. LightRAG(created_at + updated_at + status 3개), GraphRAG(시간 필드 0개)를 참고하여, "저장 vs 계산" 원칙을 적용. freshness, is_stale 같은 파생값은 저장하지 않고 쿼리 시 계산한다. 상태는 status(lifecycle)와 progress(pipeline)를 분리하여 MECE를 확보했다.

## Related Concepts

- [pipeline-responsibility-separation](../concepts/pipeline-responsibility-separation.md)
- [analytical-depth](../concepts/analytical-depth.md)
- [mece-metadata](../concepts/mece-metadata.md)
- [document-node-pattern](../concepts/document-node-pattern.md)
