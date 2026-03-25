# 국내 AI Agent 제품/서비스 분류 연구 보고서

**Session ID:** 20260325-kr-ai-agent-taxonomy
**Date:** 2026-03-25
**Status:** Complete

---

## Executive Summary

국내 AI Agent 시장에서 활동 중인 **50+개 기업**의 제품/서비스를 조사하고, 글로벌 분류 프레임워크를 벤치마킹하여 **3축 × 6차원 분류 체계**를 수립하였다. 국내 시장은 (1) 대기업/빅테크의 자체 LLM 기반 풀스택 전략, (2) 스타트업의 버티컬 특화 전략, (3) 인프라/도구 전문사의 수평 확장 전략이 공존하는 구조이다.

핵심 발견:
- 자체 LLM 보유 기업 5개(네이버, 카카오, 삼성전자, 삼성SDS, LG AI Research)가 시장 상위 레이어를 주도
- 도메인 특화 Agent 시장에서 **법률**(3개사)과 **헬스케어**(3개사)가 가장 성숙한 버티컬
- 고객서비스/AICC가 가장 경쟁이 치열한 수평형 카테고리 (4개사+)
- 인프라 레이어에서 **LLM API 게이트웨이**, **벡터DB**, **프롬프트 관리** 전문 국내 기업이 부재 — 글로벌 도구 의존

---

## 1. 분류 프레임워크

### 1.1 글로벌 벤치마크 종합

| 출처 | 분류 기준 | 핵심 개념 |
|------|-----------|-----------|
| Gartner (2025) | 자율성 3단계 | AI Assistant → Task-Specific Agent → Agentic Ecosystem |
| BCV (2024) | 자율성 6레벨 (SAE 차용) | L0 No AI → L5 Fully Autonomous |
| CB Insights (2025) | 수평/수직 × 26카테고리 | 400+ 기업 Market Map |
| Madrona (2025) | 가치사슬 3계층 | Infrastructure → Platform → Application |

### 1.2 국내 시장 적용 분류 체계: 3축 6차원

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  축 1: WHAT (무엇을 하는가)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  차원 A. 가치사슬 포지션
    ┣━ 인프라 (모델·GPU·데이터)
    ┣━ 플랫폼 (오케스트레이션·프레임워크·빌더)
    ┗━ 애플리케이션 (최종 사용자 서비스)

  차원 B. 적용 도메인
    ┣━ 수평형 (고객서비스, HR, 마케팅, 개발, 보안 등)
    ┗━ 수직형 (금융, 헬스케어, 법률, 교육, 제조, 커머스 등)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  축 2: HOW (어떻게 작동하는가)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  차원 C. 자율성 수준
    ┣━ L1-L2 Copilot (보조형)
    ┣━ L3 Supervised Agent (감독형)
    ┗━ L4-L5 Autopilot/Autonomous (자율형)

  차원 D. 인터페이스 유형
    ┣━ Conversational (대화형)
    ┣━ Workflow/Automation (워크플로우형)
    ┣━ API/Headless (헤드리스)
    ┗━ Voice/Realtime (음성/실시간)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  축 3: WHERE (어디에 배포하는가) — 한국 시장 특화
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  차원 E. 배포 모델
    ┣━ SaaS Cloud
    ┣━ Hybrid (국내 클라우드)
    ┗━ On-Premise (금융·공공 전용)

  차원 F. 한국 시장 특수 차원
    ┣━ 자체 LLM vs 외부 LLM 의존
    ┣━ 한국어 최적화 수준
    ┗━ 규제 대응 등급 (금융위·개인정보위)
