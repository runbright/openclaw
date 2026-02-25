---
title: "클라우드플레어 AI 게이트웨이"
summary: "클라우드플레어 AI 게이트웨이 설정 (인증 및 모델 선택)"
read_when:
  - OpenClaw를 Cloudflare AI Gateway와 연동하고 싶을 때
  - 계정 ID, 게이트웨이 ID 또는 API 키 설정 방법을 확인해야 할 때
---

# 클라우드플레어 AI 게이트웨이 (Cloudflare AI Gateway)

클라우드플레어 AI 게이트웨이는 기존 AI 제공업체(Anthropic 등) 앞에 위치하여 분석(Analytics), 캐싱(Caching), 그리고 각종 제어 기능을 추가할 수 있는 서비스입니다. OpenClaw는 사용자의 게이트웨이 엔드포인트를 통해 Anthropic 메시지 API 등을 호출합니다.

## 주요 정보
- **제공업체 ID**: `cloudflare-ai-gateway`
- **베이스 URL**: `https://gateway.ai.cloudflare.com/v1/<account_id>/<gateway_id>/anthropic`
- **기본 모델**: `cloudflare-ai-gateway/claude-sonnet-4-5`
- **API 키**: `CLOUDFLARE_AI_GATEWAY_API_KEY` 환경 변수를 사용합니다.

## 빠른 설정 방법

1.  **연동 정보 입력**: 온보딩 도구를 사용하여 계정 ID와 게이트웨이 정보를 입력합니다.
    ```bash
    openclaw onboard --auth-choice cloudflare-ai-gateway-api-key
    ```
2.  **기본 모델 설정 (`openclaw.json`)**:
    ```json5
    {
      "agents": {
        "defaults": {
          "model": { "primary": "cloudflare-ai-gateway/claude-sonnet-4-5" }
        }
      }
    }
    ```

## 보완 보안 (인증된 게이트웨이)
클라우드플레어 측에서 게이트웨이 인증을 활성화한 경우, `cf-aig-authorization` 헤더를 추가해야 합니다.
```json5
{
  "models": {
    "providers": {
      "cloudflare-ai-gateway": {
        "headers": {
          "cf-aig-authorization": "Bearer <게이트웨이-인증-토큰>"
        }
      }
    }
  }
}
```

## 참고 사항
- **환경 변수**: 데몬(Daemon) 형태로 실행 시 `~/.openclaw/.env` 파일 등에 API 키가 정의되어 있는지 확인하세요.
- **캐싱**: 이 게이트웨이를 사용하면 반복되는 동일한 요청에 대해 캐싱을 적용하여 비용을 절감할 수 있습니다.

---
상세 설정은 [클라우드플레어 관리 콘솔](https://dash.cloudflare.com)을 확인하세요.
---
