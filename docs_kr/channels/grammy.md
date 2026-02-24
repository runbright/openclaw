---
summary: "grammY 프레임워크를 통한 텔레그램 봇 API 연동 기술 정보"
read_when:
  - 텔레그램 연동의 기술적 구현이 궁금하거나 설정 항목을 상세히 알고 싶을 때
title: "grammY (텔레그램)"
---

# grammY 연동 (Telegram Bot API)

OpenClaw는 텔레그램 봇 API 연동을 위해 **grammY** 프레임워크를 사용합니다. grammY는 타입스크립트 기반의 강력한 봇 개발 도구로, 안정적인 메시지 수발신과 미디어 처리를 가능하게 합니다.

## 주요 특징
- **단일 클라이언트 경로**: 이전의 직접 구현 방식 대신 grammY를 전용 클라이언트로 채택하여 안정성을 높였습니다.
- **롱 폴링 및 웹훅 지원**: 설정에 따라 롱 폴링(Long polling) 또는 웹훅(Webhook) 방식을 선택하여 메시지를 받을 수 있습니다.
- **실시간 응답 (Streaming)**: `channels.telegram.streaming` 설정을 통해 AI가 답변을 작성하는 과정을 실시간으로 업데이트하여 보여줄 수 있습니다. (메시지 수정 방식 사용)
- **프록시 지원**: 네트워크 환경에 따라 전용 프록시 서버 설정을 지원합니다.

## 주요 설정 항목
- `botToken`: 텔레그램 봇 토큰 (필수)
- `dmPolicy`: 개인 메시지 보안 정책 (`pairing`, `allowlist` 등)
- `groupPolicy`: 그룹 대화 허용 정책
- `streaming`: 실시간 응답 표시 여부 (`off`, `progress` 등)
- `webhookUrl`: 웹훅 사용 시 엔드포인트 주소

---
텔레그램의 일반적인 설정 방법은 [텔레그램 가이드](/channels/telegram)를 참고하세요.
---
