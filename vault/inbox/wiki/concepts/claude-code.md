# Claude Code

Anthropic이 개발한 터미널 기반 코딩 에이전트. **TypeScript + React TUI + Zustand** 기반 약 1,884 소스 파일로 구성된 공식 CLI 도구로, 단순한 코드 생성기를 넘어 **개발 워크플로 전체를 자동화하는 플랫폼**으로 진화 중이다. 모든 설계는 **안전성, 성능, 확장성** 세 원칙을 따르며, 크로스 플랫폼 접근성(모바일/웹/Chrome), 강력한 세션/프로세스 관리, 개방형 플러그인 생태계, 자동화 인프라를 제공한다.

## Key Points

### 제품 개요

- **터미널 네이티브 에이전트**: CLI 기반으로 개발자의 기존 워크플로에 자연스럽게 통합
- **크로스 플랫폼 접근성**: 모바일 앱(iOS/Android), Chrome 확장, `/teleport`/`/remote-control`로 클라우드↔로컬 세션 이동
- **멀티레포 지원**: `--add-dir` 플래그로 모노레포가 아닌 환경에서도 여러 프로젝트 동시 작업
- **고급 세션 관리**: `/branch`로 세션 포크, `/btw`로 사이드 쿼리 삽입, Git worktrees + `/batch`로 병렬 대규모 작업

### 실행 모드 (5가지)

- **REPL**: 대화형 터미널 UI (기본)
- **헤드리스**: SDK/파이프라인용 (`-p` 플래그)
- **코디네이터**: 리더 에이전트가 여러 워커 관리
- **브리지**: 로컬 CLI와 클라우드 CCR(Claude Remote Runtime) 연결
- **어시스턴트**: 상시 대기 프로액티브 모드

### 내부 아키텍처

- **쿼리 루프**: 비동기 제너레이터 기반 토큰 단위 스트리밍 엔진
- **도구 시스템**: 45+ 내장 도구가 공통 인터페이스(`name`, `inputSchema`, `call`, `checkPermissions`, `validateInput`) 준수
- **4가지 권한 모드**: Default, Auto, Plan, Bypass — 규칙 우선순위는 로컬 → 프로젝트 → 사용자 → 플래그 → 정책
- **훅 시스템**: SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop 5개 이벤트에 커스텀 로직 주입
- **React TUI**: Yoga 레이아웃, 이중 버퍼링, 객체 풀링, 프레임 조절 최적화
- **메모리 시스템**: `~/.claude/projects/{slug}/memory/`에 user/feedback/project/reference 4타입으로 저장
- 자세한 내용: [claude-code-architecture](claude-code-architecture.md)

### 확장성 & 자동화

- **개방형 플러그인 시스템**: 슬래시 커맨드, 스킬, 훅, MCP 서버를 플러그인으로 배포. OpenAI Codex 같은 경쟁사 모델도 플러그인으로 통합 가능
- **자동화 인프라**: `/loop`, `/schedule`로 최대 1주일 단위 반복 작업 자동화
- **Auto Mode**: 트랜스크립트 분류기 기반 자율 실행 — 승인 피로 해결을 위한 2-레이어 방어 시스템 (→ [claude-code-auto-mode](claude-code-auto-mode.md))

### 실전 행동 특성 (Opus 4.6, 100h 사용 관찰)

- **속도/반응성**: 매우 빠르고 인터랙티브 — 짧은 루프에 강점
- **구조적 약점**:
  - `CLAUDE.md` 등 **지시 파일을 자주 무시**
  - 근본 원인(root cause)보다 **증상 패치(hack/patch/helper)** 선호
  - 작업을 **미완료 상태로 남기는 경향**
  - 새 파일 생성보다 **기존 파일에 함수 추가**
  - **과신(overconfidence)**: 틀린 분석도 확신 있게 제시
  - 토큰 소진 속도가 빠름
- **적합 용도**: rapid prototyping, 탐색, 초안 작성, 인터랙티브 작업
- **부적합 용도**: 장시간 fire-and-forget 자율 작업

## Sources

- [260327_anthropic-claude-code-auto-mode-design](../summaries/260327_anthropic-claude-code-auto-mode-design.md) — Auto Mode 설계
- [260331_claude_code_hidden_features](../summaries/260331_claude_code_hidden_features.md) — 15가지 숨겨진 기능
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md) — 플러그인 생태계 개방성
- [wikidocs_net_338204](../summaries/wikidocs_net_338204.md) — 소스 코드 내부 아키텍처 분석
- [260416_claude-code-vs-codex-comparison](../summaries/260416_claude-code-vs-codex-comparison.md) — Codex 대비 실전 행동 차이

## Related Concepts

- [claude-code-architecture](claude-code-architecture.md) — 내부 구조 심층 분석
- [claude-code-auto-mode](claude-code-auto-mode.md) — 자율 실행 모드 (4가지 권한 모드 중 하나)
- [claude-code-plugin](claude-code-plugin.md) — 플러그인 시스템과 배포
- [openai-codex](openai-codex.md) — 상호 보완 에이전트 (교차 검증 파트너)
- [cross-validation-workflow](cross-validation-workflow.md) — Claude 초안 + Codex 리뷰 패턴
