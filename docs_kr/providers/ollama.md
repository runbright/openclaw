---
summary: "Ollama(로컬 LLM 런타임)로 OpenClaw 실행하기"
read_when:
  - Ollama를 통해 로컬 모델로 OpenClaw을 실행하고 싶을 때
  - Ollama 설정 및 구성 안내가 필요할 때
title: "Ollama"
---

# Ollama

Ollama는 로컬 머신에서 오픈소스 모델을 쉽게 실행할 수 있게 해주는 로컬 LLM 런타임입니다. OpenClaw은 Ollama의 네이티브 API(`/api/chat`)와 통합되어 스트리밍과 도구 호출을 지원하며, `OLLAMA_API_KEY`(또는 인증 프로필)로 옵트인하고 명시적인 `models.providers.ollama` 항목을 정의하지 않으면 **도구 지원 모델을 자동 검색**할 수 있습니다.

## 빠른 시작

1. Ollama 설치: [https://ollama.ai](https://ollama.ai)

2. 모델 가져오기:

```bash
ollama pull gpt-oss:20b
# 또는
ollama pull llama3.3
# 또는
ollama pull qwen2.5-coder:32b
# 또는
ollama pull deepseek-r1:32b
```

3. OpenClaw에서 Ollama 활성화 (아무 값이나 사용 가능; Ollama는 실제 키가 필요하지 않음):

```bash
# 환경 변수 설정
export OLLAMA_API_KEY="ollama-local"

# 또는 설정 파일에서 구성
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama 모델 사용:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## 모델 검색 (암시적 프로바이더)

`OLLAMA_API_KEY`(또는 인증 프로필)를 설정하고 `models.providers.ollama`를 **정의하지 않으면**, OpenClaw은 `http://127.0.0.1:11434`의 로컬 Ollama 인스턴스에서 모델을 검색합니다:

- `/api/tags` 및 `/api/show`를 쿼리합니다
- `tools` 기능을 보고하는 모델만 유지합니다
- 모델이 `thinking`을 보고하면 `reasoning`으로 표시합니다
- 사용 가능한 경우 `model_info["<arch>.context_length"]`에서 `contextWindow`를 읽습니다
- `maxTokens`를 컨텍스트 윈도우의 10배로 설정합니다
- 모든 비용을 `0`으로 설정합니다

이를 통해 수동으로 모델 항목을 작성할 필요 없이 Ollama의 기능에 맞게 카탈로그를 유지할 수 있습니다.

사용 가능한 모델을 확인하려면:

```bash
ollama list
openclaw models list
```

새 모델을 추가하려면 Ollama로 간단히 가져오면 됩니다:

```bash
ollama pull mistral
```

새 모델이 자동으로 검색되어 사용 가능해집니다.

`models.providers.ollama`를 명시적으로 설정하면 자동 검색이 건너뛰어지고 아래와 같이 모델을 수동으로 정의해야 합니다.

## 구성

### 기본 설정 (암시적 검색)

Ollama를 활성화하는 가장 간단한 방법은 환경 변수를 사용하는 것입니다:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 명시적 설정 (수동 모델)

다음과 같은 경우 명시적 설정을 사용합니다:

- Ollama가 다른 호스트/포트에서 실행될 때.
- 특정 컨텍스트 윈도우나 모델 목록을 강제하고 싶을 때.
- 도구 지원을 보고하지 않는 모델을 포함하고 싶을 때.

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434",
        apiKey: "ollama-local",
        api: "ollama",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

`OLLAMA_API_KEY`가 설정되어 있으면 프로바이더 항목에서 `apiKey`를 생략할 수 있으며, OpenClaw이 가용성 확인을 위해 자동으로 채워줍니다.

### 사용자 정의 기본 URL (명시적 설정)

Ollama가 다른 호스트나 포트에서 실행 중인 경우 (명시적 설정은 자동 검색을 비활성화하므로 모델을 수동으로 정의해야 합니다):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434",
      },
    },
  },
}
```

### 모델 선택

구성이 완료되면 모든 Ollama 모델을 사용할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 고급 설정

### 추론 모델

Ollama가 `/api/show`에서 `thinking`을 보고하면 OpenClaw은 해당 모델을 추론 지원 모델로 표시합니다:

```bash
ollama pull deepseek-r1:32b
```

### 모델 비용

Ollama는 무료이며 로컬에서 실행되므로 모든 모델 비용은 $0으로 설정됩니다.

### 스트리밍 구성

OpenClaw의 Ollama 통합은 기본적으로 **네이티브 Ollama API**(`/api/chat`)를 사용하며, 스트리밍과 도구 호출을 동시에 완벽하게 지원합니다. 특별한 구성이 필요하지 않습니다.

#### 레거시 OpenAI 호환 모드

OpenAI 호환 엔드포인트를 대신 사용해야 하는 경우 (예: OpenAI 형식만 지원하는 프록시 뒤에 있는 경우), `api: "openai-completions"`를 명시적으로 설정합니다:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

참고: OpenAI 호환 엔드포인트는 스트리밍과 도구 호출을 동시에 지원하지 않을 수 있습니다. 모델 설정에서 `params: { streaming: false }`로 스트리밍을 비활성화해야 할 수 있습니다.

### 컨텍스트 윈도우

자동 검색된 모델의 경우, OpenClaw은 Ollama가 보고하는 컨텍스트 윈도우를 사용하며, 사용할 수 없는 경우 기본값 `8192`를 사용합니다. 명시적 프로바이더 설정에서 `contextWindow`와 `maxTokens`를 재정의할 수 있습니다.

## 문제 해결

### Ollama가 감지되지 않음

Ollama가 실행 중이고 `OLLAMA_API_KEY`(또는 인증 프로필)가 설정되어 있으며, 명시적 `models.providers.ollama` 항목을 정의하지 **않았는지** 확인하세요:

```bash
ollama serve
```

API에 접근할 수 있는지 확인합니다:

```bash
curl http://localhost:11434/api/tags
```

### 사용 가능한 모델이 없음

OpenClaw은 도구 지원을 보고하는 모델만 자동 검색합니다. 모델이 목록에 나타나지 않으면 다음 중 하나를 수행하세요:

- 도구 지원 모델을 가져오거나,
- `models.providers.ollama`에서 모델을 명시적으로 정의합니다.

모델을 추가하려면:

```bash
ollama list  # 설치된 모델 확인
ollama pull gpt-oss:20b  # 도구 지원 모델 가져오기
ollama pull llama3.3     # 또는 다른 모델
```

### 연결 거부

Ollama가 올바른 포트에서 실행 중인지 확인합니다:

```bash
# Ollama가 실행 중인지 확인
ps aux | grep ollama

# 또는 Ollama 재시작
ollama serve
```

## 관련 문서

- [모델 프로바이더](/concepts/model-providers) - 모든 프로바이더 개요
- [모델 선택](/concepts/models) - 모델 선택 방법
- [구성](/gateway/configuration) - 전체 설정 참조
