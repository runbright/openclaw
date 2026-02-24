---
summary: "수신 오디오/음성 메모가 다운로드, 변환, 응답에 주입되는 방법"
read_when:
  - 오디오 트랜스크립션 또는 미디어 처리 변경 시
title: "오디오 및 음성 메모"
---

# 오디오 / 음성 메모 -- 2026-01-17

## 작동하는 기능

- **미디어 이해 (오디오)**: 오디오 이해가 활성화되어 있으면 (또는 자동 감지), OpenClaw는:
  1. 첫 번째 오디오 첨부파일 (로컬 경로 또는 URL)을 찾고 필요시 다운로드합니다.
  2. 각 모델 항목에 전송하기 전에 `maxBytes`를 적용합니다.
  3. 순서대로 첫 번째 적격 모델 항목 (프로바이더 또는 CLI)을 실행합니다.
  4. 실패하거나 건너뛰면 (크기/타임아웃), 다음 항목을 시도합니다.
  5. 성공하면 `Body`를 `[Audio]` 블록으로 교체하고 `{{Transcript}}`를 설정합니다.
- **명령 파싱**: 트랜스크립션이 성공하면 `CommandBody`/`RawBody`가 트랜스크립트로 설정되어 슬래시 명령이 여전히 작동합니다.
- **상세 로깅**: `--verbose`에서 트랜스크립션이 실행되고 본문을 교체할 때 로그를 기록합니다.

## 자동 감지 (기본값)

모델을 **구성하지 않고** `tools.media.audio.enabled`가 `false`로 설정되지 **않은** 경우,
OpenClaw는 다음 순서로 자동 감지하며 첫 번째 작동하는 옵션에서 중지합니다:

1. **로컬 CLI** (설치된 경우)
   - `sherpa-onnx-offline` (`SHERPA_ONNX_MODEL_DIR`에 encoder/decoder/joiner/tokens 필요)
   - `whisper-cli` (`whisper-cpp`에서; `WHISPER_CPP_MODEL` 또는 번들된 tiny 모델 사용)
   - `whisper` (Python CLI; 모델 자동 다운로드)
2. **Gemini CLI** (`gemini`) `read_many_files` 사용
3. **프로바이더 키** (OpenAI -> Groq -> Deepgram -> Google)

자동 감지를 비활성화하려면 `tools.media.audio.enabled: false`로 설정하세요.
커스터마이즈하려면 `tools.media.audio.models`를 설정하세요.
참고: 바이너리 감지는 macOS/Linux/Windows 전반에서 최선 노력입니다; CLI가 `PATH`에 있는지 확인하거나 (`~`를 확장함), 전체 명령 경로가 포함된 명시적 CLI 모델을 설정하세요.

## 설정 예시

### 프로바이더 + CLI 폴백 (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### 스코프 게이팅이 있는 프로바이더 전용

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### 프로바이더 전용 (Deepgram)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

### 프로바이더 전용 (Mistral Voxtral)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "mistral", model: "voxtral-mini-latest" }],
      },
    },
  },
}
```

## 참고사항 및 제한

- 프로바이더 인증은 표준 모델 인증 순서를 따릅니다 (인증 프로필, 환경 변수, `models.providers.*.apiKey`).
- `provider: "deepgram"` 사용 시 Deepgram은 `DEEPGRAM_API_KEY`를 인식합니다.
- Deepgram 설정 세부사항: [Deepgram (오디오 트랜스크립션)](/providers/deepgram).
- Mistral 설정 세부사항: [Mistral](/providers/mistral).
- 오디오 프로바이더는 `tools.media.audio`를 통해 `baseUrl`, `headers`, `providerOptions`를 재정의할 수 있습니다.
- 기본 크기 제한은 20MB (`tools.media.audio.maxBytes`)입니다. 초과 오디오는 해당 모델에서 건너뛰어지고 다음 항목이 시도됩니다.
- 오디오의 기본 `maxChars`는 **미설정** (전체 트랜스크립트)입니다. 출력을 자르려면 `tools.media.audio.maxChars` 또는 항목별 `maxChars`를 설정하세요.
- OpenAI 자동 기본값은 `gpt-4o-mini-transcribe`; 더 높은 정확도를 위해 `model: "gpt-4o-transcribe"`를 설정하세요.
- 여러 음성 메모를 처리하려면 `tools.media.audio.attachments`를 사용하세요 (`mode: "all"` + `maxAttachments`).
- 트랜스크립트는 템플릿에서 `{{Transcript}}`로 사용 가능합니다.
- CLI stdout은 제한됩니다 (5MB); CLI 출력을 간결하게 유지하세요.

## 그룹에서의 멘션 감지

그룹 채팅에 `requireMention: true`가 설정된 경우, OpenClaw는 이제 멘션을 확인하기 **전에** 오디오를 변환합니다. 이를 통해 멘션이 포함된 음성 메모도 처리할 수 있습니다.

**작동 방식:**

1. 음성 메시지에 텍스트 본문이 없고 그룹에서 멘션이 필요한 경우, OpenClaw는 "사전 검사" 트랜스크립션을 수행합니다.
2. 트랜스크립트에서 멘션 패턴 (예: `@BotName`, 이모지 트리거)을 확인합니다.
3. 멘션이 발견되면 메시지는 전체 응답 파이프라인을 통해 진행됩니다.
4. 트랜스크립트가 멘션 감지에 사용되어 음성 메모가 멘션 게이트를 통과할 수 있습니다.

**폴백 동작:**

- 사전 검사 중 트랜스크립션이 실패하면 (타임아웃, API 오류 등), 메시지는 텍스트 전용 멘션 감지를 기반으로 처리됩니다.
- 이렇게 하면 혼합 메시지 (텍스트 + 오디오)가 잘못 삭제되지 않습니다.

**예시:** 사용자가 `requireMention: true`인 Telegram 그룹에서 "안녕 @Claude, 날씨 어때?"라고 말하는 음성 메모를 보냅니다. 음성 메모가 변환되고, 멘션이 감지되고, 에이전트가 응답합니다.

## 주의사항

- 스코프 규칙은 첫 번째 일치 우선입니다. `chatType`은 `direct`, `group` 또는 `room`으로 정규화됩니다.
- CLI가 종료 코드 0으로 종료하고 일반 텍스트를 출력하는지 확인하세요; JSON은 `jq -r .text`를 통해 처리해야 합니다.
- 응답 큐 차단을 방지하기 위해 타임아웃을 합리적으로 유지하세요 (`timeoutSeconds`, 기본값 60초).
- 사전 검사 트랜스크립션은 멘션 감지를 위해 **첫 번째** 오디오 첨부파일만 처리합니다. 추가 오디오는 메인 미디어 이해 단계에서 처리됩니다.
