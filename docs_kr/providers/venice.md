---
summary: "OpenClaw에서 Venice AI 프라이버시 중심 모델 사용하기"
read_when:
  - OpenClaw에서 프라이버시 중심 추론을 원할 때
  - Venice AI 설정 안내가 필요할 때
title: "Venice AI"
---

# Venice AI (Venice 하이라이트)

**Venice**는 프라이버시 우선 추론과 독점 모델에 대한 선택적 익명화 접근을 위한 하이라이트 Venice 설정입니다.

Venice AI는 무검열 모델 지원과 익명화 프록시를 통한 주요 독점 모델 접근을 갖춘 프라이버시 중심 AI 추론을 제공합니다. 모든 추론은 기본적으로 비공개입니다 -- 데이터를 학습에 사용하지 않으며 로깅하지 않습니다.

## OpenClaw에서 Venice를 사용하는 이유

- 오픈소스 모델을 위한 **비공개 추론** (로깅 없음).
- 필요할 때 **무검열 모델** 사용.
- 품질이 중요할 때 독점 모델(Opus/GPT/Gemini)에 대한 **익명화 접근**.
- OpenAI 호환 `/v1` 엔드포인트.

## 프라이버시 모드

Venice는 두 가지 프라이버시 수준을 제공합니다 -- 모델 선택 시 이를 이해하는 것이 중요합니다:

| 모드           | 설명                                                                                                          | 모델                                           |
| -------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **비공개**     | 완전히 비공개. 프롬프트/응답이 **저장되거나 기록되지 않음**. 임시적.                                          | Llama, Qwen, DeepSeek, Venice Uncensored 등    |
| **익명화**     | Venice를 통해 프록시되며 메타데이터가 제거됨. 기반 프로바이더(OpenAI, Anthropic)는 익명화된 요청을 봄.          | Claude, GPT, Gemini, Grok, Kimi, MiniMax       |

## 기능

- **프라이버시 중심**: "비공개" (완전 비공개)와 "익명화" (프록시) 모드 중 선택
- **무검열 모델**: 콘텐츠 제한이 없는 모델에 접근
- **주요 모델 접근**: Venice의 익명화 프록시를 통해 Claude, GPT-5.2, Gemini, Grok 사용
- **OpenAI 호환 API**: 쉬운 통합을 위한 표준 `/v1` 엔드포인트
- **스트리밍**: 모든 모델에서 지원
- **함수 호출**: 일부 모델에서 지원 (모델 기능 확인)
- **비전**: 비전 기능이 있는 모델에서 지원
- **엄격한 속도 제한 없음**: 극단적인 사용 시 공정 사용 제한이 적용될 수 있음

## 설정

### 1. API 키 발급

