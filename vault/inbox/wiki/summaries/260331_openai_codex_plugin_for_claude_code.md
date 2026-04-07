---
source: raw/260331_openai_codex_plugin_for_claude_code.md
compiled: 2026-04-07
topics: [openai-codex, claude-code-plugin, adversarial-review, plugin-ecosystem]
---

# Summary: OpenAI의 Claude Code용 Codex 플러그인

OpenAI가 Apache-2.0 라이선스로 Claude Code 안에서 직접 OpenAI Codex를 호출할 수 있는 플러그인을 공개했다. "We love an open ecosystem!" 메시지와 함께 발표된 이 움직임은 경쟁 제품의 플러그인 생태계에 자사 도구를 얹는 전략적 접근으로 해석된다 — 사용자가 어떤 플랫폼을 선택하든 OpenAI의 모델 사용량을 확보하려는 의도다.

플러그인은 6개의 슬래시 커맨드를 제공한다: `/codex:review`(읽기 전용 코드 리뷰), `/codex:adversarial-review`(설계 결정에 대한 비판적 검토 — 단순 리뷰가 아니라 "왜 이렇게 했는가"를 파고드는 방식), `/codex:rescue`(버그 조사 및 수정을 Codex에 위임, 막혔을 때 다른 모델의 관점을 빌리는 용도), `/codex:status`/`/codex:result`/`/codex:cancel`(백그라운드 작업 관리).

주요 특징으로는 ChatGPT 구독 또는 API 키로 사용 가능하며, 별도 런타임 설치가 필요 없고, 비동기 처리로 장시간 작업을 백그라운드에서 돌릴 수 있다. **Review Gate** 기능은 CI처럼 자동 검증 루프를 구성하는 개념으로, 리뷰-수정 사이클을 자동화한다. Claude Code 자체가 이 플러그인을 "really well-engineered"라고 칭찬한 점이 커뮤니티에서 화제가 됐다.

설치는 `/plugin marketplace add openai/codex-plugin-cc` → `/plugin install codex@openai-codex` → `/reload-plugins` → `/codex:setup` 순서로 진행한다. setup 과정에서 Codex CLI가 없으면 자동 설치를 제안한다. `!codex login`으로 `!` 접두사를 활용해 Claude Code 안에서 쉘 명령을 직접 실행해 로그인할 수 있다. 이 플러그인은 경쟁사 생태계에 올라타는 OpenAI의 새로운 전략을 보여주는 사례로, 모델 제공자 간 경계가 사용자 레벨에서 점점 흐려지고 있음을 시사한다.

## Key Points
- Apache-2.0 오픈 라이선스로 공개된 Claude Code용 OpenAI Codex 플러그인
- 6개 슬래시 커맨드: review, adversarial-review, rescue, status, result, cancel
- Adversarial review는 단순 리뷰가 아닌 "왜 이렇게 했는가"를 파고드는 비판적 검토
- Rescue는 막혔을 때 다른 모델(Codex)의 관점을 빌리는 위임 메커니즘
- Review Gate로 자동 검증 루프 (CI 스타일 리뷰-수정 사이클) 구성 가능
- 비동기 처리 지원으로 장시간 백그라운드 작업 가능
- 전략적 의미: OpenAI가 경쟁 제품 생태계에 자사 도구 배치 — 사용자 레벨에서 모델 제공자 경계 희석

## Related Concepts
- [openai-codex](../concepts/openai-codex.md)
- [claude-code-plugin](../concepts/claude-code-plugin.md)
- [claude-code](../concepts/claude-code.md)
