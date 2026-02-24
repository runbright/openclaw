---
summary: "LINE Messaging API 플러그인 설정, 구성 및 사용 방법 안내"
read_when:
  - OpenClaw를 LINE에 연결하고 싶을 때
  - LINE 웨브훅 및 인증 정보를 설정할 때
  - LINE 전용 메시지 기능을 사용하고 싶을 때
title: "LINE (라인)"
---

# LINE (플러그인)

OpenClaw는 LINE Messaging API를 통해 LINE과 연동됩니다. 이 플러그인은 게이트웨이에서 웨브훅(Webhook) 수신기로 작동하며, 채널 액세스 토큰과 채널 시크릿을 사용하여 인증을 처리합니다.

## 플러그인 설치

먼저 LINE 플러그인을 설치해야 합니다:

```bash
openclaw plugins install @openclaw/line
```

## 설정 가이드

1. **LINE Developers 콘솔 접속**: [https://developers.line.biz/console/](https://developers.line.biz/console/)
2. **채널 생성**: Provider를 생성하거나 선택한 뒤, **Messaging API** 채널을 추가합니다.
3. **토큰 및 시크릿 복사**: 채널 설정에서 **Channel access token**과 **Channel secret**을 복사합니다.
4. **웨브훅 활성화**: Messaging API 설정에서 **Use webhook**을 켭니다.
5. **웨브훅 URL 설정**: 본인의 게이트웨이 엔드포인트(HTTPS 필수)를 입력합니다:
   `https://게이트웨이_도메인/line/webhook`

## OpenClaw 설정 (`openclaw.json`)

최소 설정 예시:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "발급받은_액세스_토큰",
      channelSecret: "발급받은_채널_시크릿",
      dmPolicy: "pairing",
    },
  },
}
```

## 접근 제어 (Access Control)

- **DM 정책**: 기본적으로 `pairing` 모드입니다. 모르는 사용자에게는 페어링 코드를 발송하며, 승인 전까지는 메시지가 무시됩니다.
- **승인 방법**: `openclaw pairing list line`으로 대상을 확인하고 `openclaw pairing approve line <코드>`로 승인합니다.
- **그룹 정책**: 특정 사용자만 에이전트를 호출할 수 있도록 화이트리스트(`groupAllowFrom`)를 설정할 수 있습니다.

## 메시지 동작 및 고급 기능

- **텍스트 한도**: 한 번에 최대 5,000자까지 전송됩니다.
- **리치 메시지 지원**: `channelData.line`을 사용하여 퀵 리플라이, 위치 정보, 플렉스 카드(Flex Cards), 템플릿 메시지 등을 보낼 수 있습니다.
- **스트리밍**: LINE은 메시지 수정을 지원하지 않으므로, 에이전트가 답변을 생성하는 동안 사용자에게 "처리 중" 애니메이션을 보여주고 완료된 뒤 한 번에 전송합니다.

## 트러블슈팅

- **웨브훅 확인 실패**: URL이 `https`인지, 그리고 `channelSecret`이 콘솔의 값과 정확히 일치하는지 확인하세요.
- **메시지 수신 안 됨**: `webhookPath` 설정이 LINE 콘솔에 입력한 경로와 일치하는지, 게이트웨이가 외부에서 접속 가능한지 확인하세요.

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration-reference#line)를 확인하세요.
---
