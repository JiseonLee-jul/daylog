---
source: raw/260327_anthropic-claude-code-auto-mode-design.md
compiled: 2026-04-07
topics: [claude-code, auto-mode, transcript-classifier, prompt-injection, permission-system, multi-agent]
---

# Summary: Anthropic이 Claude Code Auto Mode를 설계한 방법

Anthropic은 Claude Code 사용자의 93%가 권한 프롬프트를 관성적으로 승인하는 "승인 피로(approval fatigue)" 현상을 해결하기 위해 Auto Mode를 도입했다. 핵심 설계는 **입력 레이어의 프롬프트 인젝션 프로브**와 **출력 레이어의 트랜스크립트 분류기(Sonnet 4.6 기반)** 두 겹의 방어 구조다. 이 접근은 기존의 두 극단 — 안전하지만 유지비용이 높은 샌드박스와, 보호가 전혀 없는 `--dangerously-skip-permissions` 플래그 — 사이의 중간 지점을 목표로 한다.

트랜스크립트 분류기의 가장 중요한 설계 결정은 **에이전트의 어시스턴트 메시지와 툴 결과를 제거**하고 사용자 메시지와 툴 호출 명령만으로 판단한다는 것이다. 이는 에이전트 자체의 합리화 텍스트("이전에 암묵적으로 승인됨" 같은 프레이밍)에 영향받지 않기 위한 구조적 방어다. 분류기는 또한 2단계로 작동한다: Stage 1의 빠른 단일 토큰 필터(8.5% FPR, 차단 방향 편향)와 Stage 2의 체인 오브 생각 추론(FPR 0.4%로 감소)이다.

권한 결정은 3단계 티어로 구조화된다. Tier 1은 파일 읽기 같은 상태 수정 불가 툴로 분류기 호출 없이 허용, Tier 2는 프로젝트 내 파일 작업(버전 관리로 검토 가능), Tier 3은 실질적 피해 가능성이 있는 행동(셸 명령, 웹 fetch, 외부 툴 통합)만 분류기로 평가한다. 차단 시에는 세션을 중단하지 않고 더 안전한 대안을 시도하도록 유도하며, 연속 3회 또는 총 20회 차단 시 사람에게 에스컬레이션한다.

평가 결과는 솔직하다: 실제 과도 행동 데이터셋(n=52) 기준 FNR 17%로, `--dangerously-skip-permissions` 대비 실질적 개선이지만 신중한 수동 검토의 완전한 대체재는 아니다. 가장 흔한 오류 패턴은 "PR 정리해줘" 같은 암묵적 동의 신호가 강제 푸시 같은 행동의 blast radius를 실제로 커버하는지 검증하지 않는 경우다. Anthropic은 이를 "auto mode는 --dangerously-skip-permissions를 쓰던 사람을 위한 것이며, 고위험 인프라의 드롭인 대체재가 아니다"라고 명시한다.

## Key Points
- 2-layer defense: 입력 레이어 prompt injection probe + 출력 레이어 transcript classifier
- 분류기는 어시스턴트 메시지와 툴 결과를 구조적으로 제거 — 에이전트 프레이밍에 오염 방지
- 2-stage classification: 빠른 필터(Stage 1) + 신중한 추론(Stage 2), FPR 8.5% → 0.4% 개선
- 3-tier permission system: 허용 툴, 프로젝트 파일 작업, 잠재 피해 행동
- Deny-and-continue: 차단 시 세션 중단 없이 안전한 대안 유도, 3회/20회 한계 후 에스컬레이션
- FNR 17%는 인프라 교체가 아닌 `--dangerously-skip-permissions` 대체재 수준

## Related Concepts
- [claude-code](../concepts/claude-code.md)
- [claude-code-auto-mode](../concepts/claude-code-auto-mode.md)
- [prompt-injection](../concepts/prompt-injection.md)
- [agentic-architecture](../concepts/agentic-architecture.md)
