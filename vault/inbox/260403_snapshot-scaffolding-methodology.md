# 스냅샷 기반 스캐폴딩 방법론

> 다른 기술/프로젝트의 구조를 분석해서, 우리 언어/환경에서 동일한 뼈대를 먼저 만들고,
> 이후 핵심부터 실제 코드를 채워나가는 포팅(재구현) 전략.
> Claw Code 프로젝트에서 추출한 방법론.

---

## 전체 흐름 요약

```
원본 코드베이스 분석
  → JSON 스냅샷 추출 (구조만 저장, 코드 복사 아님)
  → 빈 패키지 자동 생성 (스텁 = 이름표만 있는 빈 모듈)
  → 핵심 엔진 먼저 구현
  → 나머지를 하나씩 채워나감
  → 진행률 자동 측정 (패리티 감사)
```

---

## Step 1: 원본 분석 → 3종류의 JSON 스냅샷 추출

원본 코드를 직접 복사하지 않는다.
대신, **어떤 파일이 있고 / 어떤 기능이 있고 / 어떤 구조인지**만 JSON으로 정리한다.

### 1-1. 전체 표면 스냅샷 (surface snapshot)

원본 코드베이스의 "지도". 전체 규모와 구성을 한눈에 보여주는 파일.

```json
// archive_surface_snapshot.json
{
  "archive_root": "원본 소스의 루트 경로",
  "root_files": ["main.tsx", "commands.ts", "tools.ts", ...],
  "root_dirs": ["assistant", "hooks", "plugins", ...],
  "total_ts_like_files": 1902,
  "command_entry_count": 207,
  "tool_entry_count": 184
}
```

**만드는 법**: 원본 소스 디렉토리를 스캔해서 아래 정보를 수집
- 최상위 파일 목록 (root_files)
- 최상위 디렉토리 목록 (root_dirs)
- 전체 파일 수
- 주요 카테고리별 엔트리 수 (커맨드, 도구 등 프로젝트에 맞게)

### 1-2. 기능 단위 스냅샷 (feature snapshot)

원본의 **기능 목록**을 개별 엔트리로 나열한 배열.
기능 카테고리별로 별도 파일을 만든다.

```json
// commands_snapshot.json — 커맨드(사용자가 호출하는 기능) 목록
[
  {
    "name": "commit",
    "source_hint": "commands/commit/commit.tsx",
    "responsibility": "git commit 자동화 커맨드"
  },
  {
    "name": "review",
    "source_hint": "commands/review/index.ts",
    "responsibility": "코드 리뷰 커맨드"
  }
]
```

```json
// tools_snapshot.json — 도구(AI가 호출하는 함수) 목록
[
  {
    "name": "BashTool",
    "source_hint": "tools/BashTool/BashTool.tsx",
    "responsibility": "셸 명령어 실행 도구"
  }
]
```

**각 엔트리의 3개 필드**:
| 필드 | 설명 | 예시 |
|------|------|------|
| `name` | 기능의 이름 | "BashTool" |
| `source_hint` | 원본에서의 파일 경로 (출처 추적용) | "tools/BashTool/BashTool.tsx" |
| `responsibility` | 이 기능이 하는 일 한줄 설명 | "셸 명령어 실행 도구" |

### 1-3. 서브시스템 스냅샷 (subsystem snapshot)

원본의 **디렉토리(하위 시스템)별** 메타데이터.
디렉토리 하나당 JSON 파일 하나.

```json
// subsystems/hooks.json
{
  "archive_name": "hooks",
  "package_name": "hooks",
  "module_count": 104,
  "sample_files": [
    "hooks/fileSuggestions.ts",
    "hooks/notifs/useAutoModeUnavailableNotification.ts",
    "hooks/toolPermission/PermissionContext.ts"
  ]
}
```

**4개 필드**:
| 필드 | 설명 | 예시 |
|------|------|------|
| `archive_name` | 원본 디렉토리 이름 | "hooks" |
| `package_name` | 우리 프로젝트에서의 패키지 이름 | "hooks" |
| `module_count` | 원본에 있던 파일 수 | 104 |
| `sample_files` | 대표 파일 목록 (전체가 아닌 샘플) | ["hooks/fileSuggestions.ts", ...] |

---

## Step 2: 빈 패키지 생성 (스텁 = 이름표만 있는 빈 모듈)

서브시스템 스냅샷을 읽어서 **빈 Python 패키지**를 만든다.
모든 패키지가 동일한 코드 템플릿을 사용한다.

### 디렉토리 구조

