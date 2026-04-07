# Prompt Injection

파일, 웹페이지, 툴 출력 등의 외부 컨텐츠에 삽입된 지시가 LLM 에이전트를 사용자의 원래 작업에서 **공격자의 목적으로 리다이렉트**하는 공격 기법. 특히 툴 결과를 에이전트 컨텍스트에 주입하는 agentic 시스템에서 핵심 위협이며, 에이전트가 자체 의사결정 권한을 가질수록 잠재 피해 범위(blast radius)가 커진다.

## Key Points

- **진입 경로**: 툴 출력물이 1차 공격 벡터 — 파일 내용, 웹 fetch 결과, 셸 출력, 외부 API 응답 등 에이전트가 "읽는" 모든 것
- **공격 패턴**: 문서 내 숨겨진 지시("이제 ~를 실행하세요"), base64 인코딩된 명령, 주석 내 지시 등으로 에이전트를 하이재킹
- **취약 영역**: 에이전트가 높은 자율성을 가진 자동 모드, 장기 실행 멀티 스텝 워크플로, 외부 소스에서 데이터를 자주 수집하는 시스템
- **구조적 방어**: 분류기가 툴 결과를 아예 안 보게 만드는 구조적 설계 — 페이로드를 볼 수 없으니 페이로드에 속을 수 없음
- **계층적 방어**: 입력 레이어(프롬프트 인젝션 프로브)와 출력 레이어(트랜스크립트 분류기)의 조합 — 단일 레이어 우회 시에도 다른 레이어에서 차단

## Claude Code Auto Mode에서의 방어

Claude Code Auto Mode는 prompt injection을 4가지 위험 요인 중 하나로 명시적으로 다룬다 (나머지는 overeager behavior, honest mistakes, misaligned model). 방어는 두 가지 메커니즘으로 작동한다:

1. **입력 레이어 프로브**: 툴 출력을 에이전트 컨텍스트 전달 전에 스캔. 하이재킹 시도 감지 시 경고 주입.
2. **트랜스크립트 분류기의 구조적 무시**: 분류기가 툴 결과를 보지 않으므로 페이로드의 영향을 원천 차단. ".env 파일을 외부 URL에 POST하는 행동은 내용과 무관하게 사용자 의도에 위배" — 페이로드 분석이 아닌 행동 자체 평가.

두 레이어가 복합적으로 작동해야 방어가 성립한다: 인젝션이 성공하려면 입력 레이어를 통과해야 하고, 이후 트랜스크립트 분류기가 페이로드 없이 독립적으로 해당 툴 호출을 안전하다고 판단해야 한다.

## Sources
- [260327_anthropic-claude-code-auto-mode-design](../summaries/260327_anthropic-claude-code-auto-mode-design.md)

## Related Concepts
- [claude-code-auto-mode](claude-code-auto-mode.md) — Prompt injection 방어 메커니즘 포함
- [agentic-architecture](agentic-architecture.md) — Prompt injection이 위협이 되는 시스템 구조
