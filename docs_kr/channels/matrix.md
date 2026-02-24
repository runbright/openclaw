---
summary: "매트릭스(Matrix) 연동 상태, 분산형 메시징 구성 및 암호화 설정 가이드"
read_when:
  - 매트릭스 계정을 OpenClaw에 연결하고 싶을 때
  - 종단간 암호화(E2EE) 설정이나 룸(Room) 권한 설정을 확인해야 할 때
title: "매트릭스"
---

# 매트릭스 (Matrix / Plugin)

매트릭스는 개방형, 분산형 메시징 프로토콜입니다. OpenClaw는 플러그인을 통해 매트릭스 홈서버에 사용자로 접속하여 대화할 수 있습니다.

## 중요: 플러그인 설치 필수
매트릭스 연동 기능은 핵심 설치본에 포함되어 있지 않습니다. 아래 명령어로 먼저 플러그인을 설치해야 합니다:
```bash
openclaw plugins install @openclaw/matrix
```

## 빠른 설정 방법

1.  **계정 생성**: 임의의 매트릭스 홈서버(예: matrix.org)에서 봇으로 사용할 계정을 만듭니다.
2.  **액세스 토큰(Access Token) 발급**: 홈서버 로그인 API를 호출하거나 클라이언트(Element 등) 설정에서 토큰을 확인합니다.
3.  **OpenClaw 설정 (`openclaw.json`)**:
    ```json5
    {
      "channels": {
        "matrix": {
          "enabled": true,
          "homeserver": "https://matrix.org",
          "accessToken": "syt_...",
          "dm": { "policy": "pairing" }
        }
      }
    }
    ```
4.  **대화 시작**: 봇 계정(@봇:서버)에게 메시지를 보내거나 룸(Room)에 초대하세요.

## 주요 기능
- **종단간 암호화 (E2EE)**: `encryption: true` 설정을 통해 암호화된 대화방에서도 안전하게 대화할 수 있습니다. (최초 연결 시 다른 기기에서의 승인이 필요합니다.)
- **분산형 네트워크**: 특정 서버에 종속되지 않고 원하는 홈서버 어디든 연결 가능합니다.
- **다양한 메시지 타입**: 텍스트뿐만 아니라 스레드(Thread), 이모지 반응(Reaction), 위치 정보 등을 지원합니다.

## 주의사항
- **암호화 모듈**: 암호화 기능을 쓰려면 서버에 필요한 라이브러리(Rust crypto SDK)가 설치되어야 할 수 있습니다. `openclaw doctor` 명령어로 상태를 확인하세요.
- **룸 허용 목록**: 보안을 위해 기본적으로 허용된 룸(`groups`)에서만 작동하도록 설정하는 것을 권장합니다.

---
연결 중 문제가 발생하면 [채널 문제 해결](/channels/troubleshooting) 문서를 확인하세요.
---
