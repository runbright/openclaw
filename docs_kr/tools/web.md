---
summary: "웹 검색 + 가져오기 도구 (Brave Search API, Perplexity 직접/OpenRouter, Gemini Google Search 그라운딩)"
read_when:
  - web_search 또는 web_fetch를 활성화하려는 경우
  - Brave Search API 키 설정이 필요한 경우
  - 웹 검색에 Perplexity Sonar를 사용하려는 경우
  - Google Search 그라운딩으로 Gemini를 사용하려는 경우
title: "웹 도구"
---

# 웹 도구

OpenClaw는 두 가지 가벼운 웹 도구를 제공합니다:

- `web_search` -- Brave Search API(기본값), Perplexity Sonar 또는 Google Search 그라운딩이 포함된 Gemini를 통해 웹을 검색합니다.
- `web_fetch` -- HTTP 가져오기 + 읽기 가능한 추출(HTML → markdown/text).

이것은 브라우저 자동화가 **아닙니다**. JS가 많은 사이트나 로그인이 필요한 경우 [브라우저 도구](/tools/browser)를 사용하세요.

## 작동 방식

- `web_search`는 구성된 프로바이더를 호출하고 결과를 반환합니다.
  - **Brave**(기본값): 구조화된 결과(제목, URL, 스니펫)를 반환합니다.
  - **Perplexity**: 실시간 웹 검색에서 인용이 포함된 AI 합성 답변을 반환합니다.
  - **Gemini**: Google Search에 기반한 인용이 포함된 AI 합성 답변을 반환합니다.
- 결과는 쿼리별로 15분간 캐시됩니다(구성 가능).
- `web_fetch`는 일반 HTTP GET을 수행하고 읽기 가능한 콘텐츠를 추출합니다(HTML → markdown/text). JavaScript를 **실행하지 않습니다**.
- `web_fetch`는 기본적으로 활성화되어 있습니다(명시적으로 비활성화하지 않는 한).

## 검색 프로바이더 선택

| 프로바이더          | 장점                                         | 단점                                     | API 키                                       |
| ------------------- | -------------------------------------------- | ---------------------------------------- | -------------------------------------------- |
| **Brave**(기본값)   | 빠르고, 구조화된 결과, 무료 티어             | 전통적인 검색 결과                       | `BRAVE_API_KEY`                              |
| **Perplexity**      | AI 합성 답변, 인용, 실시간                   | Perplexity 또는 OpenRouter 접근 필요     | `OPENROUTER_API_KEY` 또는 `PERPLEXITY_API_KEY` |
| **Gemini**          | Google Search 그라운딩, AI 합성              | Gemini API 키 필요                       | `GEMINI_API_KEY`                             |

[Brave Search 설정](/brave-search) 및 [Perplexity Sonar](/perplexity)에서 프로바이더별 세부 사항을 확인하세요.

### 자동 감지

`provider`가 명시적으로 설정되지 않으면 OpenClaw는 사용 가능한 API 키를 기반으로 사용할 프로바이더를 자동 감지하며, 다음 순서로 확인합니다:

1. **Brave** -- `BRAVE_API_KEY` 환경 변수 또는 `search.apiKey` 구성
2. **Gemini** -- `GEMINI_API_KEY` 환경 변수 또는 `search.gemini.apiKey` 구성
3. **Perplexity** -- `PERPLEXITY_API_KEY` / `OPENROUTER_API_KEY` 환경 변수 또는 `search.perplexity.apiKey` 구성
4. **Grok** -- `XAI_API_KEY` 환경 변수 또는 `search.grok.apiKey` 구성

키가 없으면 Brave로 폴백합니다(키 누락 오류가 표시되어 구성하라는 안내가 나옵니다).

### 명시적 프로바이더

