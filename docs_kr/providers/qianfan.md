---
summary: "OpenClaw에서 Qianfan의 통합 API로 다양한 모델 사용하기"
read_when:
  - 여러 LLM을 위한 단일 API 키를 원할 때
  - Baidu Qianfan 설정 안내가 필요할 때
title: "Qianfan"
---

# Qianfan 프로바이더 가이드

Qianfan은 Baidu의 MaaS 플랫폼으로, 단일 엔드포인트와 API 키로 다양한 모델에 요청을 라우팅하는 **통합 API**를 제공합니다. OpenAI와 호환되므로 대부분의 OpenAI SDK는 기본 URL만 변경하면 작동합니다.

## 사전 요구 사항

1. Qianfan API 접근 권한이 있는 Baidu Cloud 계정
2. Qianfan 콘솔에서 발급한 API 키
3. 시스템에 설치된 OpenClaw

## API 키 발급

1. [Qianfan 콘솔](https://console.bce.baidu.com/qianfan/ais/console/apiKey)에 방문합니다
2. 새 애플리케이션을 만들거나 기존 애플리케이션을 선택합니다
3. API 키를 생성합니다 (형식: `bce-v3/ALTAK-...`)
4. OpenClaw에서 사용할 API 키를 복사합니다

## CLI 설정

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## 관련 문서

- [OpenClaw 구성](/gateway/configuration)
- [모델 프로바이더](/concepts/model-providers)
- [에이전트 설정](/concepts/agent)
- [Qianfan API 문서](https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)