```
src/
├── reference_data/                    ← JSON 스냅샷 보관소
│   ├── archive_surface_snapshot.json  ← 전체 지도
│   ├── commands_snapshot.json         ← 기능 목록 (커맨드)
│   ├── tools_snapshot.json            ← 기능 목록 (도구)
│   └── subsystems/                    ← 서브시스템별 메타데이터
│       ├── assistant.json
│       ├── hooks.json
│       └── ... (디렉토리 수만큼)
│
├── assistant/                         ← 빈 패키지 (스텁)
│   └── __init__.py
├── hooks/                             ← 빈 패키지 (스텁)
│   └── __init__.py
└── ... (서브시스템 수만큼)
```

### 빈 패키지의 코드 템플릿

모든 `__init__.py`가 아래 패턴을 따른다:

```python
"""Python package placeholder for the archived `{이름}` subsystem."""

from __future__ import annotations

import json
from pathlib import Path

# 자기 서브시스템의 JSON 스냅샷을 읽는다
SNAPSHOT_PATH = Path(__file__).resolve().parent.parent / 'reference_data' / 'subsystems' / '{이름}.json'
_SNAPSHOT = json.loads(SNAPSHOT_PATH.read_text())

# 4개 상수를 노출한다
ARCHIVE_NAME = _SNAPSHOT['archive_name']     # 원본 디렉토리 이름
MODULE_COUNT = _SNAPSHOT['module_count']      # 원본에 있던 파일 수
SAMPLE_FILES = tuple(_SNAPSHOT['sample_files'])  # 대표 파일 목록
PORTING_NOTE = f"Python placeholder package for '{ARCHIVE_NAME}' with {MODULE_COUNT} archived module references."

__all__ = ['ARCHIVE_NAME', 'MODULE_COUNT', 'PORTING_NOTE', 'SAMPLE_FILES']
```

**핵심**: 이 파일은 아무 기능도 실행하지 않는다. 
"나는 hooks 패키지이고, 원본에 104개 파일이 있었어"라는 정보만 제공한다.

---

## Step 3: 기능 목록을 Python 객체로 로드

JSON 스냅샷의 기능 목록(커맨드/도구)을 Python 데이터 객체로 변환한다.

### 공통 데이터 모델

```python
# models.py
@dataclass(frozen=True)   # frozen=True: 한번 만들면 수정 불가 (안전)
class PortingModule:
    name: str              # 기능 이름
    responsibility: str    # 하는 일 설명
    source_hint: str       # 원본 파일 경로
    status: str = 'planned'  # 상태: planned(예정) / mirrored(구조만) / implemented(구현완료)
```

### 로딩 패턴

```python
# commands.py
from functools import lru_cache  # lru_cache: 한번 읽으면 결과를 저장해서 다시 안 읽음

SNAPSHOT_PATH = Path(__file__).parent / 'reference_data' / 'commands_snapshot.json'

@lru_cache(maxsize=1)  # 처음 호출할 때만 JSON을 읽고, 이후는 저장된 결과를 재사용
def load_command_snapshot() -> tuple[PortingModule, ...]:
    raw_entries = json.loads(SNAPSHOT_PATH.read_text())
    return tuple(
        PortingModule(
            name=entry['name'],
            responsibility=entry['responsibility'],
            source_hint=entry['source_hint'],
            status='mirrored',      # JSON에서 로드했으니 "구조만 있음" 상태
        )
        for entry in raw_entries
    )

PORTED_COMMANDS = load_command_snapshot()  # 모듈 임포트 시 자동으로 로드됨
```

tools.py도 동일한 패턴. 카테고리가 늘어나면 같은 패턴을 복제하면 된다.

---

## Step 4: 매핑 테이블 + 진행률 자동 측정 (패리티 감사)

### 매핑 테이블

원본 파일/디렉토리 이름 → 우리 프로젝트의 대응 파일/디렉토리 이름을 딕셔너리로 선언한다.

```python
# parity_audit.py

# 최상위 파일 매핑: "원본 이름" → "우리 이름"
ARCHIVE_ROOT_FILES = {
    'QueryEngine.ts': 'QueryEngine.py',
    'main.tsx': 'main.py',
    'commands.ts': 'commands.py',
    # ...
}

# 디렉토리 매핑: "원본 디렉토리" → "우리 패키지/파일"
ARCHIVE_DIR_MAPPINGS = {
    'assistant': 'assistant',       # 디렉토리 → 디렉토리 (1:1)
    'commands': 'commands.py',      # 디렉토리 → 단일 파일 (압축)
    'native-ts': 'native_ts',      # 이름 변환 (하이픈 → 언더스코어)
    # ...
}
```

### 진행률 측정 코드

