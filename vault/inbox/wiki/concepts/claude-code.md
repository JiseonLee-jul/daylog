# Claude Code

Anthropic이 개발한 터미널 기반 코딩 에이전트. 단순한 코드 생성 도구를 넘어 **개발 워크플로 전체를 자동화하는 플랫폼**으로 진화 중이며, 크로스 플랫폼 접근성(모바일, 웹, Chrome 확장), 강력한 세션/프로세스 관리, 플러그인 생태계, 자동화 인프라를 제공한다.

## Key Points

- **터미널 네이티브 에이전트**: CLI 기반으로 개발자의 기존 워크플로에 자연스럽게 통합
- **확장 가능한 플러그인 시스템**: 슬래시 커맨드, 스킬, 훅, MCP 서버를 플러그인으로 배포 가능
- **고급 세션 관리**: `/branch`로 세션 포크, `/teleport`로 클라우드↔로컬 세션 이동, `/btw`로 사이드 쿼리 삽입, Git worktrees로 병렬 작업
- **자동화 인프라**: `/loop`, `/schedule`로 최대 1주일 단위 반복 작업 자동화. Hooks로 라이프사이클 시점 커스텀 로직 삽입
- **Auto Mode**: 트랜스크립트 분류기 기반의 자율 실행 모드 — 승인 피로 문제를 해결하기 위한 2-레이어 방어 시스템
- **멀티레포 지원**: `--add-dir` 플래그로 모노레포가 아닌 환경에서도 여러 프로젝트 동시 작업
- **플러그인 생태계 개방성**: OpenAI Codex 같은 경쟁사 모델도 플러그인으로 통합 가능

## Sources
- [260327_anthropic-claude-code-auto-mode-design](../summaries/260327_anthropic-claude-code-auto-mode-design.md)
- [260331_claude_code_hidden_features](../summaries/260331_claude_code_hidden_features.md)
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md)

## Related Concepts
- [claude-code-auto-mode](claude-code-auto-mode.md) — Claude Code의 자율 실행 모드
- [claude-code-plugin](claude-code-plugin.md) — 플러그인 시스템과 배포