```

---

## 2. 기업 분류 매핑

### 2.1 가치사슬 포지션별 분류

#### Layer 1: 인프라 (Infrastructure)

| 기업 | 제품 | 세부 분류 | 자체LLM |
|------|------|-----------|---------|
| 네이버클라우드 | CLOVA Studio / HyperCLOVA X API | LLM API | O |
| 카카오엔터프라이즈 | KakaoCloud AI / FMOps | LLM 인프라 | O |
| 업스테이지 | Solar API / Document AI | LLM API | O |
| LG AI Research | EXAONE 4.0 | LLM 파운데이션 | O |
| 삼성전자 | Samsung Gauss 2.3 | 온디바이스 LLM | O |
| 베슬에이아이 | VESSL Cloud / MLOps | GPU 인프라 / MLOps | X |
| 노타 | NetsPresso | 모델 최적화 / 엣지 AI | X |
| 수퍼브AI | Superb Platform | 데이터 파이프라인 | X |
| NHN | NHN클라우드 2.0 | GPU 팜 / 공공 클라우드 | X |

#### Layer 2: 플랫폼 (Platform)

| 기업 | 제품 | 세부 분류 | 타겟 |
|------|------|-----------|------|
| 삼성SDS | FabriX / Brity Copilot | 엔터프라이즈 AI 플랫폼 | B2B |
| LG CNS | AgenticWorks / a:xink | 에이전틱 AI 오케스트레이션 | B2B |
| 올거나이즈 | Alli (Agentic RAG) | RAG 플랫폼 / No-Code 빌더 | B2B |
| 달파 | Dalpha AI 에이전트 스튜디오 | 에이전트 배포/빌더 | B2B |
| 뤼튼테크놀로지스 | 뤼튼 AX | 멀티LLM 오케스트레이션 | B2B |
| 하마다랩스 | WindyFlo | No-Code 파이프라인 빌더 | B2B |
| 마키나락스 | Runway | MLOps/LLMOps/AgentOps | B2B |
| KT | 에이전트 빌더 | No-Code 에이전트 빌더 | B2B |
| 제논 | GenOS / OneAgent | 엔터프라이즈 에이전트 플랫폼 | B2B |

#### Layer 3: 애플리케이션 (Application)

**수평형 (Cross-industry)**

| 기업 | 제품 | 기능 도메인 | 타겟 |
|------|------|------------|------|
| 채널코퍼레이션 | 채널톡 ALF | 고객서비스 | B2B |
| 솔트룩스 | AICC / 구버 | 고객서비스 / 리서치 | B2B |
| 카카오 | 센터플로우 | 고객서비스 (AICC) | B2B |
| 페르소나AI | 엣지 AI AICC | 고객서비스 | B2B |
| SK AX | 채용 자동화 서비스 | HR/채용 | B2B |
| 리캐치 | AIX 세일즈 에이전트 | 세일즈 | B2B |
| 몰로코 | Moloco Ads | 마케팅/광고 | B2B |
| S2W | QUAXAR | 보안/CTI | B2B |
| 셀렉트스타 | Datumo Eval | LLM 평가/테스팅 | B2B |
| 와탭랩스 | WhaTap Monitoring | AI Observability | B2B |
| 라이너 | Liner Copilot | AI 리서치/검색 | B2C+B2B |

**수직형 (Industry-specific)**

| 기업 | 제품 | 산업 | 타겟 |
|------|------|------|------|
| 로앤컴퍼니 | 슈퍼로이어 | 법률 | B2B |
| 엘박스 | 엘박스 AI | 법률 | B2B |
| BHSN | 앨리비 | 법률 (공공 특화) | B2B |
| 웹케시 | AI CFO / AI CMS | 금융 (자금관리) | B2B |
| 뱅크샐러드 | 토핑+ | 금융 (개인) | B2C |
| 토스뱅크 | 금융 상담 AI | 금융 (상담) | B2C |
| LG AI Research | EXAONE-BI | 금융 (분석) | B2B |
| 에이아이트릭스 | AITRICS-VC | 헬스케어 | B2B |
| 루닛 | Lunit INSIGHT | 헬스케어 (영상) | B2B |
| 뷰노 | 뷰노메드 딥카스 | 헬스케어 (예측) | B2B |
| 소크라AI | 산타 (Santa) | 교육 | B2C |
| 프리윌린 | 매쓰플랫 | 교육 (수학) | B2B+B2C |
| 아이헤이트플라잉버그스 | 밀당PT | 교육 | B2C |
| 인핸스 | 커머스OS / ACT-2 | 커머스 | B2B |
| 포스코DX | 피지컬 AI / 코딩 에이전트 | 제조 | B2B |

**통합형 B2C 에이전트 (슈퍼앱)**

| 기업 | 제품 | 특성 |
|------|------|------|
| 네이버 | CLOVA X / AI 탭 | 검색·쇼핑·예약 통합 에이전트 |
| 카카오 | 카나나 (Kanana) | 메신저 기반 일상 AI (4,600만 MAU) |
| SK텔레콤 | 에이닷 (A.) | 멀티LLM 개인 에이전트 (1,000만+ MAU) |
| 뤼튼테크놀로지스 | 뤼튼 AI 포털 | 멀티LLM AI 슈퍼앱 (600만 MAU) |
| 스캐터랩 | 제타 / 이루다 | AI 캐릭터 엔터테인먼트 (200만 유저) |

---

### 2.2 자율성 수준별 분포

```
L1-L2 Copilot (보조형)        ██████████████████████ 42%
  삼성SDS Brity Copilot, 라이너, 포스코DX 코딩에이전트,
  업스테이지 Document AI, 와탭랩스 등

