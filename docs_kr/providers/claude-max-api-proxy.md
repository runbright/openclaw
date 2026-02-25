---
summary: "OpenAI 호환 API를 통해 클로드 맥스(Claude Max) 구독 모델을 사용하는 방법"
read_when:
  - Claude Max/Pro 구독을 OpenAI 호환 도구와 함께 사용하고 싶을 때
  - API 키 대신 구독 서비스를 사용하여 비용을 절감하고 싶을 때
title: "클로드 맥스 API 프록시 (Claude Max API Proxy)"
---

# 클로드 맥스 API 프록시 (Claude Max API Proxy)

**claude-max-api-proxy**는 클로드 맥스(Claude Max) 또는 프로(Pro) 구독 계정을 OpenAI 호환 API 엔드포인트로 변환해주는 커뮤니티 도구입니다. 이를 통해 구독 기반의 무제한 모델 사용량을 OpenAI 규격을 지원하는 도구들에서 그대로 활용할 수 있습니다.

## 왜 사용하나요?
- **비용 절감**: Anthropic API(사용량 기반 과금) 대신, 개인 구독(Claude Max/Pro) 서비스를 활용하여 개발 및 개인 용도로 무제한에 가까운 사용이 가능합니다.
- **간편한 연동**: OpenAI API 형식을 지원하는 모든 앱에서 사용할 수 있습니다.

## 작동 방식
이 프록시 도구는 로컬에서 실행되며, 여러분의 요청을 클로드 코드(Claude Code) CLI 명령으로 변환하여 Anthropic 서버로 전달합니다.
```
내 앱 → claude-max-api-proxy (로컬) → Claude Code CLI → Anthropic 서버 (구독 계정 사용)
```

## 빠른 설치 및 실행

1.  **필수 조건**: Node.js 20 버전 이상 및 Claude Code CLI가 설치되어 있어야 합니다.
2.  **설치**:
    ```bash
    npm install -g claude-max-api-proxy
    ```
3.  **서버 실행**:
    ```bash
    claude-max-api
    # 기본적으로 http://localhost:3456 에서 실행됩니다.
    ```

## OpenClaw와 함께 사용하기
OpenClaw 설정에서 커스텀 OpenAI 엔드포인트로 지정하여 사용할 수 있습니다.
```json5
{
  "env": {
    "OPENAI_API_KEY": "not-needed", // 키는 필요하지 않음
    "OPENAI_BASE_URL": "http://localhost:3456/v1"
  },
  "agents": {
    "defaults": {
      "model": { "primary": "openai/claude-opus-4" }
    }
  }
}
```

## 사용 가능 모델
- `claude-opus-4`
- `claude-sonnet-4`
- `claude-haiku-4`

## 주의 사항
- 이 도구는 **커뮤니티 프로젝트**로, Anthropic이나 OpenClaw 공식 지원 도구가 아닙니다.
- 활성화된 클로드 맥스/프로 구독 계정이 반드시 필요합니다.
- 모든 데이터 처리는 로컬 프록시 서버를 통해 안전하게 전달됩니다.

---
공식 API 연동을 원하시면 [Anthropic 제공업체 가이드](/providers/anthropic)를 참고하세요.
---
