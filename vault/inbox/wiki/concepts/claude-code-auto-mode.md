# Claude Code Auto Mode

Claude Code의 자율 실행 모드로, 사용자의 93%가 관성적으로 권한 프롬프트를 승인하는 **"승인 피로(approval fatigue)"** 문제를 해결하기 위해 설계됐다. 핵심은 **모델 기반 분류기**가 대부분의 승인 결정을 자동화하되, 위험 행동만 선별적으로 차단하는 구조다. 샌드박스(안전하지만 고비용)와 `--dangerously-skip-permissions`(보호 없음) 사이의 중간 지점을 목표로 한다.

## 아키텍처: 2-Layer Defense

Auto Mode는 두 겹의 방어 레이어로 구성된다:

1. **입력 레이어 — 프롬프트 인젝션 프로브**: 파일 읽기, 웹 fetch, 셸 출력, 외부 툴 응답 등 툴 출력물을 에이전트 컨텍스트에 전달되기 전에 스캔. 행동 하이재킹 시도 감지 시 경고를 컨텍스트에 주입.
2. **출력 레이어 — 트랜스크립트 분류기**: Sonnet 4.6 기반으로 각 행동을 실행 전 평가. 분류기는 **어시스턴트 메시지와 툴 결과를 구조적으로 제거**하고 사용자 메시지와 툴 호출만 참조 — 에이전트 자체의 합리화 텍스트에 영향받지 않기 위함.

## Key Points

- **2-Stage Classification**: Stage 1은 빠른 단일 토큰(yes/no) 필터로 차단 방향 편향(8.5% FPR), Stage 2는 플래그된 경우에만 체인 오브 생각 추론으로 FPR을 0.4%까지 감소
- **3-Tier Permission System**: Tier 1(허용 툴, 분류기 호출 없음), Tier 2(프로젝트 내 파일, 버전 관리로 검토), Tier 3(셸/웹/외부 툴, 분류기 필수)
- **Deny-and-Continue**: 차단 시 세션 중단 없이 안전한 대안 시도 유도. 연속 3회 또는 총 20회 차단 시 사람에게 에스컬레이션
- **평가 성능**: 실제 과도 행동 데이터셋(n=52)에서 FNR 17% — `--dangerously-skip-permissions` 대비 실질적 개선이지만 고위험 인프라의 대체재는 아님
- **멀티 에이전트 핸드오프 검사**: 서브에이전트 위임(outbound)과 결과 반환(return) 양쪽에서 분류기 실행 — 오케스트레이터 지시가 "사용자 메시지"처럼 보이는 문제 방지

## 설계 철학

분류기가 완벽할 필요는 없다. 충분한 위험 행동을 차단해 **가드레일 없는 상태 대비 자율 작동을 실질적으로 더 안전하게** 만드는 것이 목표다. 사용자들에게 잔여 위험 인식과 자율 실행 범위 판단을 유지하도록 권장한다.

## Sources
- [260327_anthropic-claude-code-auto-mode-design](../summaries/260327_anthropic-claude-code-auto-mode-design.md)

## Related Concepts
- [claude-code](claude-code.md) — 상위 플랫폼
- [prompt-injection](prompt-injection.md) — Auto Mode가 방어하는 핵심 위협 중 하나
