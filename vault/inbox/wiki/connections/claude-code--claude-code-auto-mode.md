# Claude Code ↔ Claude Code Auto Mode

Claude Code Auto Mode는 Claude Code의 자율 실행 모드로, **사용자 승인 피로 문제를 해결하기 위한 플랫폼 내장 기능**이다. Claude Code의 핵심 설계 철학(개발자 워크플로 자동화)과 Auto Mode의 안전 장치(트랜스크립트 분류기)가 결합되어, "편의성과 안전성의 양립"을 목표로 하는 관계.

## From Claude Code perspective

Claude Code는 개발 워크플로 자동화 플랫폼을 지향하지만, 모든 행동에 사용자 승인을 요구하면 자동화의 의미가 사라진다. Auto Mode는 이 딜레마의 해답으로, **승인 피로를 야기하는 93%의 관성적 승인은 분류기가 자동화**하고 **실제 위험한 7%만 사람에게 에스컬레이션**한다. Claude Code의 Hooks, `/loop`, `/schedule` 같은 자동화 기능이 Auto Mode 없이는 여전히 승인 프롬프트에 막혔을 것이다.

## From Claude Code Auto Mode perspective

Auto Mode는 독립적인 제품이 아니라 **Claude Code 플랫폼의 필수 신뢰 계층**이다. Claude Code의 Tier 1 기본 허용 툴, Tier 2 프로젝트 파일 작업 같은 권한 구조를 전제로 작동하며, Tier 3(실질적 피해 가능성이 있는 행동)에서만 분류기를 호출한다. 또한 Auto Mode는 Claude Code의 헤드리스 모드(`-p` 플래그), 서브에이전트 시스템, 플러그인 생태계 모두에 자연스럽게 통합되어야 한다 — 멀티 에이전트 핸드오프 검사가 대표 사례.

## Design Trade-off

Auto Mode의 FNR 17%라는 수치는 Claude Code 플랫폼이 감수하는 트레이드오프를 드러낸다: **완벽한 보안 vs 실용적 자동화**. Anthropic은 "충분히 위험한 행동을 차단해 가드레일 없는 상태 대비 자율 작동을 실질적으로 더 안전하게" 만드는 것이 목표이며, 고위험 인프라의 신중한 수동 검토의 드롭인 대체재는 아니라고 명시한다. 즉 Claude Code는 Auto Mode를 **`--dangerously-skip-permissions`의 안전한 대체재**로 포지셔닝하고, 그 이상의 영역에서는 여전히 사람의 판단을 요구한다.

## Sources
- [260327_anthropic-claude-code-auto-mode-design](../summaries/260327_anthropic-claude-code-auto-mode-design.md)
- [260331_claude_code_hidden_features](../summaries/260331_claude_code_hidden_features.md)
