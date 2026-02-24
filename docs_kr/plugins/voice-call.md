---
summary: "음성 통화 플러그인: Twilio/Telnyx/Plivo를 통한 발신 + 수신 통화 (플러그인 설치 + 설정 + CLI)"
read_when:
  - OpenClaw에서 발신 음성 통화를 하려는 경우
  - 음성 통화 플러그인을 구성하거나 개발하는 경우
title: "음성 통화 플러그인"
---

# 음성 통화 (플러그인)

플러그인을 통한 OpenClaw 음성 통화. 발신 알림 및 수신 정책을 갖춘
다중 턴 대화를 지원합니다.

현재 프로바이더:

- `twilio` (Programmable Voice + Media Streams)
- `telnyx` (Call Control v2)
- `plivo` (Voice API + XML transfer + GetInput speech)
- `mock` (개발용/네트워크 없음)

간단한 모델:

- 플러그인 설치
- Gateway 재시작
- `plugins.entries.voice-call.config` 아래에 구성
- `openclaw voicecall ...` 또는 `voice_call` 도구 사용

## 실행 위치 (로컬 vs 원격)

음성 통화 플러그인은 **Gateway 프로세스 내부에서** 실행됩니다.

원격 Gateway를 사용하는 경우, **Gateway를 실행하는 머신**에 플러그인을 설치/구성한 다음 Gateway를 재시작하여 로드하세요.

## 설치

### 옵션 A: npm에서 설치 (권장)

```bash
openclaw plugins install @openclaw/voice-call
```

이후 Gateway를 재시작하세요.

### 옵션 B: 로컬 폴더에서 설치 (개발, 복사 없음)

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

이후 Gateway를 재시작하세요.

## 설정

`plugins.entries.voice-call.config` 아래에 설정합니다:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // 또는 "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Telnyx Mission Control Portal의 Telnyx 웹훅 공개 키
            // (Base64 문자열; TELNYX_PUBLIC_KEY로도 설정 가능).
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // 웹훅 서버
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // 웹훅 보안 (터널/프록시에 권장)
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // 공개 노출 (하나 선택)
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

참고:

- Twilio/Telnyx는 **공개적으로 접근 가능한** 웹훅 URL이 필요합니다.
- Plivo는 **공개적으로 접근 가능한** 웹훅 URL이 필요합니다.
- `mock`은 로컬 개발 프로바이더입니다 (네트워크 호출 없음).
- Telnyx는 `skipSignatureVerification`이 true가 아닌 한 `telnyx.publicKey` (또는 `TELNYX_PUBLIC_KEY`)가 필요합니다.
- `skipSignatureVerification`은 로컬 테스트 전용입니다.
- ngrok 무료 티어를 사용하는 경우, `publicUrl`을 정확한 ngrok URL로 설정하세요; 서명 검증은 항상 적용됩니다.
- `tunnel.allowNgrokFreeTierLoopbackBypass: true`는 `tunnel.provider="ngrok"`이고 `serve.bind`가 루프백(ngrok 로컬 에이전트)인 경우에**만** 유효하지 않은 서명이 있는 Twilio 웹훅을 허용합니다. 로컬 개발 전용으로 사용하세요.
- ngrok 무료 티어 URL은 변경되거나 인터스티셜 동작이 추가될 수 있습니다; `publicUrl`이 드리프트하면 Twilio 서명이 실패합니다. 프로덕션에서는 안정적인 도메인이나 Tailscale funnel을 선호하세요.
- 스트리밍 보안 기본값:
  - `streaming.preStartTimeoutMs`는 유효한 `start` 프레임을 보내지 않는 소켓을 닫습니다.
  - `streaming.maxPendingConnections`는 인증되지 않은 사전 시작 소켓의 총 수를 제한합니다.
  - `streaming.maxPendingConnectionsPerIp`는 소스 IP별 인증되지 않은 사전 시작 소켓을 제한합니다.
  - `streaming.maxConnections`는 총 열린 미디어 스트림 소켓 (대기 + 활성)을 제한합니다.

