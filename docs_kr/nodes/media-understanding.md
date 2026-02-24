---
summary: "프로바이더 + CLI 폴백을 통한 수신 이미지/오디오/비디오 이해 (선택사항)"
read_when:
  - 미디어 이해 설계 또는 리팩토링 시
  - 수신 오디오/비디오/이미지 전처리 조정 시
title: "미디어 이해"
---

# 미디어 이해 (수신) -- 2026-01-17

OpenClaw는 응답 파이프라인이 실행되기 전에 **수신 미디어** (이미지/오디오/비디오)를 **요약**할 수 있습니다. 로컬 도구나 프로바이더 키가 사용 가능할 때 자동 감지하며, 비활성화하거나 커스터마이즈할 수 있습니다. 이해가 꺼져 있으면 모델은 여전히 원본 파일/URL을 평소대로 수신합니다.

## 목표

- 선택사항: 더 빠른 라우팅 + 더 나은 명령 파싱을 위해 수신 미디어를 짧은 텍스트로 사전 처리.
- 원본 미디어 전달을 모델에 보존 (항상).
- **프로바이더 API**와 **CLI 폴백** 지원.
- 순서가 있는 폴백으로 여러 모델 허용 (오류/크기/타임아웃).

## 상위 수준 동작

1. 수신 첨부파일 수집 (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. 각 활성화된 기능 (이미지/오디오/비디오)에 대해 정책에 따라 첨부파일 선택 (기본값: **첫 번째**).
3. 첫 번째 적격 모델 항목 선택 (크기 + 기능 + 인증).
4. 모델이 실패하거나 미디어가 너무 크면 **다음 항목으로 폴백**.
5. 성공 시:
   - `Body`가 `[Image]`, `[Audio]` 또는 `[Video]` 블록이 됩니다.
   - 오디오는 `{{Transcript}}`를 설정; 명령 파싱은 캡션 텍스트가 있으면 사용하고,
     없으면 트랜스크립트를 사용.
   - 캡션은 블록 내에 `User text:`로 보존됩니다.

이해가 실패하거나 비활성화되면 **원본 본문 + 첨부파일로 응답 흐름이 계속**됩니다.

## 설정 개요

`tools.media`는 **공유 모델** 및 기능별 재정의를 지원합니다:

- `tools.media.models`: 공유 모델 목록 (게이팅에 `capabilities` 사용).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - 기본값 (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - 프로바이더 재정의 (`baseUrl`, `headers`, `providerOptions`)
  - `tools.media.audio.providerOptions.deepgram`를 통한 Deepgram 오디오 옵션
  - 선택적 **기능별 `models` 목록** (공유 모델보다 우선)
  - `attachments` 정책 (`mode`, `maxAttachments`, `prefer`)
  - `scope` (채널/chatType/세션 키별 선택적 게이팅)
- `tools.media.concurrency`: 최대 동시 기능 실행 (기본값 **2**).

```json5
{
  tools: {
    media: {
      models: [
        /* 공유 목록 */
      ],
      image: {
        /* 선택적 재정의 */
      },
      audio: {
        /* 선택적 재정의 */
      },
      video: {
        /* 선택적 재정의 */
      },
    },
  },
}
```

### 모델 항목

각 `models[]` 항목은 **프로바이더** 또는 **CLI**일 수 있습니다:

```json5
{
  type: "provider", // 생략 시 기본값
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // 선택사항, 멀티모달 항목에 사용
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

CLI 템플릿에서 사용할 수 있는 것:

- `{{MediaDir}}` (미디어 파일이 포함된 디렉토리)
- `{{OutputDir}}` (이 실행을 위해 생성된 스크래치 디렉토리)
- `{{OutputBase}}` (확장자 없는 스크래치 파일 기본 경로)

## 기본값 및 제한

권장 기본값:

- `maxChars`: 이미지/비디오 **500** (짧고 명령 친화적)
- `maxChars`: 오디오 **미설정** (제한을 설정하지 않는 한 전체 트랜스크립트)
- `maxBytes`:
  - 이미지: **10MB**
  - 오디오: **20MB**
  - 비디오: **50MB**

규칙:

- 미디어가 `maxBytes`를 초과하면 해당 모델이 건너뛰어지고 **다음 모델이 시도됩니다**.
- 모델이 `maxChars`보다 많이 반환하면 출력이 잘립니다.
- `prompt`는 기본적으로 간단한 "Describe the {media}." 및 `maxChars` 안내입니다 (이미지/비디오만).
- `<capability>.enabled: true`이지만 모델이 구성되지 않으면, OpenClaw는
  프로바이더가 해당 기능을 지원할 때 **활성 응답 모델**을 시도합니다.

### 미디어 이해 자동 감지 (기본값)

`tools.media.<capability>.enabled`가 `false`로 설정되지 **않고** 모델을 구성하지 않은 경우,
OpenClaw는 다음 순서로 자동 감지하며 **첫 번째 작동하는 옵션에서 중지**합니다:

1. **로컬 CLI** (오디오만; 설치된 경우)
   - `sherpa-onnx-offline` (`SHERPA_ONNX_MODEL_DIR`에 encoder/decoder/joiner/tokens 필요)
   - `whisper-cli` (`whisper-cpp`; `WHISPER_CPP_MODEL` 또는 번들된 tiny 모델 사용)
   - `whisper` (Python CLI; 모델 자동 다운로드)
2. **Gemini CLI** (`gemini`) `read_many_files` 사용
3. **프로바이더 키**
   - 오디오: OpenAI -> Groq -> Deepgram -> Google
   - 이미지: OpenAI -> Anthropic -> Google -> MiniMax
   - 비디오: Google

자동 감지를 비활성화하려면:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

참고: 바이너리 감지는 macOS/Linux/Windows 전반에서 최선 노력입니다; CLI가 `PATH`에 있는지 확인하거나 (`~`를 확장함), 전체 명령 경로가 포함된 명시적 CLI 모델을 설정하세요.

## 기능 (선택사항)

`capabilities`를 설정하면 해당 항목은 해당 미디어 타입에서만 실행됩니다. 공유 목록의 경우
OpenClaw가 기본값을 추론할 수 있습니다:

- `openai`, `anthropic`, `minimax`: **image**
- `google` (Gemini API): **image + audio + video**
- `groq`: **audio**
- `deepgram`: **audio**

CLI 항목의 경우 의외의 매칭을 방지하기 위해 **`capabilities`를 명시적으로 설정**하세요.
`capabilities`를 생략하면 해당 항목은 나타나는 목록에 대해 적격합니다.

## 프로바이더 지원 매트릭스 (OpenClaw 통합)

| 기능 | 프로바이더 통합 | 참고 |
| ---------- | ------------------------------------------------ | --------------------------------------------------------- |
| 이미지 | OpenAI / Anthropic / Google / `pi-ai`를 통한 기타 | 레지스트리의 이미지 지원 모델이면 작동합니다. |
| 오디오 | OpenAI, Groq, Deepgram, Google, Mistral | 프로바이더 트랜스크립션 (Whisper/Deepgram/Gemini/Voxtral). |
| 비디오 | Google (Gemini API) | 프로바이더 비디오 이해. |

## 추천 프로바이더

**이미지**

- 활성 모델이 이미지를 지원하면 그것을 선호하세요.
- 좋은 기본값: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**오디오**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo`, `deepgram/nova-3`, 또는 `mistral/voxtral-mini-latest`.
- CLI 폴백: `whisper-cli` (whisper-cpp) 또는 `whisper`.
- Deepgram 설정: [Deepgram (오디오 트랜스크립션)](/providers/deepgram).

**비디오**

- `google/gemini-3-flash-preview` (빠름), `google/gemini-3-pro-preview` (풍부).
- CLI 폴백: `gemini` CLI (비디오/오디오에 `read_file` 지원).

## 첨부파일 정책

기능별 `attachments`는 어떤 첨부파일이 처리되는지 제어합니다:

- `mode`: `first` (기본값) 또는 `all`
- `maxAttachments`: 처리되는 수 제한 (기본값 **1**)
- `prefer`: `first`, `last`, `path`, `url`

`mode: "all"`인 경우, 출력은 `[Image 1/2]`, `[Audio 2/2]` 등으로 레이블됩니다.

## 설정 예시

### 1) 공유 모델 목록 + 재정의

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) 오디오 + 비디오만 (이미지 꺼짐)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) 선택적 이미지 이해

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) 멀티모달 단일 항목 (명시적 capabilities)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## 상태 출력

미디어 이해가 실행되면 `/status`에 짧은 요약 라인이 포함됩니다:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

해당하는 경우 기능별 결과와 선택된 프로바이더/모델을 표시합니다.

## 참고사항

- 이해는 **최선 노력**입니다. 오류가 응답을 차단하지 않습니다.
- 이해가 비활성화되어도 첨부파일은 여전히 모델에 전달됩니다.
- 이해가 실행되는 곳을 제한하려면 `scope`를 사용하세요 (예: DM만).

## 관련 문서

- [설정](/gateway/configuration)
- [이미지 및 미디어 지원](/nodes/images)
