# OpenAI의 Claude Code용 Codex 플러그인

> 출처: https://news.hada.io/topic?id=28023

## 개요

OpenAI가 "We love an open ecosystem!"이라는 메시지와 함께 Claude Code 안에서 직접 OpenAI Codex를 호출할 수 있는 플러그인을 공개했다. Apache-2.0 라이선스. 경쟁 제품의 플러그인 생태계에 자사 도구를 얹는 흥미로운 전략적 움직임.

## 슬래시 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/codex:review` | 읽기 전용 코드 리뷰 |
| `/codex:adversarial-review` | 설계 결정에 대한 비판적 검토. 단순 리뷰가 아니라 "왜 이렇게 했는가"를 파고드는 방식 |
| `/codex:rescue` | 버그 조사 및 수정을 Codex에 위임. 막혔을 때 다른 모델의 관점을 빌리는 용도 |
| `/codex:status` | 백그라운드 작업 현황 조회 |
| `/codex:result` | 완료된 작업 결과 확인 |
| `/codex:cancel` | 진행 중인 백그라운드 작업 취소 |

## 주요 특징

- ChatGPT 구독 또는 API 키로 사용 가능. 별도 런타임 설치 불필요.
- 비동기 처리를 지원해서 장시간 작업도 백그라운드에서 돌릴 수 있다.
- Review Gate 기능으로 자동 검증 루프를 구성할 수 있다. CI처럼 리뷰-수정 사이클을 자동화하는 개념.

## 커뮤니티 반응

Claude Code 자체가 이 플러그인을 "really well-engineered"이라 칭찬한 점이 화제가 됐다. 사용자들은 "우리 플랫폼 써도 되니까 일은 우리한테 맡겨"라는 OpenAI의 전략적 접근으로 해석하고 있다. 경쟁사 생태계에 올라타서 사용량을 확보하려는 의도.

---

## 참고: 설치 방법

### Step 1. 사전 준비

| 필요한 것 | 설명 |
|-----------|------|
| Node.js 18.18+ | Claude Code를 쓰고 있다면 이미 설치되어 있을 것 |
| ChatGPT 계정 또는 OpenAI API 키 | 무료 계정도 가능 |

### Step 2. 플러그인 설치 (Claude Code 안에서)

```bash
# 1. 마켓플레이스 추가
/plugin marketplace add openai/codex-plugin-cc

# 2. 플러그인 설치
/plugin install codex@openai-codex

# 3. 플러그인 리로드
/reload-plugins

# 4. 설치 상태 확인
/codex:setup
```

setup 과정에서 Codex CLI가 없으면 자동으로 설치를 제안한다. 수동 설치도 가능:

```bash
npm install -g @openai/codex
```

### Step 3. 로그인

Codex에 이미 로그인되어 있으면 바로 사용 가능. 아니라면:

```bash
!codex login
```

> `!` 접두사는 Claude Code 안에서 쉘 명령을 직접 실행하는 문법이다.
