# Claude Code Architecture

Claude Code 내부 아키텍처의 심층 구조. TypeScript + React TUI + Zustand 기반의 약 1,884개 소스 파일로 구성된 CLI 도구로, 모든 설계는 **안전성, 성능, 확장성** 세 원칙을 따른다. 전체 실행 흐름은 4단계 구조(시작 → 대화 루프 → 도구 실행 → 결과 표시)로 요약되며, 핵심 엔진은 비동기 제너레이터 기반 쿼리 루프다.

## 전체 구조 (4단계)

| 단계 | 역할 |
|------|------|
| **시작** | 인증, 모델 선택, 설정 로드, Git 상태/CLAUDE.md 수집 |
| **대화 루프** | Claude API 스트리밍 전송, 도구 사용 감지 |
| **도구 실행** | 45+ 내장 도구, 입력 검증, 권한 확인 |
| **결과 표시** | 터미널에 변경사항 렌더링 |

시작 과정은 "병렬 I/O 사전 실행 → 조건부 모듈 로딩 → 설정 조기 로딩 → 인증 → 모델 해석 → 초기 상태 구성" 순서. 인증은 5가지 방식 우선순위로 OAuth > API 키 > AWS Bedrock > Google Vertex > Azure Foundry.

## 5가지 실행 모드

- **REPL**: 대화형 터미널 UI (기본)
- **헤드리스**: SDK/파이프라인용, UI 없음 (`-p` 플래그)
- **코디네이터**: 리더 에이전트가 여러 워커 관리
- **브리지**: 로컬 CLI와 클라우드 CCR 연결
- **어시스턴트**: 상시 대기 프로액티브 모드

## 쿼리 루프 — 핵심 엔진

비동기 제너레이터 패턴으로 **토큰 단위 스트리밍**을 지원. 각 턴마다:

1. 메시지 전처리 (토큰 압축)
2. API 스트리밍 호출
3. 에러 보류 및 복구 (Withhold & Recover 패턴)
4. 도구 실행 (안전 도구는 병렬, 위험 도구는 순차)
5. 후처리 (훅 실행, 예산 확인)

## 도구 실행 파이프라인 (10단계)

```
이름 조회 → 중단 신호 확인 → Zod 검증 → PreToolUse 훅
→ 권한 확인 → 도구 실행 → 결과 매핑 → 오버사이즈 처리
→ PostToolUse 훅 → 텔레메트리
```

**동시성 파티셔닝**: 연속된 안전 도구는 배치로 병렬 실행, 비안전 도구는 단독 실행. 모든 도구는 공통 인터페이스(`name`, `inputSchema`, `call()`, `checkPermissions()`, `validateInput()`)를 따른다.

**주요 도구:**
- **BashTool**: Tree-sitter로 AST 분석, 기본 거부 설계
- **FileEditTool**: 퍼지 매칭, Git diff 생성
- **AgentTool**: 서브에이전트 위임
- **GrepTool**: ripgrep 기반 검색

## 권한 시스템

4가지 모드:

| 모드 | 동작 |
|------|------|
| **Default** | 읽기 자동 승인, 쓰기 확인 |
| **Auto** | AI 분류기 2단계 평가 (→ [claude-code-auto-mode](claude-code-auto-mode.md)) |
| **Plan** | 읽기 전용 |
| **Bypass** | 모든 것 자동 승인 (`--dangerously-skip-permissions`) |

규칙 우선순위: **로컬 → 프로젝트 → 사용자 → 플래그 → 정책**

## 훅 시스템

이벤트별 사용자 정의 동작 주입:

- `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `Stop`

각 훅은 작업 계속, 차단, 입력 수정, 컨텍스트 주입 가능.

## UI 레이어 (TUI)

React 기반 자체 제작 프레임워크:

- **Yoga 레이아웃 엔진**
- **이중 버퍼링**, **객체 풀링**, **더티 추적**, **프레임 조절** 최적화

## 브리지 시스템

로컬 터미널과 클라우드 **Claude Remote Runtime(CCR)** 연결:

- 폴링 루프로 작업 요청 수신
- 최대 32개 병렬 세션 관리
- 메시지 중복 제거 위해 순환 버퍼 사용

## 코디네이터 모드

리더 에이전트가 여러 워커에게 작업 분배:

1. **조사** (병렬)
2. **합성** (순차 + 리더 직접 이해)
3. **구현** (영역별 순차)
4. **검증** (병렬)

## 메모리 시스템

`~/.claude/projects/{project-slug}/memory/` 구조, 4가지 타입:

- **user**: 사용자 역할, 전문성
- **feedback**: 작업 방식, 수정사항
- **project**: 목표, 마감일, 결정사항
- **reference**: 외부 시스템 포인터

## 8가지 핵심 설계 패턴

| 패턴 | 설명 |
|------|------|
| Generator Streaming | 실시간 이벤트 표시 |
| Feature Gate | 빌드 시점 데드코드 제거 |
| Memoized Context | 세션 수명 동안 캐싱 |
| Withhold & Recover | 복구 가능 에러 버퍼링 |
| Lazy Import | 순환 의존성 회피 |
| Immutable State | 예측 가능한 상태 변화 |
| Interruption Resilience | 트랜스크립트 사전 저장 |
| Dependency Injection | 테스트/모드별 교체 가능 |

## Key Points

- TypeScript + React TUI (Yoga) + Zustand 상태 관리
- 약 1,884 소스 파일, 안전성/성능/확장성 3원칙
- 쿼리 루프는 async generator 기반 토큰 스트리밍
- 10단계 도구 실행 파이프라인 + 동시성 파티셔닝
- 45+ 공통 인터페이스 준수 도구
- 4가지 권한 모드 (Default/Auto/Plan/Bypass) + 5단계 규칙 우선순위
- 5가지 훅 이벤트로 라이프사이클 확장
- 브리지 시스템으로 로컬-클라우드 연결 (최대 32 병렬)
- 코디네이터 모드의 리더-워커 구조 (조사/합성/구현/검증 단계)
- 4타입 메모리 시스템 (user/feedback/project/reference)
- 8가지 설계 패턴 (generator streaming, feature gate 등)

## Sources
- [wikidocs_net_338204](../summaries/wikidocs_net_338204.md)

## Related Concepts
- [claude-code](claude-code.md) — 상위 제품 개요
- [claude-code-auto-mode](claude-code-auto-mode.md) — 4가지 권한 모드 중 Auto의 상세
- [claude-code-plugin](claude-code-plugin.md) — 훅/스킬/플러그인 확장 메커니즘
