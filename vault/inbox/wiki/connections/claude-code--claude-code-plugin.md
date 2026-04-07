# Claude Code ↔ Claude Code Plugin

Claude Code Plugin은 Claude Code의 **확장성을 담당하는 핵심 메커니즘**으로, 슬래시 커맨드/스킬/훅/스크립트/MCP 서버를 패키징해 재사용 가능한 도구로 배포한다. Claude Code가 "터미널 에이전트"에서 "개발 워크플로 플랫폼"으로 진화한 핵심 동력이 플러그인 시스템의 개방성이다.

## From Claude Code perspective

Claude Code는 단순 코드 생성 도구가 아니라 **플랫폼**을 지향하며, 이를 위해 Plugin 시스템이 필수적이다. 15개 숨겨진 기능 중 상당수가 플러그인 형태로 제공되거나 플러그인화될 수 있는 구조다(커스텀 에이전트, Hooks, `/loop`/`/schedule` 같은 자동화). 플러그인은 Claude Code의 핵심 제품 로드맵과 독립적으로 커뮤니티가 기능을 확장할 수 있게 하는 개방성 전략의 핵심이다.

## From Claude Code Plugin perspective

Claude Code Plugin은 **Claude Code 없이는 존재 의미가 없는** 종속적 컴포넌트다. 플러그인은 Claude Code의 프롬프트 시스템, 도구 권한 모델, 세션 관리, LLM 호출 인프라를 전제로 작동한다. `allowed-tools` frontmatter, `!`백틱 컨텍스트 주입, `${CLAUDE_PLUGIN_ROOT}` 환경 변수 같은 메커니즘이 모두 Claude Code 런타임에 의존한다. 플러그인의 설치/로드/리로드 흐름(`/plugin install`, `/reload-plugins`)도 Claude Code가 제공하는 메타 커맨드다.

## 생태계 개방성의 효과

플러그인 시스템의 가장 강력한 효과는 **경쟁사 모델까지 통합 가능한 개방성**이다. OpenAI가 Apache-2.0 라이선스로 Claude Code용 Codex 플러그인을 공개한 것은 Claude Code가 모델 lock-in을 우선시하지 않는다는 신호다. 사용자 관점에서는 "내가 쓰는 에이전트 환경(Claude Code) 안에서 여러 모델(Claude, Codex, Gemini 등)을 자유롭게 호출"할 수 있게 되는데, 이는 모델 제공자와 개발 환경의 경계를 사용자 레벨에서 약화시킨다.

## Sources
- [260331_claude_code_hidden_features](../summaries/260331_claude_code_hidden_features.md)
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md)
