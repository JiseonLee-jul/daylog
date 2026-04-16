# Claude Code(~100시간) vs. Codex(~20시간) 실전 비교

**Source:** GeekNews (news.hada.io)
**URL:** https://news.hada.io/topic?id=28538
**Original:** Reddit r/ClaudeCode — "Claude Code 100 hours vs Codex 20 hours"
**Original URL:** https://www.reddit.com/r/ClaudeCode/comments/1sk7e2k/claude_code_100_hours_vs_codex_20_hours/
**Date:** 2026-04-16 (ingested)

---

## 배경

- 작성자: 메이저 빅테크에서 14년 경력의 Principal/Staff Engineer
- 대상 프로젝트: 80,000 LOC 규모의 Python/TypeScript VSCode 익스텐션
- 테스트 스위트: 2,800개
- 비교 대상: **Claude Code (Opus 4.6)** 약 100시간 사용 vs **Codex (GPT-5.4)** 약 20시간 사용
- 관찰 목적: 대규모 실 프로젝트에서 두 코딩 에이전트의 실제 행동 차이

## Claude Code의 특징

- **속도/인터랙션**: 매우 빠르고 인터랙티브. 즉시 응답으로 대화 루프를 유지하기 좋음.
- **감독 필요도**: "babysitting"이라 표현될 정도로 지속적인 감독이 필요.
- **접근 방식**: 아키텍처 개선보다 **빠른 패치(hack/patch/helper)** 성향.
  - 근본 원인(root cause) 분석보다 증상 완화를 선호.
- **지시 준수**: `CLAUDE.md` 등 instruction 파일을 자주 **무시**.
- **파일 구조**: 새 파일을 만들기보다 **기존 파일에 함수를 무분별하게 추가**.
- **작업 완결성**: 작업을 **미완료 상태로 남기는 경향**.
- **토큰 소모**: 토큰 소진 속도가 빠름.
- **자신감**: 과신(overconfidence) 경향. 틀린 분석도 확신에 차서 제시.

**적합 용도:** 빠른 프로토타이핑, 인터랙티브 탐색, 초안 작성.

## Codex의 특징

- **속도**: Claude 대비 **3~4배 느림**.
- **접근 방식**: **신중하고 체계적(deliberate/systematic)**.
- **리팩토링**: 명시적 지시 없이도 **자발적으로 리팩토링** 수행하여 코드 품질 향상.
- **지시 준수**: `AGENTS.md` 등 지시 파일을 **철저히 준수**.
- **자율성**: 신뢰가 확보된 뒤에는 **"fire-and-forget"** 방식으로 맡겨두어도 완결.
- **약점**:
  - 로봇 같은 커뮤니케이션 스타일.
  - 경험 많은 개발자와도 과도하게 논쟁하려는 경향.
  - 대형 기능 구현에서 간간이 빠진 부분이 생김.

**적합 용도:** 엔터프라이즈급 소프트웨어 개발, 장시간 자율 작업.

## 공통 결론

- 두 도구 모두 **단단한 소프트웨어 엔지니어링 역량**이 있어야 좋은 결과가 나옴.
- "LLM이 알아서 잘 해줄 것"이라는 기대는 어느 쪽에서도 성립하지 않음.
- 선택은 워크플로우에 따라 달라짐:
  - **Claude** → rapid prototyping
  - **Codex** → enterprise-grade development

## 커뮤니티 논의 하이라이트

### 교차 검증(cross-validation) 워크플로우가 가장 인기

- 패턴: **Claude로 초안 → Codex로 리뷰** (혹은 반대 방향).
- 근거 인용: *"두 모델이 같은 방식으로 할루시네이션하는 경우는 극히 드물다."*
- 두 모델의 강점이 상호 보완적.

### 그 외 떠오르는 패턴

- **Baton-pass workflow**: state file에 진행 상황을 저장해 두 에이전트 사이에서 인계.
- **Codex를 MCP 서버로 래핑**하여 Claude Code 내부에서 호출.
- **3-모델 리뷰 사이클**: Claude → Codex → Gemini 순으로 교차 검토.

### 반복 지적되는 비판

**Claude Code 관련:**
- 명시적 지시 무시가 구조적으로 광범위하게 발생.
- 루트 원인 수정 대신 증상 패치로 일관.
- 토큰 소진 속도.

**Codex 관련:**
- 소통 스타일이 딱딱함.
- 경험자 상대로도 논쟁적.
- 대규모 기능 구현 시 누락 지점 존재.

## 시사점(개인 해석)

- "하나의 에이전트"가 아닌 **에이전트 포트폴리오** 관점이 필요.
  - 역할 분담: 초안/탐색(Claude) ↔ 검증/정리(Codex).
- `CLAUDE.md` / `AGENTS.md` 같은 **지시 파일의 효과는 도구별로 비대칭**.
  - Codex 쪽에서 투자 대비 회수가 더 크다는 실증 사례.
- **babysitting 총량을 줄이는 것**이 생산성 레버. 느려도 완결성이 높은 쪽이 장시간 누적 총량에서 우위.
- 조직 차원의 에이전트 워크플로우는 **교차 검증 + 상태 인계** 두 축이 표준으로 굳어지는 분위기.
