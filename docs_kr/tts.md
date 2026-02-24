---
summary: "OpenClaw의 텍스트 음성 변환(TTS) 기능 설정 및 사용 가이드"
read_when:
  - 답변에 TTS 기능을 활성화하고 싶을 때
  - TTS 제공자나 제한 사항을 설정할 때
  - /tts 명령어를 사용하고 싶을 때
title: "텍스트 음성 변환 (TTS)"
---

# 텍스트 음성 변환 (TTS)

OpenClaw는 ElevenLabs, OpenAI 또는 Edge TTS를 사용하여 텍스트 답변을 오디오로 변환할 수 있습니다. Telegram에서는 동그란 '음성 메시지' 형태로 전송됩니다.

## 지원 서비스

- **ElevenLabs**: 가장 고품질의 목소리를 제공합니다.
- **OpenAI**: 안정적이고 요약 기능과도 잘 연동됩니다.
- **Edge TTS**: Microsoft Edge의 신경망 TTS 서비스를 사용하며, **API 키가 필요 없습니다**. (무료 사용 가능)

## 서비스 설정

API 키가 있다면 환경 변수(`ELEVENLABS_API_KEY`, `OPENAI_API_KEY`)에 등록하세요. 키가 없는 경우 OpenClaw는 기본적으로 **Edge TTS**를 사용합니다.

## 기본 활성화 여부

아니요. 자동 TTS는 기본적으로 **꺼져(off)** 있습니다. 활성화하려면 설정 파일에서 `messages.tts.auto`를 `"always"`로 설정하거나, 채팅창에서 `/tts on` 명령어를 입력하세요.

## 설정 예시 (openclaw.json)

### 기본 활성화 (Edge TTS 사용)
```json5
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "edge"
    }
  }
}
```

### OpenAI를 주 엔진으로 사용
```json5
{
  "messages": {
    "tts": {
      "auto": "always",
      "provider": "openai",
      "openai": {
        "voice": "alloy" // 샘플 목소리: alloy, echo, fable, onyx, nova, shimmer
      }
    }
  }
}
```

## TTS 명령어 사용법

봇에게 다음과 같은 명령어를 보내 설정을 변경할 수 있습니다:

- `/tts on` (또는 `/tts always`): 모든 답변을 음성으로 바꿉니다.
- `/tts off`: 음성 기능을 끕니다.
- `/tts inbound`: 내가 음성 메시지를 보냈을 때만 에이전트도 음성으로 답변합니다.
- `/tts tagged`: 답변에 `[[tts]]` 태그가 포함된 경우에만 음성으로 변환합니다.
- `/tts status`: 현재 TTS 상태를 확인합니다.
- `/tts audio 안녕하세요`: 입력한 텍스트를 즉시 음성으로 들려줍니다.

## 주요 참고 사항

- **긴 답변 처리**: 답변이 너무 길면 자동으로 요약하여 읽어줍니다. 요약 기능은 `summaryModel` 설정을 통해 제어할 수 있습니다.
- **오디오 포맷**: Telegram에서는 음성 메시지(Voice Note) 전송을 위해 전용 규격(Opus)을 사용하며, 다른 채널은 일반적인 MP3 형식을 사용합니다.
- **모델 직접 제어**: 고급 사용자는 에이전트의 답변 내에 `[[tts:speed=1.2]]` 같은 태그를 넣어 목소리 속도나 톤을 실시간으로 조절할 수 있게 할 수 있습니다.

---
상세한 API 레퍼런스나 고급 설정법은 [영문 원본 문서](/tts)를 참고해 주시기 바랍니다.
---
