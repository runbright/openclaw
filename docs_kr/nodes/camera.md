---
summary: "에이전트 사용을 위한 카메라 캡처 (iOS 노드 + macOS 앱): 사진 (jpg) 및 짧은 비디오 클립 (mp4)"
read_when:
  - iOS 노드 또는 macOS에서 카메라 캡처 추가 또는 수정 시
  - 에이전트 접근 가능 MEDIA 임시 파일 워크플로우 확장 시
title: "카메라 캡처"
---

# 카메라 캡처 (에이전트)

OpenClaw는 에이전트 워크플로우를 위한 **카메라 캡처**를 지원합니다:

- **iOS 노드** (Gateway를 통해 페어링): `node.invoke`를 통한 **사진** (`jpg`) 또는 **짧은 비디오 클립** (`mp4`, 선택적 오디오 포함) 캡처.
- **Android 노드** (Gateway를 통해 페어링): `node.invoke`를 통한 **사진** (`jpg`) 또는 **짧은 비디오 클립** (`mp4`, 선택적 오디오 포함) 캡처.
- **macOS 앱** (Gateway를 통한 노드): `node.invoke`를 통한 **사진** (`jpg`) 또는 **짧은 비디오 클립** (`mp4`, 선택적 오디오 포함) 캡처.

모든 카메라 접근은 **사용자 제어 설정** 뒤에 게이트되어 있습니다.

## iOS 노드

### 사용자 설정 (기본 켜짐)

- iOS 설정 탭 -> **Camera** -> **Allow Camera** (`camera.enabled`)
  - 기본값: **켜짐** (키가 없으면 활성화로 처리).
  - 꺼진 경우: `camera.*` 명령이 `CAMERA_DISABLED`를 반환합니다.

### 명령 (Gateway `node.invoke`를 통해)

- `camera.list`
  - 응답 페이로드:
    - `devices`: `{ id, name, position, deviceType }` 배열

- `camera.snap`
  - 매개변수:
    - `facing`: `front|back` (기본값: `front`)
    - `maxWidth`: number (선택사항; iOS 노드에서 기본값 `1600`)
    - `quality`: `0..1` (선택사항; 기본값 `0.9`)
    - `format`: 현재 `jpg`
    - `delayMs`: number (선택사항; 기본값 `0`)
    - `deviceId`: string (선택사항; `camera.list`에서)
  - 응답 페이로드:
    - `format: "jpg"`
    - `base64: "<...>"`
    - `width`, `height`
  - 페이로드 가드: base64 페이로드를 5MB 미만으로 유지하기 위해 사진이 재압축됩니다.

- `camera.clip`
  - 매개변수:
    - `facing`: `front|back` (기본값: `front`)
    - `durationMs`: number (기본값 `3000`, 최대 `60000`으로 제한)
    - `includeAudio`: boolean (기본값 `true`)
    - `format`: 현재 `mp4`
    - `deviceId`: string (선택사항; `camera.list`에서)
  - 응답 페이로드:
    - `format: "mp4"`
    - `base64: "<...>"`
    - `durationMs`
    - `hasAudio`

### 포그라운드 요구사항

`canvas.*`와 마찬가지로, iOS 노드는 **포그라운드**에서만 `camera.*` 명령을 허용합니다. 백그라운드 호출은 `NODE_BACKGROUND_UNAVAILABLE`을 반환합니다.

### CLI 헬퍼 (임시 파일 + MEDIA)

첨부파일을 얻는 가장 쉬운 방법은 디코딩된 미디어를 임시 파일에 쓰고 `MEDIA:<path>`를 출력하는 CLI 헬퍼를 사용하는 것입니다.

예시:

```bash
openclaw nodes camera snap --node <id>               # 기본값: 전면 + 후면 모두 (2개 MEDIA 라인)
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

참고:

- `nodes camera snap`은 에이전트에게 양쪽 뷰를 제공하기 위해 기본적으로 **양쪽** 방향으로 설정됩니다.
- 출력 파일은 별도 래퍼를 만들지 않는 한 임시(OS 임시 디렉토리)입니다.

## Android 노드

### Android 사용자 설정 (기본 켜짐)

- Android 설정 시트 -> **Camera** -> **Allow Camera** (`camera.enabled`)
  - 기본값: **켜짐** (키가 없으면 활성화로 처리).
  - 꺼진 경우: `camera.*` 명령이 `CAMERA_DISABLED`를 반환합니다.

### 권한

- Android는 런타임 권한이 필요합니다:
  - `camera.snap`과 `camera.clip` 모두에 `CAMERA`.
  - `includeAudio=true`인 `camera.clip`에 `RECORD_AUDIO`.

권한이 없으면 앱이 가능한 경우 프롬프트를 표시합니다; 거부된 경우 `camera.*` 요청이
`*_PERMISSION_REQUIRED` 오류로 실패합니다.

### Android 포그라운드 요구사항

`canvas.*`와 마찬가지로, Android 노드는 **포그라운드**에서만 `camera.*` 명령을 허용합니다. 백그라운드 호출은 `NODE_BACKGROUND_UNAVAILABLE`을 반환합니다.

### 페이로드 가드

base64 페이로드를 5MB 미만으로 유지하기 위해 사진이 재압축됩니다.

## macOS 앱

### 사용자 설정 (기본 꺼짐)

macOS 컴패니언 앱은 체크박스를 노출합니다:

- **Settings -> General -> Allow Camera** (`openclaw.cameraEnabled`)
  - 기본값: **꺼짐**
  - 꺼진 경우: 카메라 요청이 "Camera disabled by user"를 반환합니다.

### CLI 헬퍼 (node invoke)

메인 `openclaw` CLI를 사용하여 macOS 노드에서 카메라 명령을 호출합니다.

예시:

```bash
openclaw nodes camera list --node <id>            # 카메라 ID 목록
openclaw nodes camera snap --node <id>            # MEDIA:<path> 출력
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # MEDIA:<path> 출력
openclaw nodes camera clip --node <id> --duration-ms 3000      # MEDIA:<path> 출력 (레거시 플래그)
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

참고:

- `openclaw nodes camera snap`은 재정의하지 않는 한 기본 `maxWidth=1600`입니다.
- macOS에서 `camera.snap`은 캡처 전 워밍업/노출 안정화 후 `delayMs` (기본값 2000ms)를 기다립니다.
- base64를 5MB 미만으로 유지하기 위해 사진 페이로드가 재압축됩니다.

## 안전 + 실용적 제한

- 카메라 및 마이크 접근은 일반적인 OS 권한 프롬프트를 트리거합니다 (Info.plist에 사용 문자열 필요).
- 비디오 클립은 과대한 노드 페이로드 (base64 오버헤드 + 메시지 제한)를 방지하기 위해 제한됩니다 (현재 `<= 60초`).

## macOS 화면 비디오 (OS 수준)

_화면_ 비디오 (카메라가 아닌)의 경우, macOS 컴패니언을 사용하세요:

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # MEDIA:<path> 출력
```

참고:

- macOS **Screen Recording** 권한 (TCC)이 필요합니다.
