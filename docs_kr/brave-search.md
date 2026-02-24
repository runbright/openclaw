---
summary: "web_search 도구를 위한 Brave Search API 설정 가이드"
read_when:
  - web_search 도구의 엔진으로 Brave Search를 사용하고 싶을 때
  - BRAVE_API_KEY 발급 및 요금제 정보가 필요할 때
title: "Brave Search"
---

# Brave Search API

OpenClaw는 웹 검색(`web_search`) 도구의 기본 엔진으로 Brave Search를 사용합니다.

## API 키 발급 방법

1. [Brave Search API 사이트](https://brave.com/search/api/)에서 계정을 생성합니다.
2. 대시보드에서 **Data for Search** 요금제를 선택하고 API 키를 생성합니다.
3. 발급받은 키를 `openclaw.json` 설정 파일에 저장하거나, 게이트웨이 환경 변수에 `BRAVE_API_KEY`로 설정합니다.

## 설정 예시 (openclaw.json)

```json5
{
  "tools": {
    "web": {
      "search": {
        "provider": "brave",
        "apiKey": "발급받은_API_키",
        "maxResults": 5, // 최대 검색 결과 수
        "timeoutSeconds": 30 // 타임아웃 설정
      }
    }
  }
}
```

## 주의 사항

- **Data for AI** 요금제는 `web_search` 도구와 호환되지 않습니다. 반드시 **Data for Search**를 사용하세요.
- Brave Search는 무료 티어와 유료 요금제를 제공합니다. 현재 사용량 제한 등은 Brave API 포털에서 확인하세요.

모든 웹 관련 도구 설정에 대한 내용은 [웹 도구 가이드](/tools/web)에서 확인할 수 있습니다.
---
