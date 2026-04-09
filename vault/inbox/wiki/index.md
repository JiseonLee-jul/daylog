# Knowledge Base Index

*Last updated: 2026-04-09*
*Sources: 7 | Concepts: 20 | Connections: 10*

## Sources

- [260327_anthropic-claude-code-auto-mode-design](summaries/260327_anthropic-claude-code-auto-mode-design.md) — Anthropic이 Claude Code Auto Mode를 설계한 방법: 2-레이어 방어, 트랜스크립트 분류기, 3-tier permission system
- [260331_shopify_data_structuring_dspy](summaries/260331_shopify_data_structuring_dspy.md) — Shopify의 데이터 구조화 여정: One-Shot LLM에서 DSPy 기반 전문화 서브 에이전트로의 3단계 진화, 비용 1/75 달성
- [260331_claude_code_hidden_features](summaries/260331_claude_code_hidden_features.md) — Claude Code 15가지 숨겨진 기능: 모바일/Chrome/Teleport, /loop, /schedule, Hooks, Git Worktrees, /batch 등
- [260331_openai_codex_plugin_for_claude_code](summaries/260331_openai_codex_plugin_for_claude_code.md) — OpenAI가 Apache-2.0으로 공개한 Claude Code용 Codex 플러그인: 경쟁 제품 생태계 통합 사례
- [260403_snapshot-scaffolding-methodology](summaries/260403_snapshot-scaffolding-methodology.md) — Clean room 포팅 방법론: 3종 JSON 스냅샷 + 빈 패키지 자동 생성 + Parity Audit으로 진행률 자동 측정
- [260409_graph-pollution-heterogeneous-corpus](summaries/260409_graph-pollution-heterogeneous-corpus.md) — Graph Pollution 문제 정의와 해결 전략: 4개 축(Chunking/Graph 계층/Saliency/Routing)으로 이기종 코퍼스 문제 해법 정리
- [wikidocs_net_338204](summaries/wikidocs_net_338204.md) — Claude Code 소스 코드 분석서: TypeScript + React TUI + Zustand, 4단계 구조, 5가지 실행 모드, 쿼리 루프, 10단계 도구 파이프라인, 8가지 설계 패턴

## Concepts

- [agentic-architecture](concepts/agentic-architecture.md) — 도구 사용과 다단계 의사결정을 수행하는 LLM 에이전트 기반 시스템 설계
- [chunking-strategy](concepts/chunking-strategy.md) — RAG 청킹 전략 4개 실행 방식(룰/구조/의미/LLM)
- [claude-code](concepts/claude-code.md) — Anthropic의 터미널 기반 코딩 에이전트 플랫폼 (제품 개요)
- [claude-code-architecture](concepts/claude-code-architecture.md) — Claude Code 내부 아키텍처 심층 분석
- [claude-code-auto-mode](concepts/claude-code-auto-mode.md) — 2-레이어 방어 기반 Claude Code 자율 실행 모드
- [claude-code-plugin](concepts/claude-code-plugin.md) — Claude Code 확장 메커니즘
- [dspy](concepts/dspy.md) — LLM 파이프라인 선언형 프레임워크 + 프롬프트 자동 최적화
- [graph-pollution](concepts/graph-pollution.md) — 지식 그래프에 저품질 노드·엣지가 축적되어 검색·추론 품질을 저하시키는 현상
- [graphrag](concepts/graphrag.md) — LLM 기반 엔티티·관계 추출로 지식 그래프를 구축하는 RAG 패러다임
- [heterogeneous-corpus](concepts/heterogeneous-corpus.md) — 성격이 다른 문서들이 한 KB에 공존하는 상황, 정보 밀도 편차의 원인
- [hybrid-rag](concepts/hybrid-rag.md) — Retrieval/Source/Index Structure 세 층위의 하이브리드 RAG 분류
- [ket-rag](concepts/ket-rag.md) — 풀 KG + 이분 그래프 이중 인덱스, PageRank 기반 중요도 선별
- [lightrag](concepts/lightrag.md) — 단일 그래프 + 쿼리 시점 dual-level retrieval
- [llm-cost-optimization](concepts/llm-cost-optimization.md) — 소형 특화 모델, 인프라 계층 분리, 파이프라인 자동화 결합 비용 절감
- [openai-codex](concepts/openai-codex.md) — Claude Code용 OpenAI Codex 플러그인
- [parity-audit](concepts/parity-audit.md) — 포팅 프로젝트 진행률 자동 수치화
- [prompt-injection](concepts/prompt-injection.md) — 외부 컨텐츠로 LLM 에이전트를 하이재킹하는 공격
- [rag-routing](concepts/rag-routing.md) — 문서 타입별 인덱스 분리 및 쿼리 시 라우팅 아키텍처
- [raptor](concepts/raptor.md) — 재귀적 요약 트리 RAG, Collapsed Tree 검색, 계층은 생성 장치
- [saliency-denoising](concepts/saliency-denoising.md) — 인덱싱 이전 단계의 LLMLingua/요약/분류 기반 노이즈 제거
- [snapshot-scaffolding](concepts/snapshot-scaffolding.md) — 3종 JSON 스냅샷 기반 clean room 포팅 방법론

## Connections

- [agentic-architecture ↔ dspy](connections/agentic-architecture--dspy.md) — 에이전트 구조 선언과 프롬프트 자동 최적화의 보완 관계
- [claude-code ↔ claude-code-architecture](connections/claude-code--claude-code-architecture.md) — 외부 기능과 내부 동작 원리의 양면 관계
- [claude-code ↔ claude-code-auto-mode](connections/claude-code--claude-code-auto-mode.md) — 개발 자동화 플랫폼과 자율 실행 안전 계층의 결합
- [claude-code ↔ claude-code-plugin](connections/claude-code--claude-code-plugin.md) — 코어 플랫폼과 확장 메커니즘의 종속/개방 관계
- [claude-code-plugin ↔ openai-codex](connections/claude-code-plugin--openai-codex.md) — 경쟁 제품 생태계에 자사 모델 통합 사례
- [graph-pollution ↔ graphrag](connections/graph-pollution--graphrag.md) — GraphRAG가 이기종 코퍼스에서 필연적으로 부딪히는 오염 문제
- [graph-pollution ↔ heterogeneous-corpus](connections/graph-pollution--heterogeneous-corpus.md) — 원인과 증상의 인과 관계
- [hybrid-rag ↔ ket-rag](connections/hybrid-rag--ket-rag.md) — Index Structure Hybrid의 가장 명확한 구현 사례
- [ket-rag ↔ raptor](connections/ket-rag--raptor.md) — 서로 다른 철학의 계층화 (중요도 분리 vs 의미 재귀)
- [parity-audit ↔ snapshot-scaffolding](connections/parity-audit--snapshot-scaffolding.md) — 구조 생성과 진행 검증의 보완 관계

## Recent Additions

- 2026-04-09: Compiled 260409_graph-pollution-heterogeneous-corpus (new concepts: graph-pollution, heterogeneous-corpus, graphrag, raptor, ket-rag, lightrag, hybrid-rag, chunking-strategy, saliency-denoising, rag-routing)
- 2026-04-07: Compiled wikidocs_net_338204
- 2026-04-07: Compiled 260327_anthropic-claude-code-auto-mode-design
- 2026-04-07: Compiled 260331_shopify_data_structuring_dspy
- 2026-04-07: Compiled 260331_claude_code_hidden_features
- 2026-04-07: Compiled 260331_openai_codex_plugin_for_claude_code
- 2026-04-07: Compiled 260403_snapshot-scaffolding-methodology
