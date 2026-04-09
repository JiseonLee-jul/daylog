---
source: raw/wikidocs_net_338204.md
compiled: 2026-04-07
topics: [claude-code, architecture, tool-system, query-loop, permission-modes, hooks, tui, bridge-system, coordinator-mode]
---

# Summary: 클로드 코드 소스 코드 분석서

Claude Code 공식 CLI 도구(TypeScript + React TUI + Zustand, 약 1,884 소스 파일)의 내부 아키텍처를 총체적으로 분석한 문서. 핵심 테제는 "사용자의 자연어 요청을 AI가 판단 → 적절한 도구 실행 → 권한 확인 → 결과를 AI에게 피드백하는 루프"이며, 모든 설계는 **안전성, 성능, 확장성** 세 원칙을 따른다. 전체 구조는 4단계(시작 → 대화 루프 → 도구 실행 → 결과 표시)로 요약되고, 5가지 실행 모드(REPL, 헤드리스, 코디네이터, 브리지, 어시스턴트)를 지원한다.

**쿼리 루프**는 비동기 제너레이터 패턴으로 토큰 단위 스트리밍을 지원하는 핵심 엔진이다. 각 턴은 메시지 전처리(토큰 압축), API 스트리밍 호출, 에러 보류 및 복구, 도구 실행, 후처리(훅 실행, 예산 확인)로 구성된다. **도구 실행 파이프라인**은 이름 조회 → 중단 신호 확인 → Zod 검증 → PreToolUse 훅 → 권한 확인 → 도구 실행 → 결과 매핑 → 오버사이즈 처리 → PostToolUse 훅 → 텔레메트리 순서로 진행되며, 동시성 파티셔닝(연속된 안전 도구는 병렬, 비안전 도구는 단독 실행)이 적용된다.

**도구 시스템**은 45개 이상의 내장 도구가 공통 인터페이스(`name`, `inputSchema`, `call()`, `checkPermissions()`, `validateInput()`)를 따른다. 주요 도구로 BashTool(Tree-sitter로 AST 분석, 기본 거부 설계), FileEditTool(퍼지 매칭, Git diff 생성), AgentTool(서브에이전트 위임), GrepTool(ripgrep 기반 검색) 등이 있다. **권한 시스템**은 4가지 모드(Default, Auto, Plan, Bypass)와 5단계 규칙 우선순위(로컬 → 프로젝트 → 사용자 → 플래그 → 정책)로 구성되며, **훅 시스템**은 SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop 이벤트에 사용자 정의 동작을 주입할 수 있다.

**UI 레이어**는 React 기반 자체 제작 프레임워크로 Yoga 레이아웃 엔진, 이중 버퍼링, 객체 풀링, 더티 추적, 프레임 조절 최적화를 포함한다. **브리지 시스템**은 로컬 터미널과 클라우드 CCR(Claude Remote Runtime) 연결을 담당해 최대 32개 병렬 세션 관리, 순환 버퍼 기반 메시지 중복 제거를 수행한다. **코디네이터 모드**는 리더 에이전트가 여러 워커에게 작업을 분배하는 구조로 조사(병렬) → 합성(순차 + 리더 직접 이해) → 구현(영역별 순차) → 검증(병렬) 순서로 진행된다. **메모리 시스템**은 `~/.claude/projects/{slug}/memory/`에 user/feedback/project/reference 4가지 타입으로 저장된다.

핵심 설계 패턴 8가지: Generator Streaming(실시간 이벤트), Feature Gate(빌드 시점 데드코드 제거), Memoized Context(세션 수명 캐싱), Withhold & Recover(복구 가능 에러 버퍼링), Lazy Import(순환 의존성 회피), Immutable State(예측 가능한 상태 변화), Interruption Resilience(트랜스크립트 사전 저장), Dependency Injection(테스트/모드별 교체). 이 패턴들은 모두 앞서 언급한 안전성/성능/확장성 원칙을 구현하는 수단이다.

## Key Points

- 4단계 구조: 시작 → 대화 루프 → 도구 실행 → 결과 표시
- 5가지 실행 모드: REPL, 헤드리스, 코디네이터, 브리지, 어시스턴트
- 쿼리 루프: 비동기 제너레이터 + 토큰 단위 스트리밍 + 동시성 파티셔닝
- 도구 실행 파이프라인: Zod 검증부터 PostToolUse 훅까지 10단계 순차 흐름
- 45+ 내장 도구가 공통 인터페이스(`name`, `inputSchema`, `call`, `checkPermissions`, `validateInput`) 준수
- 4가지 권한 모드 (Default/Auto/Plan/Bypass) + 5단계 규칙 우선순위
- 훅 5개 이벤트 (SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop)
- React + Yoga 기반 자체 TUI 프레임워크 (이중 버퍼링, 객체 풀링, 프레임 조절)
- 브리지 시스템으로 로컬-클라우드 연결, 최대 32개 병렬 세션
- 코디네이터 모드: 리더-워커 구조의 멀티 에이전트 조정
- 메모리 시스템 4타입: user, feedback, project, reference
- 8가지 핵심 설계 패턴

## Related Concepts

- [claude-code](../concepts/claude-code.md)
- [claude-code-architecture](../concepts/claude-code-architecture.md)
- [claude-code-auto-mode](../concepts/claude-code-auto-mode.md)
