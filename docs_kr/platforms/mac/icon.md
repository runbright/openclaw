---
summary: "macOS에서의 메뉴 바 아이콘 상태 및 애니메이션"
read_when:
  - 메뉴 바 아이콘 동작 변경
title: "메뉴 바 아이콘"
---

# 메뉴 바 아이콘 상태

작성자: steipete · 업데이트: 2025-12-06 · 범위: macOS 앱 (`apps/macos`)

- **대기:** 일반 아이콘 애니메이션 (깜빡임, 가끔 흔들림).
- **일시 중지:** 상태 항목이 `appearsDisabled`를 사용; 움직임 없음.
- **음성 트리거 (큰 귀):** 음성 웨이크 감지기가 웨이크 워드를 들으면 `AppState.triggerVoiceEars(ttl: nil)`을 호출하여, 발화가 캡처되는 동안 `earBoostActive=true`를 유지합니다. 귀가 확대되고 (1.9배), 가독성을 위해 원형 귀 구멍이 생기며, 1초간의 침묵 후 `stopVoiceEars()`를 통해 감소합니다. 인앱 음성 파이프라인에서만 발생합니다.
- **작업 중 (에이전트 실행 중):** `AppState.isWorking=true`가 "꼬리/다리 움직임" 미세 동작을 구동합니다: 작업 진행 중 더 빠른 다리 흔들림과 약간의 오프셋. 현재 WebChat 에이전트 실행 시 전환됩니다; 다른 긴 작업을 연결할 때 동일한 전환을 추가하세요.

연결 포인트

- 음성 웨이크: 런타임/테스터가 트리거 시 `AppState.triggerVoiceEars(ttl: nil)`을 호출하고 캡처 윈도우에 맞춰 1초 침묵 후 `stopVoiceEars()`를 호출합니다.
- 에이전트 활동: 작업 구간 주위에 `AppStateStore.shared.setWorking(true/false)`를 설정합니다 (이미 WebChat 에이전트 호출에서 수행됨). 구간을 짧게 유지하고 `defer` 블록에서 초기화하여 멈춘 애니메이션을 방지하세요.

모양 및 크기

- 기본 아이콘은 `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)`에서 그려집니다.
- 귀 스케일 기본값은 `1.0`; 음성 부스트는 전체 프레임을 변경하지 않고 `earScale=1.9`로 설정하고 `earHoles=true`를 전환합니다 (36x36 px Retina 배킹 스토어에 렌더링된 18x18 pt 템플릿 이미지).
- 움직임은 약 ~1.0까지의 다리 흔들림과 작은 수평 흔들림을 사용합니다; 기존 대기 흔들림에 추가됩니다.

동작 참고사항

- 귀/작업 중에 대한 외부 CLI/브로커 전환이 없습니다; 실수로 인한 깜빡임을 방지하기 위해 앱 자체 신호에 내부적으로 유지합니다.
- TTL을 짧게 유지하세요 (<10초) 작업이 중단되어도 아이콘이 빠르게 기본 상태로 돌아가도록.
