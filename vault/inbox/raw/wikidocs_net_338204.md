# 별첨 91. 클로드 코드 소스 코드 분석서

> 출처: https://wikidocs.net/338204
> 분석 대상: Claude Code (2026-03-31)
> 분석 일자: 2026-04-01
> 총 소스 파일: 약 1,884개 (TypeScript + React)

## 요약

Claude Code는 사용자의 자연어 요청을 AI가 판단하여 적절한 도구를 실행하고, 권한 확인을 거친 후 결과를 AI에게 피드백하는 루프가 핵심이다. 인증, 설정, 상태 관리, MCP, 플러그인 등의 시스템이 이를 감싸며, 모든 설계는 안전성, 성능, 확장성 세 원칙을 따른다.

## 1. Claude Code란 무엇인가

TypeScript로 작성된 공식 CLI 도구로, 터미널에서 사용자가 자연어로 요청하면 AI가 파일을 편집하고 명령을 실행한다. React 기반의 TUI(Terminal User Interface)를 사용하며 상태 관리에 Zustand를 활용한다.

## 2. 전체 구조

네 가지 단계로 구성:

- **시작**: 인증, 모델 선택, 설정 로드, Git 상태/CLAUDE.md 수집
- **대화 루프**: 메시지를 Claude API로 스트리밍 전송, 도구 사용 감지
- **도구 실행**: 45개 이상의 내장 도구, 입력 검증 및 권한 확인
- **결과 표시**: 터미널에 변경 사항 렌더링

## 3. 실행 모드

- **REPL 모드**: 대화형 터미널 UI
- **헤드리스 모드**: SDK/파이프라인용 UI 없음
- **코디네이터 모드**: 리더 에이전트가 여러 워커 관리
- **브리지 모드**: 로컬 CLI와 클라우드 연결
- **어시스턴트 모드**: 상시 대기 프로액티브 모드

## 4. 시작 과정

```
병렬 I/O 사전 실행 → 조건부 모듈 로딩 → 설정 조기 로딩
→ 인증 → 모델 해석 → 초기 상태 구성
```

인증은 5가지 방식 우선순위: OAuth > API 키 > AWS Bedrock > Google Vertex > Azure Foundry

## 5. 쿼리 루프 — 핵심 엔진

비동기 제너레이터 패턴으로 "토큰 단위 스트리밍"을 지원한다. 각 턴마다:

1. 메시지 전처리 (토큰 압축)
2. API 스트리밍 호출
3. 에러 보류 및 복구
4. 도구 실행 (안전한 도구는 병렬, 위험한 도구는 순차)
5. 후처리 (훅 실행, 예산 확인)

## 6. 도구(Tool) 시스템

모든 도구는 공통 인터페이스를 따름:

- `name`, `inputSchema`, `call()`, `checkPermissions()`, `validateInput()`

**주요 도구**:
- **BashTool**: Tree-sitter로 AST 분석, 기본 거부 설계
- **FileEditTool**: 퍼지 매칭, Git diff 생성
- **AgentTool**: 서브에이전트 위임
- **GrepTool**: ripgrep 기반 검색

## 7. 도구 실행 파이프라인

```
이름 조회 → 중단 신호 확인 → Zod 검증 → PreToolUse 훅
→ 권한 확인 → 도구 실행 → 결과 매핑 → 오버사이즈 처리
→ PostToolUse 훅 → 텔레메트리
```

동시성 파티셔닝: 연속된 안전 도구는 배치로 병렬 실행, 비안전 도구는 단독 실행.

## 8. 권한(Permission) 시스템

네 가지 모드:

- **Default**: 읽기 자동 승인, 쓰기 확인
- **Auto**: AI 분류기 2단계 평가
- **Plan**: 읽기 전용
- **Bypass**: 모든 것 자동 승인

규칙 우선순위: 로컬 → 프로젝트 → 사용자 → 플래그 → 정책

## 9. 훅(Hook) 시스템

이벤트별 사용자 정의 동작:

- `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `Stop`

각 훅은 작업 계속, 차단, 입력 수정, 컨텍스트 주입 가능.

## 10. 스킬(Skill)과 플러그인(Plugin)

- **스킬**: 마크다운 파일 기반 재사용 가능 작업 템플릿
- **플러그인**: 스킬 + 훅 + MCP 서버의 패키지

## 11. UI 레이어 (TUI)

React 기반 자체 제작 프레임워크:

- Yoga 레이아웃 엔진
- 이중 버퍼링, 객체 풀링, 더티 추적, 프레임 조절 최적화

## 12. 브리지(Bridge) 시스템

로컬 터미널과 클라우드 Claude Remote Runtime(CCR) 연결:

- 폴링 루프로 작업 요청 수신
- 최대 32개 병렬 세션 관리
- 메시지 중복 제거 위해 순환 버퍼 사용

## 13. 코디네이터(Coordinator) 모드

리더 에이전트가 여러 워커에게 작업 분배:

1. 조사 (병렬)
2. 합성 (순차적 + 리더가 직접 이해)
3. 구현 (영역별 순차)
4. 검증 (병렬)

## 14. 메모리(Memory) 시스템

`~/.claude/projects/{project-slug}/memory/` 구조:

- **user**: 사용자 역할, 전문성
- **feedback**: 작업 방식, 수정사항
- **project**: 목표, 마감일, 결정사항
- **reference**: 외부 시스템 포인터

## 15. 핵심 설계 패턴

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

## 16. 데이터 흐름 요약

사용자 입력 → main.tsx 초기화 → 정규화 → query() 생성기 루프 → 사전 처리 → API 스트리밍 → 에러 처리 → 도구 실행 (권한 체크) → 결과 수집 → 반복 또는 종료 → 디스크 저장 → 비용 추적