1. [venice.ai](https://venice.ai)에서 가입합니다
2. **Settings -> API Keys -> Create new key**로 이동합니다
3. API 키를 복사합니다 (형식: `vapi_xxxxxxxxxxxx`)

### 2. OpenClaw 구성

**옵션 A: 환경 변수**

```bash
export VENICE_API_KEY="vapi_xxxxxxxxxxxx"
```

**옵션 B: 대화형 설정 (권장)**

```bash
openclaw onboard --auth-choice venice-api-key
```

이 명령은 다음을 수행합니다:

1. API 키를 요청합니다 (또는 기존 `VENICE_API_KEY` 사용)
2. 사용 가능한 모든 Venice 모델을 표시합니다
3. 기본 모델을 선택할 수 있게 합니다
4. 프로바이더를 자동으로 구성합니다

**옵션 C: 비대화형**

```bash
openclaw onboard --non-interactive \
  --auth-choice venice-api-key \
  --venice-api-key "vapi_xxxxxxxxxxxx"
```

### 3. 설정 확인

```bash
openclaw agent --model venice/llama-3.3-70b --message "Hello, are you working?"
```

## 모델 선택

설정 후 OpenClaw은 사용 가능한 모든 Venice 모델을 표시합니다. 필요에 따라 선택하세요:

- **기본값 (추천)**: `venice/llama-3.3-70b` - 비공개, 균형 잡힌 성능.
- **최고 품질**: `venice/claude-opus-45` - 어려운 작업에 적합 (Opus가 여전히 가장 강력).
- **프라이버시**: 완전 비공개 추론을 위해 "비공개" 모델 선택.
- **기능**: Venice 프록시를 통해 Claude, GPT, Gemini에 접근하려면 "익명화" 모델 선택.

기본 모델은 언제든 변경 가능합니다:

```bash
openclaw models set venice/claude-opus-45
openclaw models set venice/llama-3.3-70b
```

사용 가능한 모든 모델 목록:

```bash
openclaw models list | grep venice
```

## `openclaw configure`를 통한 구성

1. `openclaw configure`를 실행합니다
2. **Model/auth**를 선택합니다
3. **Venice AI**를 선택합니다

## 어떤 모델을 사용해야 할까요?

| 사용 사례                    | 추천 모델                         | 이유                                      |
| ---------------------------- | -------------------------------- | ----------------------------------------- |
| **일반 채팅**                | `llama-3.3-70b`                  | 전반적으로 우수, 완전 비공개              |
| **최고 품질**                | `claude-opus-45`                 | 어려운 작업에 Opus가 가장 강력            |
| **프라이버시 + Claude 품질** | `claude-opus-45`                 | 익명화 프록시를 통한 최고 추론            |
| **코딩**                     | `qwen3-coder-480b-a35b-instruct` | 코드 최적화, 262k 컨텍스트               |
| **비전 작업**                | `qwen3-vl-235b-a22b`             | 최고의 비공개 비전 모델                   |
| **무검열**                   | `venice-uncensored`              | 콘텐츠 제한 없음                          |
| **빠르고 저렴한**            | `qwen3-4b`                       | 가볍지만 유능                             |
| **복잡한 추론**              | `deepseek-v3.2`                  | 강력한 추론, 비공개                       |

## 사용 가능한 모델 (총 25개)

### 비공개 모델 (15개) -- 완전 비공개, 로깅 없음

| 모델 ID                          | 이름                    | 컨텍스트 (토큰)  | 기능                    |
| -------------------------------- | ----------------------- | ---------------- | ----------------------- |
| `llama-3.3-70b`                  | Llama 3.3 70B           | 131k             | 범용                    |
| `llama-3.2-3b`                   | Llama 3.2 3B            | 131k             | 빠름, 경량              |
| `hermes-3-llama-3.1-405b`        | Hermes 3 Llama 3.1 405B | 131k             | 복잡한 작업             |
| `qwen3-235b-a22b-thinking-2507`  | Qwen3 235B Thinking     | 131k             | 추론                    |
| `qwen3-235b-a22b-instruct-2507`  | Qwen3 235B Instruct     | 131k             | 범용                    |
| `qwen3-coder-480b-a35b-instruct` | Qwen3 Coder 480B        | 262k             | 코드                    |
| `qwen3-next-80b`                 | Qwen3 Next 80B          | 262k             | 범용                    |
| `qwen3-vl-235b-a22b`             | Qwen3 VL 235B           | 262k             | 비전                    |
| `qwen3-4b`                       | Venice Small (Qwen3 4B) | 32k              | 빠름, 추론              |
| `deepseek-v3.2`                  | DeepSeek V3.2           | 163k             | 추론                    |
| `venice-uncensored`              | Venice Uncensored       | 32k              | 무검열                  |
| `mistral-31-24b`                 | Venice Medium (Mistral) | 131k             | 비전                    |
| `google-gemma-3-27b-it`          | Gemma 3 27B Instruct    | 202k             | 비전                    |
| `openai-gpt-oss-120b`            | OpenAI GPT OSS 120B     | 131k             | 범용                    |
| `zai-org-glm-4.7`                | GLM 4.7                 | 202k             | 추론, 다국어            |

### 익명화 모델 (10개) -- Venice 프록시 경유

| 모델 ID                  | 원본 모델         | 컨텍스트 (토큰)  | 기능              |
| ------------------------ | ----------------- | ---------------- | ----------------- |
| `claude-opus-45`         | Claude Opus 4.5   | 202k             | 추론, 비전        |
| `claude-sonnet-45`       | Claude Sonnet 4.5 | 202k             | 추론, 비전        |
| `openai-gpt-52`          | GPT-5.2           | 262k             | 추론              |
| `openai-gpt-52-codex`    | GPT-5.2 Codex     | 262k             | 추론, 비전        |
| `gemini-3-pro-preview`   | Gemini 3 Pro      | 202k             | 추론, 비전        |
| `gemini-3-flash-preview` | Gemini 3 Flash    | 262k             | 추론, 비전        |
| `grok-41-fast`           | Grok 4.1 Fast     | 262k             | 추론, 비전        |
| `grok-code-fast-1`       | Grok Code Fast 1  | 262k             | 추론, 코드        |
| `kimi-k2-thinking`       | Kimi K2 Thinking  | 262k             | 추론              |
| `minimax-m21`            | MiniMax M2.1      | 202k             | 추론              |

## 모델 검색

`VENICE_API_KEY`가 설정되면 OpenClaw이 Venice API에서 자동으로 모델을 검색합니다. API에 접근할 수 없는 경우 정적 카탈로그로 대체됩니다.

`/models` 엔드포인트는 공개적이며 (목록 조회에 인증 불필요), 추론에는 유효한 API 키가 필요합니다.

## 스트리밍 및 도구 지원

| 기능                 | 지원                                                    |
| -------------------- | ------------------------------------------------------- |
| **스트리밍**         | 모든 모델                                               |
| **함수 호출**        | 대부분의 모델 (API에서 `supportsFunctionCalling` 확인)  |
| **비전/이미지**      | "비전" 기능으로 표시된 모델                              |
| **JSON 모드**        | `response_format`을 통해 지원                           |

## 가격

Venice는 크레딧 기반 시스템을 사용합니다. 현재 요금은 [venice.ai/pricing](https://venice.ai/pricing)에서 확인하세요:

- **비공개 모델**: 일반적으로 더 저렴
- **익명화 모델**: 직접 API 가격과 유사 + 소액의 Venice 수수료

## 비교: Venice vs 직접 API

| 측면         | Venice (익명화)                 | 직접 API            |
| ------------ | ----------------------------- | ------------------- |
| **프라이버시** | 메타데이터 제거, 익명화       | 계정 연결           |
| **지연 시간** | +10-50ms (프록시)             | 직접 연결           |
| **기능**     | 대부분의 기능 지원             | 전체 기능           |
| **과금**     | Venice 크레딧                  | 프로바이더 과금     |

## 사용 예시

```bash
# 기본 비공개 모델 사용
openclaw agent --model venice/llama-3.3-70b --message "Quick health check"

# Venice를 통한 Claude 사용 (익명화)
openclaw agent --model venice/claude-opus-45 --message "Summarize this task"

# 무검열 모델 사용
openclaw agent --model venice/venice-uncensored --message "Draft options"

# 비전 모델로 이미지 사용
openclaw agent --model venice/qwen3-vl-235b-a22b --message "Review attached image"

# 코딩 모델 사용
openclaw agent --model venice/qwen3-coder-480b-a35b-instruct --message "Refactor this function"
```

## 문제 해결

### API 키가 인식되지 않음

```bash
echo $VENICE_API_KEY
openclaw models list | grep venice
```

키가 `vapi_`로 시작하는지 확인하세요.

### 모델을 사용할 수 없음

Venice 모델 카탈로그는 동적으로 업데이트됩니다. `openclaw models list`를 실행하여 현재 사용 가능한 모델을 확인하세요. 일부 모델은 일시적으로 오프라인일 수 있습니다.

### 연결 문제

Venice API는 `https://api.venice.ai/api/v1`에 있습니다. 네트워크에서 HTTPS 연결이 허용되는지 확인하세요.

## 설정 파일 예시

```json5
{
  env: { VENICE_API_KEY: "vapi_..." },
  agents: { defaults: { model: { primary: "venice/llama-3.3-70b" } } },
  models: {
    mode: "merge",
    providers: {
      venice: {
        baseUrl: "https://api.venice.ai/api/v1",
        apiKey: "${VENICE_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.3-70b",
            name: "Llama 3.3 70B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 131072,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## 링크

- [Venice AI](https://venice.ai)
- [API 문서](https://docs.venice.ai)
- [가격](https://venice.ai/pricing)
- [상태](https://status.venice.ai)
