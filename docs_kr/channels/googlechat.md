---
summary: "구글 챗(Google Chat) 앱 연동 상태, 기능 및 설정 가이드"
read_when:
  - 구글 챗 API를 통해 에이전트를 연동하고 싶을 때
  - 서비스 계정(Service Account) 및 웹훅(Webhook) 설정을 확인해야 할 때
title: "구글 챗"
---

# 구글 챗 (Google Chat API)

구글 챗 채널은 구글 클라우드 플랫폼(GCP)의 **Chat API**를 사용하여 에이전트와 대화할 수 있게 해줍니다.

## 빠른 설정 방법 (초보자 가이드)

### 1단계: 구글 클라우드 설정
1.  [Google Cloud Console](https://console.cloud.google.com/)에서 새 프로젝트를 생성하고 **Google Chat API**를 활성화합니다.
2.  **서비스 계정(Service Account)** 생성:
    - 'API 및 서비스 > 사용자 인증 정보'에서 서비스 계정을 만들고 이름을 정합니다.
3.  **JSON 키 다운로드**:
    - 생성된 서비스 계정의 '키(Keys)' 탭에서 '새 키 만들기(JSON)'를 선택하여 파일을 다운로드합니다. 이 파일을 OpenClaw 서버의 안전한 곳(예: `~/.openclaw/googlechat-key.json`)에 저장합니다.
4.  **Chat 앱 구성**:
    - 'Google Chat API > 구성' 메뉴에서 앱 이름, 아바타 등을 설정합니다.
    - **연결 설정**에서 'HTTP 엔드포인트 URL'을 선택하고, 자신의 게이트웨이 주소(예: `https://mydomain.com/googlechat`)를 입력합니다.

### 2단계: OpenClaw 설정
`openclaw.json` 파일에 서비스 계정 경로와 웹훅 정보를 입력합니다.
```json5
{
  "channels": {
    "googlechat": {
      "enabled": true,
      "serviceAccountFile": "/홈/경로/.openclaw/googlechat-key.json",
      "audienceType": "app-url",
      "audience": "https://mydomain.com/googlechat"
    }
  }
}
```

## 외부 접속을 위한 HTTPS 웹훅 설정
구글 챗은 반드시 보안 연결(HTTPS)이 된 공용 서버 주소가 필요합니다.

- **권장 방법 (Tailscale Funnel)**: 포트 포워딩 없이도 안전한 HTTPS 주소를 만들어줍니다.
  ```bash
  tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat
  ```

## 사용 방법
1.  [Google Chat](https://chat.google.com/)에 접속합니다.
2.  '새 대화'에서 자신이 설정한 **앱 이름**을 검색하여 추가합니다. (비공개 앱이므로 이름으로 직접 찾아야 합니다.)
3.  인사말을 건네면 에이전트와 대화가 시작됩니다!

## 주요 보안 설정
- **DM 정책**: 기본적으로 `pairing` 모드가 적용됩니다. 처음 대화 시 페어링 코드로 승인해 주세요.
- **공간(Spaces)**: 그룹 공간에서는 기본적으로 `@에이전트이름`을 멘션해야 응답합니다.

---
구체적인 보안 정책은 [보안 가이드](/gateway/security)를 참고하세요.
---
