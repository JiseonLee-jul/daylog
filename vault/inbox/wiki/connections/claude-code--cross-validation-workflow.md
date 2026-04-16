# Claude Code ↔ Cross-Validation Workflow

**연결 유형:** 구성 요소 — Claude Code는 교차 검증 워크플로우에서 **초안 작성자(drafter)** 역할에 최적화

## 관계 요약

Claude Code의 빠른 응답성과 인터랙티브 특성은 **초안 작성 단계**에 이상적이다. 반면 지시 파일 무시, 증상 패치, 미완결성 같은 약점 때문에 **최종 산출물로 바로 쓰기 어렵다** — 이는 정확히 교차 검증 워크플로우가 보완하려는 지점이다.

## 구체적 사용 패턴

- `/codex:review` / `/codex:adversarial-review` — Claude가 작성한 코드를 Codex에 검토 요청
- `/codex:rescue` — Claude가 막혔을 때 동일 컨텍스트를 Codex에 위임
- MCP 서버 래핑으로 Claude Code 세션 내부에서 Codex 호출 통합

## 시사점

- Claude Code 사용자는 **처음부터 완벽을 노리지 않고** 빠른 초안 → 다른 모델 검토 → 반영의 루프를 설계하는 것이 현실적
- "babysitting 총량"을 줄이는 방법은 Claude 자체의 품질 향상을 기다리는 것이 아니라 **검토 단계의 자동화**

## Sources

- [260416_claude-code-vs-codex-comparison](../summaries/260416_claude-code-vs-codex-comparison.md)
