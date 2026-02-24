---
summary: "비용이 발생할 수 있는 기능, 사용되는 키, 사용량 확인 방법에 대한 감사"
read_when:
  - You want to understand which features may call paid APIs
  - You need to audit keys, costs, and usage visibility
  - You're explaining /status or /usage cost reporting
title: "API 사용량 및 비용"
---

# API 사용량 및 비용

이 문서는 **API 키를 호출할 수 있는 기능**과 해당 비용이 어디에 표시되는지 나열합니다.
프로바이더 사용량이나 유료 API 호출을 생성할 수 있는 OpenClaw 기능에 초점을 맞춥니다.

## 비용이 표시되는 곳 (채팅 + CLI)

**세션별 비용 스냅샷**

- `/status`는 현재 세션 모델, 컨텍스트 사용량, 마지막 응답 토큰을 표시합니다.
- 모델이 **API 키 인증**을 사용하면, `/status`는 마지막 응답의 **예상 비용**도 표시합니다.

**메시지별 비용 푸터**

- `/usage full`은 모든 응답에 **예상 비용** (API 키 전용)을 포함한 사용량 푸터를 추가합니다.
- `/usage tokens`는 토큰만 표시합니다; OAuth 플로우는 달러 비용을 숨깁니다.

**CLI 사용량 윈도우 (프로바이더 할당량)**

- `openclaw status --usage`와 `openclaw channels list`는 프로바이더 **사용량 윈도우**
  (할당량 스냅샷, 메시지별 비용이 아님)를 표시합니다.

자세한 내용은 [토큰 사용량 및 비용](/reference/token-use)을 참조하세요.

## 키가 검색되는 방법

OpenClaw은 다음에서 자격 증명을 가져올 수 있습니다:

- **인증 프로필** (에이전트별, `auth-profiles.json`에 저장).
- **환경 변수** (예: `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`).
- **설정** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`,
  `memorySearch.*`, `talk.apiKey`).
- **스킬** (`skills.entries.<name>.apiKey`) - 스킬 프로세스 환경에 키를 내보낼 수 있습니다.

## 키를 사용할 수 있는 기능

### 1) 핵심 모델 응답 (채팅 + 도구)

모든 응답이나 도구 호출은 **현재 모델 프로바이더** (OpenAI, Anthropic 등)를 사용합니다. 이것이
사용량과 비용의 주요 원인입니다.

가격 설정은 [모델](/providers/models)을, 표시에 대해서는 [토큰 사용량 및 비용](/reference/token-use)을 참조하세요.

### 2) 미디어 이해 (오디오/이미지/비디오)

수신 미디어는 응답 실행 전에 요약/전사될 수 있습니다. 모델/프로바이더 API를 사용합니다.

- 오디오: OpenAI / Groq / Deepgram (키가 있을 때 **자동 활성화**).
- 이미지: OpenAI / Anthropic / Google.
- 비디오: Google.

[미디어 이해](/nodes/media-understanding)를 참조하세요.

### 3) 메모리 임베딩 + 시맨틱 검색

시맨틱 메모리 검색은 원격 프로바이더로 설정된 경우 **임베딩 API**를 사용합니다:

- `memorySearch.provider = "openai"` → OpenAI 임베딩
- `memorySearch.provider = "gemini"` → Gemini 임베딩
- `memorySearch.provider = "voyage"` → Voyage 임베딩
- `memorySearch.provider = "mistral"` → Mistral 임베딩
- 로컬 임베딩 실패 시 원격 프로바이더로 선택적 폴백

`memorySearch.provider = "local"`로 로컬 유지 가능 (API 사용 없음).

[메모리](/concepts/memory)를 참조하세요.

### 4) 웹 검색 도구 (Brave / Perplexity via OpenRouter)

`web_search`는 API 키를 사용하며 사용 요금이 발생할 수 있습니다:

- **Brave Search API**: `BRAVE_API_KEY` 또는 `tools.web.search.apiKey`
- **Perplexity** (via OpenRouter): `PERPLEXITY_API_KEY` 또는 `OPENROUTER_API_KEY`

**Brave 무료 티어 (넉넉함):**

- **월 2,000건 요청**
- **초당 1건 요청**
- **신용카드 필요** (인증용, 업그레이드하지 않으면 청구 없음)

[웹 도구](/tools/web)를 참조하세요.

### 5) 웹 가져오기 도구 (Firecrawl)

`web_fetch`는 API 키가 있을 때 **Firecrawl**을 호출할 수 있습니다:

- `FIRECRAWL_API_KEY` 또는 `tools.web.fetch.firecrawl.apiKey`

Firecrawl이 설정되지 않으면 직접 fetch + readability로 폴백합니다 (유료 API 없음).

[웹 도구](/tools/web)를 참조하세요.

### 6) 프로바이더 사용량 스냅샷 (상태/상태 확인)

일부 상태 명령은 할당량 윈도우나 인증 상태를 표시하기 위해 **프로바이더 사용량 엔드포인트**를 호출합니다.
일반적으로 저빈도 호출이지만 여전히 프로바이더 API에 접근합니다:

- `openclaw status --usage`
- `openclaw models status --json`

[모델 CLI](/cli/models)를 참조하세요.

### 7) 컴팩션 안전장치 요약

컴팩션 안전장치는 **현재 모델**을 사용하여 세션 히스토리를 요약할 수 있으며,
실행 시 프로바이더 API를 호출합니다.

[세션 관리 + 컴팩션](/reference/session-management-compaction)을 참조하세요.

### 8) 모델 스캔 / 프로브

`openclaw models scan`은 OpenRouter 모델을 프로빙할 수 있으며 프로빙이 활성화된 경우
`OPENROUTER_API_KEY`를 사용합니다.

[모델 CLI](/cli/models)를 참조하세요.

### 9) 대화 (음성)

대화 모드는 설정 시 **ElevenLabs**를 호출할 수 있습니다:

- `ELEVENLABS_API_KEY` 또는 `talk.apiKey`

[대화 모드](/nodes/talk)를 참조하세요.

### 10) 스킬 (서드파티 API)

스킬은 `skills.entries.<name>.apiKey`에 `apiKey`를 저장할 수 있습니다. 스킬이 해당 키를 외부
API에 사용하면 스킬의 프로바이더에 따라 비용이 발생할 수 있습니다.

[스킬](/tools/skills)을 참조하세요.