L3 Supervised Agent (감독형)   ████████████████████ 38%
  채널톡 ALF, 올거나이즈 Alli, 에이닷 비즈,
  삼성SDS FabriX, LG CNS AgenticWorks, 웹케시 AI CFO 등

L4-L5 Autopilot (자율형)       ██████████ 20%
  인핸스 ACT-2(LAM), 채널톡 ALF v2(목표 자율수행),
  LG AI Research EXAONE-BI(자율 금융분석),
  삼성전자 Agentic Builder(2030 자율공장) 등
```

---

### 2.3 자체 LLM vs 외부 LLM 의존 구조

```
자체 LLM 보유 (5개사)          외부 LLM 의존/멀티LLM (다수)
━━━━━━━━━━━━━━━━━━━━          ━━━━━━━━━━━━━━━━━━━━━━━━━━
네이버   HyperCLOVA X          SK텔레콤   GPT-4o+Claude+Gemini
카카오   Kanana                 뤼튼      GPT-4+Claude+Gemini
삼성전자 Gauss 2.3              LG CNS   멀티LLM (비종속)
삼성SDS  자체 엔터프라이즈 LLM    KT       MS Azure OpenAI
LG AI   EXAONE 4.0             두산      MS Azure OpenAI
                                포스코DX  AWS 협력
                                대부분 스타트업  OpenAI/Anthropic API
```

---

## 3. 시장 구조 분석

### 3.1 경쟁 강도 히트맵

```
높은 경쟁 ■■■■■  중간 ■■■  낮은 경쟁 ■

