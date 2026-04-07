# Agentic Architecture

LLM이 단순히 입력-출력 함수로 동작하는 대신, **도구를 사용하고 다단계 의사결정을 수행하는 에이전트**로 구성된 시스템 설계. 단일 거대 모델 호출(One-Shot)의 한계를 넘어 **관찰-추론-행동 루프**(ReAct), 서브 에이전트 위임, 자동화된 자기 수정 등을 통해 복잡한 작업을 해결한다.

## Key Points

- **도구 사용 기반**: LLM이 브라우징, 코드 실행, 파일 읽기/쓰기, API 호출 같은 도구를 직접 호출하며 작업 수행
- **ReAct 패턴**: Reason(추론) + Act(행동) 루프 — 관찰 → 추론 → 행동 → 관찰 반복
- **서브 에이전트 전문화**: 단일 에이전트가 모든 걸 처리하는 대신 역할별로 쪼갠 전문 에이전트가 독립적으로 최적화
- **계층적 오케스트레이션**: 메인 에이전트가 서브 에이전트에 위임, 서브 에이전트가 결과를 반환하는 멀티 에이전트 핸드오프 구조
- **자기 수정 루프**: 결과를 관찰하고 실패 시 재시도하는 피드백 루프로 견고성 확보
- **보안 고려**: 도구 사용 권한, 행동의 blast radius, prompt injection 방어 등이 핵심 고민

## 전문화의 우위 (Shopify 사례)

Shopify는 초기에 하나의 에이전트가 사기 탐지와 프로필 추출을 모두 처리하려 했으나, 작업 간 간섭이 발생해 역할별 분리가 필요해졌다:

- **Fraud Agent**: 외부 리뷰 사이트를 검색해 사기 여부 판별
- **Profile Agent**: 상점 내부 페이지를 파싱해 정책/프로필 정보 추출

각 에이전트를 독립적으로 최적화할 수 있게 되자 성능 향상이 훨씬 수월해졌다. 이는 "복잡한 작업일수록 단일 에이전트보다 관심사 분리된 서브 에이전트 구조가 유리하다"는 교훈으로 이어진다.

## 멀티 에이전트 보안 (Claude Code Auto Mode 사례)

Agentic architecture의 서브 에이전트 위임은 보안상 새로운 위협 표면을 만든다. Claude Code Auto Mode는 이를 해결하기 위해 분류기를 **아웃바운드(위임 시)**와 **리턴(결과 반환 시)** 양쪽에서 실행한다. 아웃바운드 검사는 "서브에이전트 내부에서는 오케스트레이터의 지시가 사용자 메시지처럼 보여 완전히 승인된 것처럼 보이는 문제"를 방지하고, 리턴 검사는 실행 중 프롬프트 인젝션 노출 가능성에 대응한다.

## Sources
- [260331_shopify_data_structuring_dspy](../summaries/260331_shopify_data_structuring_dspy.md)
- [260327_anthropic-claude-code-auto-mode-design](../summaries/260327_anthropic-claude-code-auto-mode-design.md)

## Related Concepts
- [dspy](dspy.md) — Agentic architecture의 파이프라인 자동 최적화
- [prompt-injection](prompt-injection.md) — Agentic 시스템의 핵심 위협
- [llm-cost-optimization](llm-cost-optimization.md) — 전문화 에이전트를 통한 비용 절감
