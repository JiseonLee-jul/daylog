# Claude Code Plugin

Claude Code의 확장 메커니즘으로, **슬래시 커맨드 / 스킬 / 훅 / 스크립트**를 패키징해 개발자가 반복 워크플로를 재사용 가능한 도구로 배포할 수 있게 한다. 마켓플레이스를 통해 설치/공유되며, 플러그인 시스템은 경쟁사 도구까지 통합 가능할 정도로 개방적이다.

## Key Points

- **구성 요소**: 슬래시 커맨드 (`.md` 프롬프트), 스킬 (SKILL.md), 훅 (hooks.json 기반 이벤트 리스너), 스크립트 (Python/Shell 유틸리티), MCP 서버
- **마켓플레이스**: `/plugin marketplace add <source>` → `/plugin install <name>@<marketplace>` → `/reload-plugins` 흐름으로 설치
- **네임스페이스 구조**: 플러그인 이름이 슬래시 커맨드의 네임스페이스가 됨 (예: `/plugin-name:command`)
- **설정 분리**: marketplace.json(번들 식별)과 plugin.json(개별 플러그인 식별)이 독립적 — 마켓플레이스 이름과 플러그인 이름이 달라도 됨
- **헤드리스 지원**: `claude -p --plugin-dir <path>` 로 플러그인 명령을 스크립트에서 호출 가능
- **경쟁사 통합 개방**: OpenAI Codex 같은 타사 모델도 플러그인으로 제공 가능 — 생태계 개방성

## 플러그인 구조 예시

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json         # 매니페스트
├── commands/
│   └── command.md          # 슬래시 커맨드 (프롬프트 + allowed-tools)
├── skills/
│   └── skill-name/
│       └── SKILL.md        # 스킬 정의
├── scripts/
│   └── helper.py           # 보조 Python 스크립트
├── hooks/
│   └── hooks.json          # 이벤트 훅
└── README.md
```

## 생태계 개방성의 의미

OpenAI가 Apache-2.0 라이선스로 Claude Code용 Codex 플러그인을 공개한 것은 **경쟁사 생태계에 자사 도구를 얹는 전략**의 사례다. 사용자가 어떤 플랫폼을 선택하든 OpenAI 모델의 사용량을 확보하려는 접근으로, 모델 제공자 간 경계가 사용자 레벨에서 점점 흐려지고 있음을 보여준다. Claude Code는 이런 타사 플러그인을 자연스럽게 수용하는 구조로 설계됐으며, 이는 ecosystem lock-in보다 openness를 우선시하는 철학이다.

## Sources
- [260331_claude_code_hidden_features](../summaries/260331_claude_code_hidden_features.md)
- [260331_openai_codex_plugin_for_claude_code](../summaries/260331_openai_codex_plugin_for_claude_code.md)

## Related Concepts
- [claude-code](claude-code.md) — 상위 플랫폼
- [openai-codex](openai-codex.md) — 주목할 만한 타사 플러그인 사례
