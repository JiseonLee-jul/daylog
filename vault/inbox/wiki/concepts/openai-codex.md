# OpenAI Codex

OpenAI의 코드 생성 및 리뷰 도구. Claude Code용 Codex 플러그인이 Apache-2.0 라이선스로 공개되면서 **경쟁 제품의 생태계에 자사 도구를 배치**하는 전략적 사례로 주목받았다. ChatGPT 구독 또는 OpenAI API 키로 사용 가능하며, 비동기 처리와 Review Gate 기능을 통해 자동 검증 루프를 구성할 수 있다.

## Key Points

- **Apache-2.0 라이선스**: 오픈 라이선스로 공개되어 누구나 포크/수정/배포 가능
- **Claude Code 통합**: `/codex:*` 슬래시 커맨드로 Claude Code 세션 안에서 직접 호출
- **비동기 처리**: 장시간 작업을 백그라운드에서 실행, `/codex:status`로 현황 조회
- **Review Gate**: CI 스타일 자동 검증 루프 — 리뷰-수정 사이클 자동화
- **듀얼 인증**: ChatGPT 구독 또는 API 키 둘 다 지원
- **CLI 통합**: `@openai/codex` npm 패키지로 전역 설치, `/codex:setup`에서 자동 설치 제안

## 슬래시 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/codex:review` | 읽기 전용 코드 리뷰 |
| `/codex:adversarial-review` | 설계 결정에 대한 비판적 검토 — "왜 이렇게 했는가"를 파고드는 방식 |
| `/codex:rescue` | 버그 조사/수정을 Codex에 위임 — 막혔을 때 다른 모델의 관점 활용 |
| `/codex:status` | 백그라운드 작업 현황 조회 |
| `/codex:result` | 완료된 작업 결과 확인 |
| `/codex:cancel` | 진행 중인 백그라운드 작업 취소 |

## Adversarial Review의 의미

일반 코드 리뷰가 "문제점을 찾아 수정 제안"이라면, adversarial review는 "설계 결정의 정당성을 공격적으로 검증"한다. 단순히 "이 코드가 잘 동작하는가?"가 아니라 "왜 이렇게 설계했는가?"를 파고들어, 설계자가 간과한 대안과 트레이드오프를 드러낸다. 이는 모델이 단일 관점의 에코 챔버에 빠지는 것을 방지하는 메커니즘으로 활용된다.

## Rescue 패턴 — 다른 모델의 관점

`/codex:rescue`는 "막혔을 때 다른 모델의 관점을 빌린다"는 독특한 사용 패턴을 제시한다. 동일한 모델로 계속 시도하면 같은 편향에 빠질 수 있지만, 다른 모델(Codex)에 위임하면 새로운 접근법이 나올 수 있다. 이는 모델 다양성을 활용한 집단 지성(collective intelligence) 전략이다.

## 전략적 의미

OpenAI가 경쟁사(Anthropic) 생태계에 자사 도구를 얹은 것은 다음을 시사한다:

- **사용량 확보 우선**: 사용자가 어떤 플랫폼을 쓰든 OpenAI 모델의 호출량을 확보
- **생태계 경계 희석**: 모델 제공자와 개발 환경의 lock-in이 사용자 레벨에서 약해짐
- **개방성 트렌드**: Claude Code가 이를 "really well-engineered"라고 칭찬한 것은 양측이 개방적 생태계를 지향함을 보여줌

## Sources
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md)

## Related Concepts
- [claude-code-plugin](claude-code-plugin.md) — Codex가 제공되는 플러그인 시스템