구성에서 프로바이더를 설정합니다:

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave", // 또는 "perplexity" 또는 "gemini"
      },
    },
  },
}
```

예제: Perplexity Sonar(직접 API)로 전환:

```json5
{
  tools: {
    web: {
      search: {
        provider: "perplexity",
        perplexity: {
          apiKey: "pplx-...",
          baseUrl: "https://api.perplexity.ai",
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

## Brave API 키 받기

1. [https://brave.com/search/api/](https://brave.com/search/api/)에서 Brave Search API 계정을 만듭니다
2. 대시보드에서 **Data for Search** 플랜("Data for AI" 아님)을 선택하고 API 키를 생성합니다.
3. `openclaw configure --section web`을 실행하여 구성에 키를 저장하거나(권장), 환경에 `BRAVE_API_KEY`를 설정합니다.

Brave는 무료 티어와 유료 플랜을 제공합니다; 현재 제한 및 가격은 Brave API 포털을 확인하세요.

### 키를 설정할 위치(권장)

**권장:** `openclaw configure --section web`을 실행하세요. `~/.openclaw/openclaw.json`의 `tools.web.search.apiKey` 아래에 키를 저장합니다.

**환경 대안:** Gateway 프로세스 환경에 `BRAVE_API_KEY`를 설정하세요. 게이트웨이 설치의 경우 `~/.openclaw/.env`(또는 서비스 환경)에 넣으세요. [환경 변수](/help/faq#how-does-openclaw-load-environment-variables)를 참조하세요.

## Perplexity 사용(직접 또는 OpenRouter를 통해)

Perplexity Sonar 모델은 내장 웹 검색 기능을 가지고 있으며 인용이 포함된 AI 합성 답변을 반환합니다. OpenRouter를 통해 사용할 수 있습니다(신용카드 불필요 - 암호화폐/선불 지원).

### OpenRouter API 키 받기

1. [https://openrouter.ai/](https://openrouter.ai/)에서 계정을 만듭니다
2. 크레딧을 추가합니다(암호화폐, 선불 또는 신용카드 지원)
3. 계정 설정에서 API 키를 생성합니다

### Perplexity 검색 설정

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        provider: "perplexity",
        perplexity: {
          // API 키(OPENROUTER_API_KEY 또는 PERPLEXITY_API_KEY가 설정되면 선택 사항)
          apiKey: "sk-or-v1-...",
          // Base URL(생략 시 키 인식 기본값)
          baseUrl: "https://openrouter.ai/api/v1",
          // 모델(기본값 perplexity/sonar-pro)
          model: "perplexity/sonar-pro",
        },
      },
    },
  },
}
```

**환경 대안:** Gateway 환경에 `OPENROUTER_API_KEY` 또는 `PERPLEXITY_API_KEY`를 설정하세요. 게이트웨이 설치의 경우 `~/.openclaw/.env`에 넣으세요.

base URL이 설정되지 않으면 OpenClaw는 API 키 소스에 따라 기본값을 선택합니다:

- `PERPLEXITY_API_KEY` 또는 `pplx-...` → `https://api.perplexity.ai`
- `OPENROUTER_API_KEY` 또는 `sk-or-...` → `https://openrouter.ai/api/v1`
- 알 수 없는 키 형식 → OpenRouter(안전한 폴백)

### 사용 가능한 Perplexity 모델

| 모델                             | 설명                                 | 적합한 용도       |
| -------------------------------- | ------------------------------------ | ----------------- |
| `perplexity/sonar`               | 웹 검색이 포함된 빠른 Q&A            | 빠른 조회         |
| `perplexity/sonar-pro`(기본값)   | 웹 검색이 포함된 다단계 추론         | 복잡한 질문       |
| `perplexity/sonar-reasoning-pro` | 사고 체인 분석                       | 심층 리서치       |

## Gemini 사용(Google Search 그라운딩)

Gemini 모델은 내장 [Google Search 그라운딩](https://ai.google.dev/gemini-api/docs/grounding)을 지원하며, 인용이 포함된 실시간 Google Search 결과에 기반한 AI 합성 답변을 반환합니다.

### Gemini API 키 받기

1. [Google AI Studio](https://aistudio.google.com/apikey)로 이동합니다
2. API 키를 만듭니다
3. Gateway 환경에 `GEMINI_API_KEY`를 설정하거나 `tools.web.search.gemini.apiKey`를 구성합니다

### Gemini 검색 설정

```json5
{
  tools: {
    web: {
      search: {
        provider: "gemini",
        gemini: {
          // API 키(GEMINI_API_KEY가 설정되면 선택 사항)
          apiKey: "AIza...",
          // 모델(기본값 "gemini-2.5-flash")
          model: "gemini-2.5-flash",
        },
      },
    },
  },
}
```

**환경 대안:** Gateway 환경에 `GEMINI_API_KEY`를 설정하세요.
게이트웨이 설치의 경우 `~/.openclaw/.env`에 넣으세요.

### 참고 사항

- Gemini 그라운딩의 인용 URL은 Google의 리다이렉트 URL에서 직접 URL로 자동 해석됩니다.
- 리다이렉트 해석은 최종 인용 URL을 반환하기 전에 SSRF 가드 경로(HEAD + 리다이렉트 확인 + http/https 유효성 검사)를 사용합니다.
- 이 리다이렉트 해석기는 Gateway 운영자 신뢰 가정에 맞추어 신뢰 네트워크 모델(기본적으로 사설/내부 네트워크 허용)을 따릅니다.
- 기본 모델(`gemini-2.5-flash`)은 빠르고 비용 효율적입니다.
  그라운딩을 지원하는 모든 Gemini 모델을 사용할 수 있습니다.

## web_search

구성된 프로바이더를 사용하여 웹을 검색합니다.

### 요구 사항

- `tools.web.search.enabled`가 `false`가 아니어야 합니다(기본값: 활성화)
- 선택한 프로바이더의 API 키:
  - **Brave**: `BRAVE_API_KEY` 또는 `tools.web.search.apiKey`
  - **Perplexity**: `OPENROUTER_API_KEY`, `PERPLEXITY_API_KEY` 또는 `tools.web.search.perplexity.apiKey`

### 구성

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE", // BRAVE_API_KEY가 설정되면 선택 사항
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
    },
  },
}
```

### 도구 파라미터

- `query`(필수)
- `count`(1-10; 구성의 기본값)
- `country`(선택 사항): 지역별 결과를 위한 2자리 국가 코드(예: "DE", "US", "ALL"). 생략하면 Brave가 기본 지역을 선택합니다.
- `search_lang`(선택 사항): 검색 결과를 위한 ISO 언어 코드(예: "de", "en", "fr")
- `ui_lang`(선택 사항): UI 요소를 위한 ISO 언어 코드
- `freshness`(선택 사항): 발견 시간별 필터
  - Brave: `pd`, `pw`, `pm`, `py` 또는 `YYYY-MM-DDtoYYYY-MM-DD`
  - Perplexity: `pd`, `pw`, `pm`, `py`

**예제:**

```javascript
// 독일 특화 검색
await web_search({
  query: "TV online schauen",
  count: 10,
  country: "DE",
  search_lang: "de",
});

// 프랑스어 UI가 포함된 프랑스어 검색
await web_search({
  query: "actualités",
  country: "FR",
  search_lang: "fr",
  ui_lang: "fr",
});

// 최근 결과(지난 주)
await web_search({
  query: "TMBG interview",
  freshness: "pw",
});
```

## web_fetch

URL을 가져오고 읽기 가능한 콘텐츠를 추출합니다.

### web_fetch 요구 사항

- `tools.web.fetch.enabled`가 `false`가 아니어야 합니다(기본값: 활성화)
- 선택적 Firecrawl 폴백: `tools.web.fetch.firecrawl.apiKey` 또는 `FIRECRAWL_API_KEY`를 설정합니다.

### web_fetch 구성

```json5
{
  tools: {
    web: {
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        maxResponseBytes: 2000000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 14_7_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
        readability: true,
        firecrawl: {
          enabled: true,
          apiKey: "FIRECRAWL_API_KEY_HERE", // FIRECRAWL_API_KEY가 설정되면 선택 사항
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 86400000, // ms (1일)
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

### web_fetch 도구 파라미터

- `url`(필수, http/https만)
- `extractMode`(`markdown` | `text`)
- `maxChars`(긴 페이지 잘림)

참고:

- `web_fetch`는 먼저 Readability(메인 콘텐츠 추출)를 사용하고, 그다음 Firecrawl(구성된 경우)을 사용합니다. 둘 다 실패하면 도구가 오류를 반환합니다.
- Firecrawl 요청은 봇 우회 모드를 사용하고 기본적으로 결과를 캐시합니다.
- `web_fetch`는 기본적으로 Chrome 유사 User-Agent와 `Accept-Language`를 전송합니다; 필요하면 `userAgent`를 재정의하세요.
- `web_fetch`는 사설/내부 호스트 이름을 차단하고 리다이렉트를 재확인합니다(`maxRedirects`로 제한).
- `maxChars`는 `tools.web.fetch.maxCharsCap`으로 제한됩니다.
- `web_fetch`는 파싱 전에 다운로드된 응답 본문 크기를 `tools.web.fetch.maxResponseBytes`로 제한합니다; 초과된 응답은 잘리고 경고가 포함됩니다.
- `web_fetch`는 최선의 노력 추출입니다; 일부 사이트는 브라우저 도구가 필요합니다.
- 키 설정 및 서비스 세부 사항은 [Firecrawl](/tools/firecrawl)을 참조하세요.
- 응답은 캐시됩니다(기본 15분, 반복 가져오기를 줄이기 위해).
- 도구 프로필/허용 목록을 사용하는 경우 `web_search`/`web_fetch` 또는 `group:web`을 추가하세요.
- Brave 키가 없으면 `web_search`는 문서 링크가 포함된 짧은 설정 힌트를 반환합니다.
