---
summary: "Zalo(잘로) 봇 연동 상태, 지원 기능 및 설정 가이드 (베트남 메신저)"
read_when:
  - Zalo 연동 기능이나 웨브훅을 작업할 때
title: "Zalo"
---

# Zalo (봇 API)

**상태**: 실험적 지원. 현재는 개인 메시지(DM)만 지원하며, 그룹 기능은 Zalo 측에서 준비 중입니다.

## 플러그인 설치

Zalo는 별도의 플러그인 설치가 필요합니다:

```bash
openclaw plugins install @openclaw/zalo
```

## 빠른 설정 가이드

1. **봇 토큰 발급**: [Zalo Bot Platform](https://bot.zaloplatforms.com)에 로그인하여 새 봇을 생성하고 봇 토큰(`12345689:abc-xyz` 형식)을 복사합니다.
2. **OpenClaw 설정 (`openclaw.json`)**:

```json5
{
  channels: {
    zalo: {
      enabled: true,
      botToken: "발급받은_봇_토큰",
      dmPolicy: "pairing",
    },
  },
}
```

3. **게이트웨이 실행**: 봇에게 메시지를 보낸 후 `openclaw pairing approve zalo <코드>` 명령어로 승인합니다.

## 작동 방식

- **메시지 수신**: 기본적으로 롱 폴링(Long-polling) 방식을 사용하므로 외부에서 에이전트 서버로 접속할 수 있는 공인 URL이 없어도 동작합니다.
- **웨브훅**: `webhookUrl` 설정을 통해 실시간 웨브훅 방식으로 전환할 수 있습니다 (HTTPS 필수).
- **메시지 분할**: Zalo API 제한에 따라 2,000자가 넘는 긴 메시지는 자동으로 분할되어 전송됩니다.
- **이미지**: 발신자가 보낸 이미지를 처리하고, 에이전트가 답변으로 이미지를 보낼 수 있습니다.

## 접근 제어 (Access Control)

- **DM**: 기본적으로 `pairing` 모드입니다. 상세 내용은 [페어링 가이드](/channels/pairing)를 참고하세요.
- **허용 목록**: `allowFrom` 설정에 숫자 형태의 사용자 ID를 등록하여 특정 사용자만 허용할 수 있습니다.

## 지원 기능 요약

| 기능 | 상태 |
| :--- | :--- |
| 개인 메시지(DM) | ✅ 지원 |
| 그룹 | ❌ Zalo에서 준비 중 |
| 이미지/미디어 | ✅ 지원 |
| 스트리밍 | ⚠️ 제한적 (2,000자 제한으로 인해 차단됨) |

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration)를 확인하세요.
---