## 비활성 통화 리퍼

터미널 웹훅을 수신하지 못하는 통화를 종료하려면 `staleCallReaperSeconds`를 사용하세요
(예: 완료되지 않는 알림 모드 통화). 기본값은 `0` (비활성화)입니다.

권장 범위:

- **프로덕션:** 알림 스타일 흐름에 `120`-`300`초.
- 정상 통화가 완료될 수 있도록 이 값을 **`maxDurationSeconds`보다 높게** 유지하세요. 좋은 시작점은 `maxDurationSeconds + 30-60`초입니다.

예시:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## 웹훅 보안

프록시나 터널이 Gateway 앞에 있을 때, 플러그인은 서명 검증을 위해 공개 URL을 재구성합니다.
이 옵션들은 어떤 전달 헤더를 신뢰할지 제어합니다.

`webhookSecurity.allowedHosts`는 전달 헤더의 호스트를 허용 목록에 추가합니다.

`webhookSecurity.trustForwardingHeaders`는 허용 목록 없이 전달 헤더를 신뢰합니다.

`webhookSecurity.trustedProxyIPs`는 요청 원격 IP가 목록과 일치할 때만 전달 헤더를 신뢰합니다.

Twilio 및 Plivo에 대해 웹훅 리플레이 보호가 활성화되어 있습니다. 리플레이된 유효한 웹훅
요청은 확인되지만 부작용에 대해서는 건너뛰어집니다.

Twilio 대화 턴은 `<Gather>` 콜백에 턴별 토큰을 포함하므로,
오래된/리플레이된 음성 콜백이 최신 대기 중인 트랜스크립트 턴을 충족시킬 수 없습니다.

안정적인 공개 호스트 예시:

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## 통화를 위한 TTS

음성 통화는 통화에서 스트리밍 음성을 위해 코어 `messages.tts` 설정 (OpenAI 또는 ElevenLabs)을
사용합니다. 플러그인 설정 아래에서 **동일한 형태**로 재정의할 수 있습니다 --
`messages.tts`와 딥 머지됩니다.

```json5
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

참고:

- **Edge TTS는 음성 통화에서 무시됩니다** (전화 오디오에는 PCM이 필요; Edge 출력은 불안정).
- Twilio 미디어 스트리밍이 활성화되면 코어 TTS가 사용됩니다; 그렇지 않으면 통화가 프로바이더 네이티브 음성으로 폴백합니다.

### 추가 예시

코어 TTS만 사용 (재정의 없음):

```json5
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

통화에서만 ElevenLabs로 재정의 (다른 곳에서는 코어 기본값 유지):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

통화에서만 OpenAI 모델 재정의 (딥 머지 예시):

```json5
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## 수신 통화

수신 정책은 기본적으로 `disabled`입니다. 수신 통화를 활성화하려면:

```json5
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "Hello! How can I help?",
}
```

자동 응답은 에이전트 시스템을 사용합니다. 다음으로 조정하세요:

- `responseModel`
- `responseSystemPrompt`
- `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "Hello from OpenClaw"
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall speak --call-id <id> --message "One moment"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## 에이전트 도구

도구 이름: `voice_call`

액션:

- `initiate_call` (message, to?, mode?)
- `continue_call` (callId, message)
- `speak_to_user` (callId, message)
- `end_call` (callId)
- `get_status` (callId)

이 저장소에는 `skills/voice-call/SKILL.md`에 매칭 스킬 문서가 있습니다.

## Gateway RPC

- `voicecall.initiate` (`to?`, `message`, `mode?`)
- `voicecall.continue` (`callId`, `message`)
- `voicecall.speak` (`callId`, `message`)
- `voicecall.end` (`callId`)
- `voicecall.status` (`callId`)
