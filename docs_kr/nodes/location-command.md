---
summary: "노드를 위한 위치 명령 (location.get), 권한 모드 및 백그라운드 동작"
read_when:
  - 위치 노드 지원 또는 권한 UI 추가 시
  - 백그라운드 위치 + 푸시 흐름 설계 시
title: "위치 명령"
---

# 위치 명령 (노드)

## 요약

- `location.get`은 노드 명령입니다 (`node.invoke`를 통해).
- 기본적으로 꺼져 있습니다.
- 설정은 셀렉터를 사용합니다: Off / While Using / Always.
- 별도 토글: Precise Location.

## 셀렉터인 이유 (단순 스위치가 아닌)

OS 권한은 다중 수준입니다. 앱 내에서 셀렉터를 노출할 수 있지만, OS가 여전히 실제 부여를 결정합니다.

- iOS/macOS: 사용자가 시스템 프롬프트/설정에서 **While Using** 또는 **Always**를 선택할 수 있습니다. 앱이 업그레이드를 요청할 수 있지만, OS가 설정을 요구할 수 있습니다.
- Android: 백그라운드 위치는 별도 권한; Android 10+에서는 종종 설정 흐름이 필요합니다.
- 정밀 위치는 별도 부여입니다 (iOS 14+ "Precise", Android "fine" vs "coarse").

UI의 셀렉터는 요청된 모드를 구동합니다; 실제 부여는 OS 설정에 있습니다.

## 설정 모델

노드 기기별:

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: bool

UI 동작:

- `whileUsing` 선택 시 포그라운드 권한을 요청합니다.
- `always` 선택 시 먼저 `whileUsing`을 확보한 다음 백그라운드를 요청합니다 (또는 필요시 사용자를 설정으로 안내).
- OS가 요청된 수준을 거부하면, 부여된 가장 높은 수준으로 되돌리고 상태를 표시합니다.

## 권한 매핑 (node.permissions)

선택사항. macOS 노드는 권한 맵을 통해 `location`을 보고합니다; iOS/Android는 이를 생략할 수 있습니다.

## 명령: `location.get`

`node.invoke`를 통해 호출됩니다.

매개변수 (제안):

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

응답 페이로드:

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

오류 (안정적 코드):

- `LOCATION_DISABLED`: 셀렉터가 꺼져 있음.
- `LOCATION_PERMISSION_REQUIRED`: 요청된 모드에 대한 권한 누락.
- `LOCATION_BACKGROUND_UNAVAILABLE`: 앱이 백그라운드이지만 While Using만 허용됨.
- `LOCATION_TIMEOUT`: 시간 내에 위치를 고정하지 못함.
- `LOCATION_UNAVAILABLE`: 시스템 실패 / 프로바이더 없음.

## 백그라운드 동작 (향후)

목표: 노드가 백그라운드일 때도 모델이 위치를 요청할 수 있지만, 다음 경우에만:

- 사용자가 **Always**를 선택함.
- OS가 백그라운드 위치를 부여함.
- 앱이 위치를 위해 백그라운드에서 실행 허용됨 (iOS 백그라운드 모드 / Android 포그라운드 서비스 또는 특별 허용).

푸시 트리거 흐름 (향후):

1. Gateway가 노드에 푸시를 전송 (사일런트 푸시 또는 FCM 데이터).
2. 노드가 잠시 깨어나 기기에서 위치를 요청.
3. 노드가 페이로드를 Gateway로 전달.

참고:

- iOS: Always 권한 + 백그라운드 위치 모드 필요. 사일런트 푸시가 제한될 수 있어 간헐적 실패 예상.
- Android: 백그라운드 위치에 포그라운드 서비스가 필요할 수 있음; 그렇지 않으면 거부 예상.

## 모델/도구 통합

- 도구 인터페이스: `nodes` 도구가 `location_get` 액션 추가 (노드 필요).
- CLI: `openclaw nodes location get --node <id>`.
- 에이전트 가이드라인: 사용자가 위치를 활성화하고 범위를 이해할 때만 호출.

## UX 문구 (제안)

- Off: "위치 공유가 비활성화되어 있습니다."
- While Using: "OpenClaw가 열려 있을 때만."
- Always: "백그라운드 위치를 허용합니다. 시스템 권한이 필요합니다."
- Precise: "정밀 GPS 위치를 사용합니다. 대략적인 위치를 공유하려면 끄세요."
