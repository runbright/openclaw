---
summary: "Nextcloud Talk(넥스트클라우드 토크) 연동 상태, 지원 기능 및 설정 가이드"
read_when:
  - Nextcloud Talk 채널 기능을 구현하거나 디버깅할 때
title: "Nextcloud Talk"
---

# Nextcloud Talk (플러그인)

OpenClaw는 플러그인을 통해 Nextcloud Talk 웨브훅 봇 연동을 지원합니다. 개인 메시지(DM), 룸 대화, 리액션 및 마크다운 메시지 처리가 가능합니다.

## 플러그인 설치

Nextcloud Talk 플러그인을 먼저 설치해야 합니다:

```bash
openclaw plugins install @openclaw/nextcloud-talk
```

## 빠른 설정 (초보자용)

1. **Nextcloud 서버에서 봇 설치**:
   서버 터미널에서 `occ` 명령어를 사용하여 봇을 등록합니다:
   ```bash
   ./occ talk:bot:install "OpenClaw" "<공유_시크릿>" "<웨브훅_URL>" --feature reaction
   ```
2. **룸에서 봇 활성화**: 대화방 설정 메뉴에서 방금 생성한 봇을 활성화합니다.
3. **OpenClaw 설정 (`openclaw.json`)**:
   - `baseUrl`: Nextcloud 인스턴스 주소
   - `botSecret`: 위에서 설정한 공유 시크릿

### 최소 설정 예시

```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "공유_시크릿",
      dmPolicy: "pairing",
    },
  },
}
```

## 주요 특징 및 제한 사항

- **DM 시작 불가**: 봇이 먼저 개인 메시지를 보낼 수는 없습니다. 사용자가 먼저 봇에게 메시지를 보내야 대화가 시작됩니다.
- **미디어 처리**: 봇 API 특성상 직접적인 파일 업로드는 지원하지 않으며, 미디어 데이터는 URL 형태로 전달됩니다.
- **DM 감지**: 정확한 DM과 룸 구분을 위해서는 `apiUser` 및 `apiPassword` 설정을 추가하여 룸 정보를 조회할 수 있도록 권한을 부여하는 것이 좋습니다.

## 접근 제어 (Access Control)

- **DM**: 기본적으로 `pairing` 모드입니다. `openclaw pairing approve nextcloud-talk <코드>`로 승인 후 사용 가능합니다.
- **룸(그룹)**: 기본적으로 `allowlist` 모드입니다. `rooms` 설정에 룸 토큰을 추가하여 에이전트가 응답할 방을 지정하십시오.

## 지원 기능 요약

| 기능 | 상태 |
| :--- | :--- |
| 개인 메시지(DM) | ✅ 지원 |
| 룸(그룹) | ✅ 지원 |
| 리액션 | ✅ 지원 |
| 미디어 | ⚠️ URL 형식만 지원 |
| 스레드 | ❌ 지원 안 함 |

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration)를 확인하세요.
---