```python
def run_parity_audit():
    current_entries = {path.name for path in CURRENT_ROOT.iterdir()}
    
    # 매핑 테이블의 각 항목이 실제로 존재하는지 체크
    root_hits = [t for t in ARCHIVE_ROOT_FILES.values() if t in current_entries]
    dir_hits  = [t for t in ARCHIVE_DIR_MAPPINGS.values() if t in current_entries]
    
    # 전체 파일 수 비교
    python_files = sum(1 for p in CURRENT_ROOT.rglob('*.py'))
    
    return ParityAuditResult(
        root_file_coverage=(len(root_hits), len(ARCHIVE_ROOT_FILES)),  # 예: (18, 18)
        directory_coverage=(len(dir_hits), len(ARCHIVE_DIR_MAPPINGS)), # 예: (35, 35)
        total_file_ratio=(python_files, 1902),                        # 예: (66, 1902)
    )
```

**결과 해석**:
- `(18, 18)` = 구조 100% 완성
- `(66, 1902)` = 실제 구현 3.5%

---

## 우리가 새 프로젝트에 적용할 때의 체크리스트

### 준비물
- [ ] 원본 코드베이스 접근 (읽기만 가능하면 됨)
- [ ] 우리가 사용할 언어/환경 결정

### 실행 순서

```
1. 원본 소스 디렉토리를 스캔하는 스크립트 작성
   → archive_surface_snapshot.json 생성
   → subsystems/{이름}.json 생성 (디렉토리 수만큼)

2. 원본의 주요 기능 카테고리를 식별
   (Claw Code에서는 "커맨드"와 "도구"가 해당)
   → {카테고리}_snapshot.json 생성

3. 공통 데이터 모델 정의
   → PortingModule 같은 dataclass 작성

4. 빈 패키지 자동 생성 스크립트 작성
   → subsystems/*.json을 순회하며 __init__.py 템플릿 적용

5. 매핑 테이블 작성
   → 원본 이름 → 우리 이름 딕셔너리

6. 패리티 감사 함수 작성
   → 매핑 테이블 순회하며 존재 여부 체크

7. 핵심 엔진부터 구현 시작
   → 가장 중요한 모듈을 식별하고, 스텁을 실제 코드로 교체
```

### JSON 스냅샷 생성 스크립트 예시

```python
"""원본 코드베이스를 스캔하여 JSON 스냅샷을 생성하는 스크립트."""

import json
from pathlib import Path

def scan_codebase(source_root: Path, extensions: set[str] = {'.ts', '.tsx', '.js'}):
    """원본 코드베이스를 스캔하여 3종류의 JSON을 생성한다."""
    
    all_files = [p for p in source_root.rglob('*') if p.suffix in extensions]
    root_files = [p.name for p in source_root.iterdir() if p.is_file() and p.suffix in extensions]
    root_dirs = [p.name for p in source_root.iterdir() if p.is_dir() and not p.name.startswith('.')]
    
    # 1. 전체 표면 스냅샷
    surface = {
        "archive_root": str(source_root),
        "root_files": sorted(root_files),
        "root_dirs": sorted(root_dirs),
        "total_file_count": len(all_files),
    }
    
    # 2. 서브시스템 스냅샷 (디렉토리별)
    subsystems = {}
    for dir_name in root_dirs:
        dir_path = source_root / dir_name
        dir_files = [str(p.relative_to(source_root)) for p in dir_path.rglob('*') if p.suffix in extensions]
        subsystems[dir_name] = {
            "archive_name": dir_name,
            "package_name": dir_name.replace('-', '_'),  # 하이픈 → 언더스코어
            "module_count": len(dir_files),
            "sample_files": sorted(dir_files)[:25],  # 최대 25개 샘플
        }
    
    return surface, subsystems

# 사용법:
# surface, subsystems = scan_codebase(Path("원본/소스/경로"))
# Path("reference_data/archive_surface_snapshot.json").write_text(json.dumps(surface, indent=2))
# for name, data in subsystems.items():
#     Path(f"reference_data/subsystems/{name}.json").write_text(json.dumps(data, indent=2))
```

---

## 이 방법론의 장점과 한계

### 장점
- 원본 코드를 복사하지 않으므로 **저작권 문제 없음** (클린룸)
- 포팅 진행률을 **숫자로 자동 측정** 가능
- 모듈 간 관계가 먼저 확정되므로 **순서 상관없이 구현** 가능
- 새 팀원이 와도 "어떤 모듈이 있고 각각 뭘 하는지" 즉시 파악 가능

### 한계
- 원본의 **내부 로직**(알고리즘, 데이터 흐름)은 스냅샷에 담기지 않음
  → 별도로 분석하여 문서화하거나, 동작을 관찰하여 재구현해야 함
- 원본이 업데이트되면 스냅샷도 **수동으로 갱신**해야 함
- 구조가 있다고 구현이 된 건 아님 — **빈 방이 많으면 혼란** 유발 가능
