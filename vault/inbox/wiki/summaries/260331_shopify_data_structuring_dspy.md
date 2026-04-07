---
source: raw/260331_shopify_data_structuring_dspy.md
compiled: 2026-04-07
topics: [dspy, agentic-architecture, shopify, sub-agent-specialization, llm-cost, ReAct]
---

# Summary: Shopify의 데이터 구조화 여정 — One-Shot LLM에서 DSPy 기반 에이전틱 아키텍처로

Shopify는 수백만 개의 상점에서 "이 상점이 휴대폰을 파는가?" 같은 단순한 질문조차도 표준화된 답변으로 뽑아내기 어려운 문제를 해결하기 위해 3단계로 솔루션을 진화시켰다. 1단계는 상점 페이지를 통째로 GPT-4/5에 넘기는 One-Shot LLM 방식이었지만 컨텍스트 윈도우 제약, 프롬프트 민감도, 비용 기하급수적 증가 문제에 부딪혔다. 2단계에서는 ReAct 패턴의 에이전트가 직접 브라우징하고 조사할 수 있게 했고, DSPy 옵티마이저가 프롬프트를 프로그래밍 방식으로 자동 최적화하도록 전환했다.

3단계의 핵심은 **전문화된 서브 에이전트로의 분리**였다. 하나의 에이전트가 모든 걸 처리하던 구조에서 Fraud Agent(외부 리뷰 사이트 기반 사기 판별)와 Profile Agent(상점 내부 페이지 파싱)로 역할별로 쪼갰고, 각 에이전트를 독립적으로 최적화할 수 있게 되어 성능 향상이 훨씬 수월해졌다. 평가 재현성을 위해 라벨링 시점의 상점 상태를 통째로 고정하는 **ShopNap 스냅샷 서비스**를 별도로 구축했다 — 실시간 크롤링은 웹사이트가 시시각각 바뀌어 라벨과 데이터가 어긋나기 때문.

인프라는 3계층으로 구성된다. Flink 기반 Batch Layer가 일일 15만 개 이상 상점을 처리하고, Kubernetes 기반 Agent Layer가 CPU 클러스터에서 에이전트 로직을 수행하며, GPU Cluster의 vLLM이 자체 호스팅 Qwen 모델로 추론을 담당한다. 비싼 LLM 추론(GPU)과 CPU로 충분한 에이전트 로직을 물리적으로 분리한 게 비용 효율의 열쇠다.

결과적으로 Agentic + DSPy + Qwen 조합은 One-Shot GPT-5 대비 **비용은 1/75 수준, 품질은 약 2배**, 커버리지는 전체 상점으로 확대됐다. 세 가지 교훈: (1) 복잡한 작업일수록 단일 에이전트보다 관심사 분리된 서브 에이전트 구조가 유리, (2) 프롬프트 미세 조정보다 시스템 아키텍처 설계와 최적화 자동화가 지속 가능, (3) 도메인 최적화된 중소형 모델이 범용 대형 모델보다 가성비와 성능 모두 우위.

## Key Points
- 3단계 진화: One-Shot → Agentic + DSPy → Specialized Sub-Agents
- DSPy가 프롬프트를 프로그래밍 방식으로 자동 최적화 (수동 튜닝 대체)
- 관심사 분리된 서브 에이전트 (Fraud, Profile)로 독립 최적화 가능
- ShopNap: 실시간 크롤링 대신 스냅샷 고정으로 평가 재현성 확보
- 3-layer infra: Flink(batch) + K8s(agent) + GPU cluster(LLM), 비용 분리 최적화
- 결과: 비용 1/75, 품질 2배, 전체 상점 커버

## Related Concepts
- [dspy](../concepts/dspy.md)
- [agentic-architecture](../concepts/agentic-architecture.md)
- [llm-cost-optimization](../concepts/llm-cost-optimization.md)
