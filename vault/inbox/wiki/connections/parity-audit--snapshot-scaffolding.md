# Parity Audit ↔ Snapshot Scaffolding

Parity Audit은 Snapshot Scaffolding 방법론의 **핵심 측정 도구**로, 둘은 포팅 프로젝트에서 "구조 생성"과 "진행 검증"의 보완적 관계를 이룬다. Snapshot Scaffolding이 "빈 뼈대를 어떻게 자동 생성할 것인가"를 답한다면, Parity Audit은 "그 뼈대가 얼마나 채워졌는지 어떻게 객관적으로 측정할 것인가"를 답한다.

## From Snapshot Scaffolding perspective

Snapshot Scaffolding은 그 자체로는 "빈 방이 많은 건물"을 만드는 방법론이다. 이 방법론의 최대 한계로 지적된 "구조가 있다고 구현이 된 건 아님 — 빈 방이 많으면 혼란 유발 가능"이라는 문제를 해결하는 것이 Parity Audit이다:

- **진행 가시화**: 포팅 팀이 "어디까지 왔는지" 정확히 알 수 있음
- **우선순위 결정**: 매핑 테이블에서 "아직 빈 방"을 식별해 작업 대상 선정
- **완성도 증명**: 스텁 단계와 실제 구현 단계를 분리해 수치로 표현
- **신뢰할 수 있는 제로 베이스**: 자동 측정이 있으면 "이 모듈은 실제로 구현됐다"는 주장을 검증 가능

Parity Audit 없이 Snapshot Scaffolding만 사용하면 "구조는 다 만들었는데 실제 동작은 얼마나 되는지 모르겠다"는 상태가 되어 프로젝트 관리가 어렵다.

## From Parity Audit perspective

Parity Audit은 **Snapshot Scaffolding이 전제되어야 의미가 있다**. Parity Audit의 매핑 테이블은 "원본 이름 → 우리 이름"이라는 양측의 존재를 가정하며, 이 중 "우리 이름"은 Snapshot Scaffolding이 자동 생성한 스텁에서 시작된다. 즉:

- **측정 대상의 존재**: 스텁이 없으면 "아직 안 한 것"과 "할 계획이 없는 것"을 구분 불가
- **매핑의 근거**: Snapshot의 subsystem/feature 스냅샷이 매핑 테이블의 원본 목록을 제공
- **자동화 연쇄**: Snapshot 생성 → 스텁 생성 → 매핑 자동 생성 → Audit 실행이 파이프라인화 가능

Snapshot Scaffolding 없이 Parity Audit만 하면 매핑 테이블을 수동으로 작성해야 하고, 이는 원본 코드베이스의 변화를 추적하기 어렵게 만든다.

## 이중 측정의 의미

두 개념의 결합이 만드는 가장 큰 가치는 **구조 커버리지와 구현 비율의 분리**다:

| 측정 | 예시 | 의미 |
|------|------|------|
| Root file coverage | `(18, 18)` | 구조 100% — 모든 파일 스텁 존재 |
| Directory coverage | `(35, 35)` | 디렉토리 100% — 모든 서브시스템 존재 |
| Total file ratio | `(66, 1902)` | 실제 구현 3.5% — 대부분 빈 방 |

이 숫자 조합은 "뼈대는 다 세웠는데 살이 3.5%만 붙었다"는 상태를 정확히 전달한다. 이는 포팅 프로젝트의 **진행 체감(perceived progress)과 실제 진도(actual progress)의 괴리**를 객관적으로 드러내는 장치다. 프로젝트 관리자는 이 수치를 보고 "구조 작업은 끝났으니 이제 구현에 집중"이라는 판단을 내릴 수 있고, 팀원은 "빈 방이 97%이므로 작업할 게 많다"는 현실을 공유할 수 있다.

## Sources
- [260403_snapshot-scaffolding-methodology](../summaries/260403_snapshot-scaffolding-methodology.md)
