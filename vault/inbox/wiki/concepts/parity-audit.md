# Parity Audit

포팅 프로젝트에서 **원본 대비 진행률을 자동으로 수치화**하는 검증 메커니즘. 원본 파일/디렉토리 이름과 우리 프로젝트의 대응 이름을 매핑 테이블로 선언하고, 매핑의 각 항목이 실제로 존재하는지 체크해 "구조 커버리지"와 "구현 비율"을 독립적으로 리포트한다.

## Key Points

- **매핑 테이블 기반**: 원본 → 우리 이름 딕셔너리로 대응 관계 명시적 선언
- **이중 측정**: 구조 커버리지(이름만 존재)와 구현 비율(실제 코드 양)을 분리
- **자동 측정**: 매핑 순회하며 파일시스템 체크 → 수동 개입 없이 진행률 계산
- **해석 가능한 숫자**: `(18, 18)`처럼 튜플로 표현, "18개 중 18개 완성 = 100%"처럼 직관적
- **구조와 구현의 차이를 드러냄**: 구조 100%인데 구현 3.5%인 경우 "빈 방이 많음"을 노출
- **포팅 진행률 가시화**: 작업자가 다음에 집중할 영역을 데이터 기반으로 결정

## 매핑 테이블 예시

```python
# parity_audit.py

# 최상위 파일 매핑
ARCHIVE_ROOT_FILES = {
    'QueryEngine.ts': 'QueryEngine.py',
    'main.tsx': 'main.py',
    'commands.ts': 'commands.py',
}

# 디렉토리 매핑 (타입 변환 허용)
ARCHIVE_DIR_MAPPINGS = {
    'assistant': 'assistant',      # 디렉토리 → 디렉토리 (1:1)
    'commands': 'commands.py',     # 디렉토리 → 단일 파일 (압축)
    'native-ts': 'native_ts',      # 이름 변환 (하이픈 → 언더스코어)
}
```

## 측정 코드

```python
def run_parity_audit():
    current_entries = {path.name for path in CURRENT_ROOT.iterdir()}

    # 매핑 테이블의 각 항목이 실제로 존재하는지 체크
    root_hits = [t for t in ARCHIVE_ROOT_FILES.values() if t in current_entries]
    dir_hits  = [t for t in ARCHIVE_DIR_MAPPINGS.values() if t in current_entries]

    # 전체 파일 수 비교
    python_files = sum(1 for p in CURRENT_ROOT.rglob('*.py'))

    return ParityAuditResult(
        root_file_coverage=(len(root_hits), len(ARCHIVE_ROOT_FILES)),
        directory_coverage=(len(dir_hits), len(ARCHIVE_DIR_MAPPINGS)),
        total_file_ratio=(python_files, 1902),
    )
```

## 결과 해석

| 결과 | 의미 |
|------|------|
| `(18, 18)` | 구조 100% 완성 (모든 매핑 대응 파일 존재) |
| `(35, 35)` | 디렉토리 구조 100% 완성 |
| `(66, 1902)` | 실제 구현 3.5% (파일 수 기준) |

구조 커버리지와 구현 비율이 분리되어 있다는 게 핵심이다 — "뼈대는 다 세웠지만 살이 3.5%만 붙었다"는 상태를 정확히 표현할 수 있다. 이는 대규모 포팅 프로젝트에서 **진행 체감(perceived progress)과 실제 진도(actual progress)의 괴리**를 드러내는 데 유용하다.

## Sources
- [260403_snapshot-scaffolding-methodology](../summaries/260403_snapshot-scaffolding-methodology.md)

## Related Concepts
- [snapshot-scaffolding](snapshot-scaffolding.md) — Parity Audit이 작동하는 상위 방법론
