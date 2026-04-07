# Snapshot Scaffolding

원본 코드베이스를 **구조만 JSON 스냅샷으로 추출**하고, 이를 기반으로 다른 언어/환경에서 빈 패키지(스텁)를 자동 생성한 뒤, 핵심부터 실제 코드로 채워나가는 포팅(재구현) 방법론. 원본 코드를 직접 복사하지 않으므로 **저작권 문제 없는 clean room 포팅**이 가능하며, 진행률을 수치로 자동 측정할 수 있다.

## Key Points

- **Clean Room 포팅**: 원본 코드 복사 없이 구조만 추출 → 저작권 안전
- **3종 JSON 스냅샷**: Surface(전체 지도), Feature(기능 목록), Subsystem(디렉토리별 메타)
- **자동 스텁 생성**: 서브시스템 스냅샷을 순회하며 동일 템플릿의 빈 `__init__.py` 자동 생성
- **진행 상태 추적**: `PortingModule` dataclass의 `status` 필드(planned/mirrored/implemented)로 각 모듈의 포팅 단계 관리
- **매핑 테이블**: 원본 이름 → 우리 이름 딕셔너리로 대응 관계 명시적 선언
- **핵심 엔진 우선 전략**: 빈 구조를 먼저 만든 후 가장 중요한 모듈부터 실제 코드로 교체

## 3종 JSON 스냅샷 구조

### Surface Snapshot (전체 지도)
```json
{
  "archive_root": "원본 경로",
  "root_files": ["main.tsx", "commands.ts", ...],
  "root_dirs": ["assistant", "hooks", ...],
  "total_ts_like_files": 1902,
  "command_entry_count": 207
}
```

### Feature Snapshot (기능 목록)
```json
[
  {
    "name": "commit",
    "source_hint": "commands/commit/commit.tsx",
    "responsibility": "git commit 자동화 커맨드"
  }
]
```

### Subsystem Snapshot (디렉토리 메타)
```json
{
  "archive_name": "hooks",
  "package_name": "hooks",
  "module_count": 104,
  "sample_files": ["hooks/fileSuggestions.ts", ...]
}
```

## 워크플로

```
원본 코드베이스 분석
  → 3종 JSON 스냅샷 추출 (구조만, 코드 복사 아님)
  → 빈 패키지 자동 생성 (스텁)
  → 핵심 엔진 먼저 구현
  → 매핑 테이블 기반으로 진행률 자동 측정 (parity audit)
  → 나머지를 하나씩 실제 코드로 교체
```

## 장점과 한계

**장점:**
- 저작권 안전 (clean room)
- 포팅 진행률 자동 숫자화
- 모듈 간 관계가 먼저 확정되어 구현 순서 자유
- 새 팀원이 즉시 "어떤 모듈이 있고 각각 뭘 하는지" 파악 가능

**한계:**
- 원본의 내부 로직(알고리즘, 데이터 흐름)은 스냅샷에 담기지 않음 → 별도 분석 필요
- 원본 업데이트 시 스냅샷 수동 갱신 필요
- 구조만 있다고 구현이 된 건 아님 — 빈 방이 많으면 혼란 유발 가능

## Sources
- [260403_snapshot-scaffolding-methodology](../summaries/260403_snapshot-scaffolding-methodology.md)

## Related Concepts
- [parity-audit](parity-audit.md) — 진행률 측정 도구