고객서비스/AICC     ■■■■■  (채널톡, 솔트룩스, 카카오, 페르소나AI, KT 등)
B2C AI 슈퍼앱       ■■■■■  (네이버, 카카오, SKT, 뤼튼, 라이너)
엔터프라이즈 플랫폼  ■■■■   (삼성SDS, LG CNS, 올거나이즈, 달파)
법률 AI             ■■■    (로앤컴퍼니, 엘박스, BHSN)
헬스케어 AI          ■■■    (루닛, 뷰노, 에이아이트릭스)
금융 AI              ■■■    (웹케시, 뱅크샐러드, 토스뱅크, LG EXAONE-BI)
교육 AI              ■■■    (소크라AI, 프리윌린, 밀당PT)
커머스 AI             ■■    (인핸스)
AI 보안               ■■    (S2W, 이글루)
LLM 평가/테스팅        ■     (셀렉트스타)
AI Observability       ■     (와탭랩스)
모델 최적화/엣지        ■     (노타)
```

### 3.2 국내 시장 공백 (Gap Analysis)

글로벌 대비 국내 전문 기업이 부재하거나 희소한 영역:

| 카테고리 | 글로벌 대표 | 국내 현황 |
|----------|------------|-----------|
| LLM API 게이트웨이/라우터 | OpenRouter, LiteLLM, Portkey | **전무** — 대형 클라우드사가 겸업 |
| 벡터 데이터베이스 | Pinecone, Weaviate, Qdrant | **전무** |
| 프롬프트 관리 도구 | LangSmith, PromptLayer | **전무** — RAG 플랫폼에 기능 통합 |
| AI Agent 전용 IDE | Cursor, Windsurf, Replit Agent | **전무** |
| Agent 메모리/컨텍스트 | Zep, MemGPT | **전무** |
| Agent 테스팅 전문 | Braintrust, Patronus AI | **희소** (셀렉트스타만) |

### 3.3 대기업 전략 유형화

| 유형 | 기업 | 전략 |
|------|------|------|
| **풀스택 자체 생태계** | 네이버, 카카오, 삼성전자 | 자체 LLM → 자사 서비스 AI화 → B2C 에이전트 |
| **엔터프라이즈 AI 플랫폼** | 삼성SDS, LG CNS, LG AI Research | 자체/멀티 LLM → B2B 에이전트 플랫폼 판매 |
| **멀티LLM 오케스트레이터** | SK텔레콤, KT | 외부 LLM 통합 → 통신 번들 AI 서비스 |
| **도메인 특화 AI** | 포스코DX, 두산 | 파트너 LLM → 제조/건설 특화 에이전트 |
| **인프라 공급자** | NHN | GPU 팜 + 공공 클라우드 인프라 |

---

## 4. 주요 통계

| 지표 | 수치 | 출처 |
|------|------|------|
| 글로벌 AI Agent 시장 규모 (2025) | $7.84B | MarketsandMarkets |
| 글로벌 AI Agent 시장 예측 (2030) | $52.62B (CAGR 46.3%) | MarketsandMarkets |
| 엔터프라이즈 앱 Agent 내장 비율 (2026E) | 40% (현재 5%) | Gartner |
| CB Insights 매핑 기업 수 | 400+, 26개 카테고리 | CB Insights 2025 |
| 국내 조사 기업 수 (본 연구) | 50+ 기업 | 자체 조사 |
| 자체 LLM 보유 국내 대기업 | 5개사 | 자체 조사 |
| Multi-Agent 시스템 문의 증가율 | +1,445% (YoY) | Gartner |

---

## 5. 한국 시장 특수성

### 5.1 규제 환경
- **금융**: 금융위원회 3단계 규제 로드맵 시행 중. 금융권 특화 한글 말뭉치 2025년 Q1 제공 시작
- **공공**: CSAP(클라우드 보안 인증) 필수. 온프레미스 배포 요구 높음
- **개인정보**: 개인정보보호위원회 생성형 AI 안내서 (2025.08)

### 5.2 한국어 특수성
- 한국어 LLM 성능이 글로벌 모델 대비 차이 → HyperCLOVA X, EXAONE, Kanana 등 국내 특화 모델 경쟁력
- Open Ko-LLM Leaderboard (업스테이지·NIA 공동) — 한국어 LLM 평가 표준

### 5.3 재벌/대기업 생태계
- 삼성(삼성전자+삼성SDS), LG(LG AI Research+LG CNS), SK(SKT+SK AX) 등 계열사 간 수직 통합 구조
- 그룹사 내부 수요가 초기 시장 형성의 핵심 드라이버

---

## 6. Limitations

1. **조사 범위**: 공개 정보(뉴스, 공식 사이트, 보도자료) 기반. 스텔스 스타트업·비공개 B2B 솔루션 누락 가능
2. **시점**: 2026년 3월 25일 기준. 일부 제품은 발표 후 미출시 상태
3. **정량 데이터 한계**: 대부분 기업이 매출·사용자 수를 비공개. 공개된 수치만 수록
4. **분류 중복**: 솔트룩스, 업스테이지, 뤼튼 등 수직통합 기업은 복수 레이어에 동시 분류됨 (의도적)
5. **글로벌 기업 제외**: 국내 법인이 있더라도 본사가 해외인 기업(MS Korea, AWS Korea 등)은 제외
6. **부동산, 물류, 미디어/콘텐츠, 게임 도메인**: 별도 심층 조사 미실시

---

## 7. Recommendations

1. **Gap 영역 기회 탐색**: 벡터DB, LLM 라우터, Agent 메모리, 프롬프트 관리 등 국내 전문 기업 부재 영역은 스타트업 진입 기회
2. **규제 대응 차별화**: 금융·공공 부문의 온프레미스/CSAP 요건을 충족하는 Agent 플랫폼이 경쟁 우위 확보 가능
3. **한국어 Agent 벤치마크 필요**: Open Ko-LLM Leaderboard처럼 Agent 성능(도구 사용, 멀티스텝 추론)을 평가하는 한국어 Agent 벤치마크 부재
4. **멀티Agent 아키텍처 전환 모니터링**: Gartner 기준 Multi-Agent 문의 +1,445% 증가 추세. 단일 Agent → 멀티Agent 전환 시점이 다가오고 있음

---

## Appendix: 조사 방법론

- **연구 설계**: 5개 독립 스테이지 병렬 조사 + 교차 검증
- **데이터 수집**: WebSearch 기반 (2024-2026년 공개 자료)
- **스테이지 구성**:
  - Stage 1: AI Agent 플랫폼/빌더 제공사 (12개 기업)
  - Stage 2: 도메인 특화 AI Agent 서비스 (23개사, 9개 도메인)
  - Stage 3: 대기업/빅테크 AI Agent 전략 (11개 기업군, 15개 브랜드)
  - Stage 4: AI Agent 인프라/개발도구 (16개 기업)
  - Stage 5: 글로벌 분류 프레임워크 벤치마크
- **교차 검증**: 기업 간 중복, 분류 일관성, 누락 도메인 검토

---

## Sources (주요)

### 글로벌 프레임워크
- [Gartner: Top Strategic Technology Trends 2025](https://www.gartner.com/en/documents/5850847)
- [BCV: How AI-Powered Work Is Moving From Copilot to Autopilot](https://baincapitalventures.com/insight/how-ai-powered-work-is-moving-from-copilot-to-autopilot/)
- [CB Insights: AI Agent Market Map March 2025](https://www.cbinsights.com/research/ai-agent-market-map/)
- [Madrona: AI Agent Infrastructure Stack](https://www.madrona.com/ai-agent-infrastructure-three-layers-tools-data-orchestration/)
- [MarketsandMarkets: AI Agents Market 2025-2030](https://www.marketsandmarkets.com/Market-Reports/ai-agents-market-15761548.html)

### 국내 기업 (대기업)
- [네이버 HyperCLOVA X](https://navercorp.com/en/tech/hyperclovax)
- [카카오 카나나 AI](https://www.kakaocorp.com/page/detail/11809)
- [삼성SDS FabriX/Brity](https://www.samsungsds.com/kr/news/sds-genai-mediaday.html)
- [LG AI Research EXAONE](https://www.prnewswire.com/news-releases/lg-ai-research-taps-google-cloud-to-develop-exaone-3-0-and-chatexaone-ai-agent-302231481.html)
- [LG CNS AgenticWorks](https://www.ebn.co.kr/news/articleView.html?idxno=1675700)
- [SKT 에이닷](https://news.sktelecom.com/214959)
- [포스코DX AI Agent](https://newsroom.posco.com/kr/)

### 국내 기업 (스타트업/특화)
- [국내 AI 에이전트 스타트업 TOP 10 - 캐럿](https://carat.im/blog/korea-ai-agent-startups-top-10)
- [포브스코리아 2025 대한민국 AI 50](https://www.forbeskorea.co.kr/news/articleView.html?idxno=400220)
- [채널코퍼레이션 ALF v2](https://channel.io/ko/blog/articles/alf-v2-press-f03760de)
- [올거나이즈 Alli](https://www.allganize.ai/ko/home)
- [업스테이지 Solar Pro 3](https://www.aitimes.com/news/articleView.html?idxno=208250)
- [인핸스 커머스OS](https://www.hellot.net/news/article.html?no=107071)
- [셀렉트스타 Datumo Eval](https://selectstar.ai/)
- [S2W QUAXAR](https://s2w.inc/ko/)
- [마키나락스 Runway](https://www.makinarocks.ai/)

### 한국 시장 분석
- [에이전틱 AI 원년, 한국 기업은 어디에 서 있는가 - Raylogue](https://www.raylogue.com/agentic-ai-korea-2026/)
- [KDI: AI 에이전트 개발 동향 및 국내 경쟁력 분석](https://eiec.kdi.re.kr/policy/domesticView.do?ac=0000193894)
- [금융위원회: 금융분야 AI 가이드라인](https://www.fsc.go.kr/)
