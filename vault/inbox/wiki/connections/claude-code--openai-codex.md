# Claude Code ↔ OpenAI Codex

**연결 유형:** 상호 보완 에이전트 (대비적 장단점)

두 제품은 경쟁사의 동급 코딩 에이전트이면서, 장시간 실사용에서 드러나는 **행동 특성이 정반대에 가깝다**. 그 대비가 교차 검증 워크플로우의 근거가 된다.

## 대비 축

| 축 | Claude Code (Opus 4.6) | Codex (GPT-5.4) |
|---|---|---|
| 속도 | 매우 빠름, 인터랙티브 | 3~4배 느림, 신중 |
| 지시 파일 준수 | `CLAUDE.md` 자주 무시 | `AGENTS.md` 철저 준수 |
| 코드 성향 | 증상 패치, 헬퍼 함수 남발 | 자발적 리팩토링, 구조 개선 |
| 작업 완결성 | 미완료로 끝나는 경우 많음 | fire-and-forget 가능 |
| 파일 구조 | 기존 파일에 함수 누적 | 적절한 분리 |
| 커뮤니케이션 | 과신, 자신감 | 로봇적, 논쟁적 |
| 적합 용도 | rapid prototyping | 엔터프라이즈급 개발 |

## 결합 방식

- **교차 검증 워크플로우**(→ [cross-validation-workflow](../concepts/cross-validation-workflow.md)): 한쪽이 초안, 다른 쪽이 검토
- **플러그인 통합**(→ [claude-code-plugin ↔ openai-codex](claude-code-plugin--openai-codex.md)): Codex를 Claude Code 내부에서 호출 (`/codex:*`)
- **Baton-pass**: state 파일 기반 컨텍스트 인계로 한 에이전트의 한계를 다른 쪽이 이어받음

## 시사점

- 선택은 "둘 중 하나"가 아니라 **역할 분담** 문제
- **지시 파일 ROI 비대칭**: Codex 쪽에서 AGENTS.md 투자 효과가 훨씬 큼 → Claude Code의 지시 준수 메커니즘은 구조적 개선이 필요
- "더 좋은 단일 에이전트" 경쟁이 아니라 **에이전트 포트폴리오**의 시대로 이동

## Sources

- [260416_claude-code-vs-codex-comparison](../summaries/260416_claude-code-vs-codex-comparison.md)
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md)
