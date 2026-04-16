# Cross-Validation Workflow

서로 다른 LLM 코딩 에이전트(예: Claude, Codex, Gemini) 두 대 이상을 **생성자와 검토자 역할로 짝지어** 교차 검증하는 워크플로우 패턴. 단일 모델의 과신, 할루시네이션, 편향을 상호 보완하려는 집단 지성(collective intelligence) 전략이며, 2026년 시점에서 실무자들의 **가장 인기 있는 다중 에이전트 사용 패턴**으로 자리잡았다.

## Key Points

### 핵심 원리

- **모델 다양성**: *"두 모델이 같은 방식으로 할루시네이션하는 경우는 극히 드물다"*는 관찰에 기반
- **역할 분리**: 한 모델은 생성(draft)에, 다른 모델은 비판/검토에 특화
- **편향 상쇄**: 같은 모델 내부 반복으로는 벗어나기 어려운 사고 루프를 다른 모델이 끊어줌

### 대표 패턴

1. **Claude draft → Codex review** (또는 역방향)
   - Claude Code로 빠르게 초안 작성 → Codex가 체계적으로 검토/리팩토링
   - 서로의 약점을 보완: Claude의 미완결성 ↔ Codex의 속도
2. **Baton-pass workflow**
   - state 파일에 진행 상황/컨텍스트를 저장하고 에이전트 간 인계
   - 한 모델의 세션 한계/토큰 한계를 넘어 작업 연속성 확보
3. **MCP 래핑**
   - Codex를 MCP 서버로 노출해 Claude Code 세션 안에서 직접 호출
   - 단일 IDE 경험 안에 교차 검증이 녹아들도록 구성
4. **3-모델 리뷰 사이클**
   - Claude → Codex → Gemini 순차 검토
   - 독립적 편향이 셋 다 겹칠 확률이 낮아지는 것을 활용

### 왜 필요한가

- **과신(overconfidence) 문제**: LLM은 틀린 답도 자신 있게 제시 — 자체 검증으로 잡기 어려움
- **지시 준수 편차**: 같은 지시 파일(`CLAUDE.md`/`AGENTS.md`)을 어떤 모델은 따르고 어떤 모델은 무시 → 크로스 체크로 드러남
- **근본 원인 vs 증상 패치**: 한 모델의 "빠른 패치 성향"을 다른 모델의 "리팩토링 성향"으로 보정

### 한계 / 주의점

- **비용 증가**: 동일 작업을 두 번 이상 처리 — 토큰 비용 2~3배
- **속도 저하**: 순차 파이프라인일 경우 가장 느린 모델(Codex)이 병목
- **의견 충돌 처리**: 두 모델의 판단이 갈릴 때 최종 결정자는 결국 인간 엔지니어
- **공통 약점은 잡히지 않음**: 모든 LLM이 동일하게 틀리는 영역(예: 최신 API 변경)은 교차 검증으로도 해결 불가 → 외부 검증(테스트, 문서 조회)이 여전히 필요

### 관련 도구/메커니즘

- `/codex:review` — Codex 기반 읽기 전용 코드 리뷰
- `/codex:adversarial-review` — 설계 정당성을 공격적으로 검증
- `/codex:rescue` — 막혔을 때 다른 모델로 관점 전환
- MCP 서버 래핑을 통한 통합 실행

## Sources

- [260416_claude-code-vs-codex-comparison](../summaries/260416_claude-code-vs-codex-comparison.md) — 100h vs 20h 실전 비교에서 가장 인기 있는 워크플로우로 확인
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md) — Codex adversarial-review / rescue 패턴의 원형

## Related Concepts

- [claude-code](claude-code.md) — 초안/탐색에 강점
- [openai-codex](openai-codex.md) — 리뷰/리팩토링에 강점
- [claude-code-plugin](claude-code-plugin.md) — Codex를 Claude 내부에서 호출하는 통합 경로
