# LLM Wiki

## Definition

LLM이 원시 소스 문서로부터 **지속적인 위키를 점진적으로 구축하고 유지**하는 개인 지식 관리 패턴. Andrej Karpathy가 제안. 기존 RAG가 매 질의마다 지식을 재도출하는 반면, LLM 위키는 지식을 한 번 컴파일하고 최신 상태로 유지한다. 위키는 "복리로 쌓이는 산출물(compounding artifact)"로, 소스 추가와 질문마다 풍부해진다.

## Architecture (3 Layers)

1. **Raw Sources** — 불변의 소스 문서 컬렉션. 진실의 원천. LLM은 읽기만 함
2. **Wiki** — LLM이 전적으로 소유하는 마크다운 파일 디렉토리. 요약, 개체, 개념, 비교, 종합 페이지
3. **Schema** — 위키 구조/규칙/워크플로를 정의하는 설정 문서 (CLAUDE.md, AGENTS.md 등)

## Core Operations

- **Ingest**: 새 소스 → LLM이 읽고 → 요약 작성 → 인덱스 업데이트 → 관련 페이지 10-15개 갱신 → 로그 기록
- **Query**: 인덱스에서 관련 페이지 탐색 → 답변 종합 → 좋은 답변은 새 위키 페이지로 재정리
- **Lint**: 주기적 건강 검진 — 모순, 오래된 주장, 고아 페이지, 누락 교차 참조, 데이터 공백 탐지

## Navigation Infrastructure

- **index.md** (콘텐츠 지향): 모든 페이지 카탈로그. ~100소스 규모에서 임베딩 RAG 인프라 대체
- **log.md** (시간순): 추가 전용 기록. 파싱 가능한 접두사 포맷 (`## [날짜] ingest | 제목`)

## Key Principle

> 인간 = 소스 큐레이션 + 분석 지시 + 질문
> LLM = 요약 + 교차 참조 + 정리 + 기록 관리 (모든 고된 작업)

위키 유지 관리 비용이 거의 제로이므로, 인간이 위키를 포기하는 근본 원인(유지 부담 > 가치)을 제거한다.

## Practical Workflow

Obsidian을 IDE로, LLM을 프로그래머로, 위키를 코드베이스로 비유. 한쪽에 LLM 에이전트, 다른 쪽에 Obsidian을 열어두고 실시간 협업.

## Use Cases

- 개인 자기 계발/건강/심리 추적
- 연구 논문 종합 (수주~수개월)
- 독서 동반자 위키 (팬 위키 수준)
- 비즈니스 내부 위키 (Slack/회의록/고객 통화)
- 경쟁 분석, 실사, 여행 계획, 수업 노트

## Sources

- [gist_github_com_karpathy-llm-wiki](../summaries/gist_github_com_karpathy-llm-wiki.md)

## Related Concepts

- [personal-knowledge-base](personal-knowledge-base.md) — LLM 위키가 구축하는 대상
- [knowledge-compilation](knowledge-compilation.md) — LLM 위키의 핵심 메커니즘
- [memex](memex.md) — 역사적 영감
- [graphrag](graphrag.md) — 대비되는 RAG 접근
- [agentic-architecture](agentic-architecture.md) — LLM이 도구를 사용해 위키를 유지하는 에이전트 패턴
