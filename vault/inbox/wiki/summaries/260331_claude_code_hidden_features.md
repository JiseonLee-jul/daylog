---
source: raw/260331_claude_code_hidden_features.md
compiled: 2026-04-07
topics: [claude-code, cli-flags, hooks, scheduled-tasks, git-worktrees, custom-agents]
---

# Summary: Claude Code의 숨겨진 강력한 기능들 15가지

Claude Code 제작자 Boris Cherny가 정리한 15개 기능 목록은 Claude Code가 단순 코드 생성 도구를 넘어 개발 워크플로 전체를 자동화하는 플랫폼으로 진화하고 있음을 보여준다. 기능들은 크게 세 영역으로 분류할 수 있다: (1) 크로스 플랫폼 접근성(모바일 앱, Chrome 확장, 세션 이동/원격 제어), (2) 세션/프로세스 관리(세션 포크, /btw 사이드 쿼리, Git Worktrees, /batch), (3) 자동화/확장(/loop, /schedule, Hooks, 커스텀 에이전트, /voice, --bare, --add-dir).

주목할 만한 자동화 기능으로는 최대 1주일 단위 스케줄 설정이 가능한 `/loop`과 `/schedule`이 있어 PR 관리, 코드 리뷰, Slack 피드백 같은 반복 작업을 완전 자동화할 수 있다. Hooks는 에이전트 라이프사이클의 특정 시점(예: 파일 저장 전/후)에 커스텀 로직을 삽입해 린트 자동 실행 같은 워크플로를 구성한다. 커스텀 에이전트는 `.claude/agents` 디렉토리에 에이전트 정의 파일을 두어 프로젝트에 특화된 반복 워크플로를 캡슐화한다.

개발 생산성 도구로는 `/branch`로 현재 세션을 분기해 여러 접근 방식을 동시에 시도할 수 있고, Git Worktrees로 여러 브랜치를 동시 체크아웃해 병렬 개발이 가능하며, `/batch`는 수십~수천 개의 워크트리 에이전트에 작업을 분산해 대규모 리팩토링이나 마이그레이션에 활용할 수 있다. `/btw`는 메인 작업 흐름을 중단하지 않고 사이드 쿼리를 던질 수 있어 작업 중 갑자기 궁금한 게 생겼을 때 맥락을 잃지 않는다.

성능과 멀티레포 지원을 위한 플래그도 주목할 만하다. `--bare` 플래그는 SDK 시작 속도를 최대 10배 향상시키는 경량 모드로 자동화 파이프라인에 유용하고, `--add-dir`는 모노레포가 아닌 환경에서 여러 프로젝트를 동시에 다룰 때 쓴다. `/teleport`/`/remote-control`은 클라우드 세션을 로컬 머신에서 이어받거나 원격 제어하는 기능이다.

## Key Points
- Mobile/Chrome/Teleport로 크로스 플랫폼 접근성 확보
- `/loop`, `/schedule`로 최대 1주일 단위 반복 작업 완전 자동화
- Hooks로 에이전트 라이프사이클 시점 커스텀 로직 삽입 (예: 파일 저장 전후 린트)
- Git Worktrees + `/batch`로 대규모 병렬 작업 지원
- 커스텀 에이전트로 프로젝트 특화 워크플로 캡슐화
- `--bare` (SDK 10x 시작 속도), `--add-dir` (멀티레포 접근)

## Related Concepts
- [claude-code](../concepts/claude-code.md)
- [claude-code-plugin](../concepts/claude-code-plugin.md)
