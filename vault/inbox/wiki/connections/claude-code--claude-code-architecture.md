# Claude Code ↔ Claude Code Architecture

Claude Code는 사용자 관점의 제품 개념이고, Claude Code Architecture는 그 내부 구현의 심층 구조다. 두 개념은 **"외부 기능"과 "내부 동작 원리"의 관계**로, 사용자가 경험하는 모든 기능이 아키텍처의 구체적 컴포넌트로 환원된다.

## From Claude Code perspective

사용자 관점에서 Claude Code는 "자연어로 지시하면 파일을 편집하고 명령을 실행하는 터미널 에이전트"다. 하지만 이 "마법 같은" 동작은 결국 **잘 설계된 아키텍처의 결과**다:

- `/loop`, `/schedule` 같은 자동화 기능 → 훅 시스템 + 쿼리 루프 후처리 단계
- 멀티레포 지원(`--add-dir`) → 설정 조기 로딩 단계의 경로 수집
- 세션 포크(`/branch`) → 브리지 시스템의 세션 격리
- Git worktrees + `/batch` → 코디네이터 모드의 리더-워커 구조
- 플러그인 시스템 → 스킬/훅/MCP 서버 통합

사용자가 "편리하다"고 느끼는 모든 기능은 **아키텍처의 4단계(시작 → 대화 루프 → 도구 실행 → 결과 표시)** 중 어딘가에 정확히 매핑된다. Claude Code를 깊이 이해하려면 아키텍처를 알아야 한다.

## From Claude Code Architecture perspective

아키텍처는 그 자체로 존재 이유가 없다 — 사용자 경험을 구현하기 위한 수단일 뿐이다. 각 구조 결정은 Claude Code의 제품 원칙(**안전성, 성능, 확장성**)에 기여한다:

- **쿼리 루프의 비동기 제너레이터**: 사용자가 "빠르게 응답한다"고 느끼는 근원 (성능)
- **도구 실행 파이프라인의 10단계**: Zod 검증 + 권한 확인 + 훅이 모든 도구 호출을 감싸 위험 행동 차단 (안전성)
- **4가지 권한 모드**: 사용자가 위험 수준에 따라 선택 가능 (안전성 + 사용성)
- **45+ 도구의 공통 인터페이스**: 새 도구 추가가 쉬움 (확장성)
- **플러그인 시스템 연계**: 스킬/훅/MCP가 아키텍처의 확장 지점으로 설계됨 (확장성)
- **브리지 시스템**: 로컬 CLI에 클라우드 CCR 기능을 투명하게 통합 (확장성)

## 8가지 설계 패턴의 의미

| 패턴 | 사용자 관점의 효과 |
|------|-------------------|
| Generator Streaming | 실시간으로 응답이 타이핑되는 것처럼 보임 |
| Feature Gate | 빌드에 포함된 기능만 로드 (시작 빠름) |
| Memoized Context | 동일 요청 반복 시 빠른 응답 |
| Withhold & Recover | 일시 오류가 사용자에게 노출되지 않음 |
| Lazy Import | CLI 시작 시간 단축 |
| Immutable State | "어라 상태가 이상해" 문제 없음 |
| Interruption Resilience | Ctrl+C 해도 작업 상태 유지 |
| Dependency Injection | 헤드리스/REPL/코디네이터 모드 등 교체 가능 |

이 패턴들은 모두 **사용자가 "편리하다"고 느끼는 순간을 만드는 엔지니어링 결정**이다. 아키텍처 없이는 그 경험이 없고, 사용자 없이는 아키텍처가 의미 없다 — 두 개념은 동전의 양면이다.

## Sources
- [260331_claude_code_hidden_features](../summaries/260331_claude_code_hidden_features.md)
- [wikidocs_net_338204](../summaries/wikidocs_net_338204.md)
