---
source: raw/260403_snapshot-scaffolding-methodology.md
compiled: 2026-04-07
topics: [snapshot-scaffolding, parity-audit, clean-room-porting, code-porting, porting-methodology]
---

# Summary: 스냅샷 기반 스캐폴딩 방법론 — Clean Room 포팅 전략

원본 코드를 직접 복사하지 않고 다른 언어/환경에서 동일한 구조를 재구현하는 방법론으로, Claw Code 프로젝트에서 추출되었다. 핵심 아이디어는 원본 코드베이스를 분석해 **구조만 JSON으로 추출**하고, 이를 기반으로 빈 패키지(스텁)를 자동 생성한 뒤, 핵심 엔진부터 실제 코드로 채워나가는 것이다. 이 접근은 저작권 문제 없이(clean room) 포팅 진행률을 숫자로 자동 측정할 수 있다는 이점을 제공한다.

방법론은 3종류의 JSON 스냅샷으로 원본을 표현한다. (1) **Surface Snapshot**은 원본 코드베이스의 "지도"로 루트 파일/디렉토리 목록과 전체 규모를 담는다. (2) **Feature Snapshot**은 기능 카테고리별 목록(예: 커맨드, 도구)으로, 각 엔트리는 `name`/`source_hint`/`responsibility` 3개 필드를 가진다. (3) **Subsystem Snapshot**은 디렉토리별 메타데이터로 `archive_name`/`package_name`/`module_count`/`sample_files` 4개 필드를 가진다.

빈 패키지 생성 단계에서는 각 서브시스템마다 동일한 `__init__.py` 템플릿을 적용한다. 이 템플릿은 자기 서브시스템의 JSON 스냅샷을 읽어 `ARCHIVE_NAME`, `MODULE_COUNT`, `SAMPLE_FILES`, `PORTING_NOTE` 4개 상수를 노출한다 — 실제 기능은 아무것도 하지 않지만 "나는 hooks 패키지이고 원본에 104개 파일이 있었다"는 정보를 제공한다. 기능 목록은 `@dataclass(frozen=True)`로 정의된 `PortingModule` 객체로 로드되며, `status` 필드로 `planned`/`mirrored`/`implemented` 3단계 진행 상태를 추적한다.

**Parity Audit**는 이 방법론의 핵심 측정 도구다. 원본 파일/디렉토리 이름 → 우리 프로젝트 대응 이름을 딕셔너리로 선언하고, 매핑의 각 항목이 실제로 존재하는지 체크해 진행률을 숫자로 출력한다. 예를 들어 `root_file_coverage=(18, 18)`은 구조 100% 완성을, `total_file_ratio=(66, 1902)`는 실제 구현 3.5%를 의미한다. 이 방법론의 한계는 원본의 내부 로직(알고리즘, 데이터 흐름)이 스냅샷에 담기지 않아 별도 분석이 필요하다는 점, 원본 업데이트 시 스냅샷 수동 갱신이 필요하다는 점, 구조만 있다고 구현이 된 건 아니므로 "빈 방"이 혼란을 유발할 수 있다는 점이다.

## Key Points
- 3종류 JSON 스냅샷: Surface(지도), Feature(기능 목록), Subsystem(디렉토리 메타)
- 빈 패키지 자동 생성: 모든 `__init__.py`가 동일 템플릿, 자기 스냅샷 읽어 상수 노출
- `PortingModule` dataclass로 기능 진행 상태 추적 (planned/mirrored/implemented)
- Parity Audit: 매핑 테이블 기반으로 구조/구현 진행률 자동 숫자화
- Clean room 포팅: 원본 복사 없이 구조만 추출 → 저작권 안전
- 한계: 내부 로직은 별도 분석 필요, 스냅샷 수동 갱신, 빈 구조의 혼란 가능성

## Related Concepts
- [snapshot-scaffolding](../concepts/snapshot-scaffolding.md)
- [parity-audit](../concepts/parity-audit.md)
