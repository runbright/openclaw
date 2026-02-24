---
summary: "대화 모드: ElevenLabs TTS를 사용한 연속 음성 대화"
read_when:
  - macOS/iOS/Android에서 대화 모드 구현 시
  - 음성/TTS/인터럽트 동작 변경 시
title: "대화 모드"
---

# 대화 모드

대화 모드는 연속 음성 대화 루프입니다:

1. 음성 수신 대기
2. 트랜스크립트를 모델로 전송 (메인 세션, chat.send)
3. 응답 대기
4. ElevenLabs를 통해 음성 출력 (스트리밍 재생)

## 동작 (macOS)

- 대화 모드가 활성화된 동안 **항상 표시되는 오버레이**.
- **수신 중 -> 생각 중 -> 말하기** 단계 전환.
- **짧은 일시 중지** (침묵 창) 시 현재 트랜스크립트가 전송됩니다.
- 응답은 **WebChat에 기록됩니다** (타이핑과 동일).
- **음성 인터럽트** (기본 켜짐): 어시스턴트가 말하는 동안 사용자가 말하기 시작하면 재생을 중지하고 다음 프롬프트에 인터럽트 타임스탬프를 기록합니다.

## 응답에서의 음성 지시문

어시스턴트는 응답 앞에 음성을 제어하는 **단일 JSON 라인**을 접두사로 붙일 수 있습니다:

```json
{ "voice": "<voice-id>", "once": true }
```

규칙:

- 첫 번째 비어있지 않은 라인만.
- 알 수 없는 키는 무시됨.
- `once: true`는 현재 응답에만 적용됨.
- `once` 없이 음성은 대화 모드의 새 기본값이 됨.
- JSON 라인은 TTS 재생 전에 제거됨.

지원되는 키:

- `voice` / `voice_id` / `voiceId`
- `model` / `model_id` / `modelId`
- `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
- `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
- `once`

## 설정 (`~/.openclaw/openclaw.json`)

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

기본값:

- `interruptOnSpeech`: true
- `voiceId`: `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID`로 폴백 (또는 API 키가 있으면 첫 번째 ElevenLabs 음성)
- `modelId`: 미설정 시 기본값 `eleven_v3`
- `apiKey`: `ELEVENLABS_API_KEY`로 폴백 (또는 가능한 경우 gateway 셸 프로필)
- `outputFormat`: macOS/iOS에서 기본값 `pcm_44100`, Android에서 `pcm_24000` (MP3 스트리밍을 강제하려면 `mp3_*` 설정)

## macOS UI

- 메뉴 바 토글: **Talk**
- 설정 탭: **Talk Mode** 그룹 (음성 ID + 인터럽트 토글)
- 오버레이:
  - **수신 중**: 마이크 레벨에 따라 클라우드 펄스
  - **생각 중**: 가라앉는 애니메이션
  - **말하기**: 방사형 링
  - 클라우드 클릭: 말하기 중지
  - X 클릭: 대화 모드 종료

## 참고사항

- Speech + Microphone 권한이 필요합니다.
- 세션 키 `main`에 대해 `chat.send`를 사용합니다.
- TTS는 `ELEVENLABS_API_KEY`가 있는 ElevenLabs 스트리밍 API를 사용하며, 지연 시간 단축을 위해 macOS/iOS/Android에서 점진적 재생을 지원합니다.
- `eleven_v3`에 대한 `stability`는 `0.0`, `0.5` 또는 `1.0`으로 검증됩니다; 다른 모델은 `0..1`을 수용합니다.
- `latency_tier`는 설정 시 `0..4`로 검증됩니다.
- Android는 저지연 AudioTrack 스트리밍을 위해 `pcm_16000`, `pcm_22050`, `pcm_24000`, `pcm_44100` 출력 형식을 지원합니다.
