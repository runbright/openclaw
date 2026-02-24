---
summary: "OpenClaw Docker 기반 설치 및 샌드박스 설정 가이드"
read_when:
  - 시스템에 프로그램을 직접 설치하지 않고 컨테이너 환경에서 실행하고 싶을 때
  - 에이전트의 작업 환경을 격리(Sandboxing)하고 싶을 때
title: "Docker 설치"
---

# Docker 설치 및 설정 (Docker)

Docker 사용은 **선택 사항**입니다. 시스템 환경을 깨끗하게 유지하고 싶거나, 에이전트가 실행하는 명령어가 내 컴퓨터에 영향을 주지 않도록 격리하고 싶을 때 사용합니다.

## 시스템 요구 사항
- Docker Desktop 또는 Docker Engine
- Docker Compose v2 이상

## 1. 게이트웨이 컨테이너 실행

전체 OpenClaw 시스템을 Docker 컨테이너로 실행하는 방법입니다.

### 빠른 시작 (권장)
소스 코드를 내려받은 폴더에서 다음 스크립트를 실행하세요:

```bash
./docker-setup.sh
```

**이 스크립트가 수행하는 작업:**
- OpenClaw 이미지를 빌드합니다.
- 초기 설정(Onboarding) 프로세스를 시작합니다.
- Docker Compose를 통해 서비스를 실행합니다.
- 접속에 필요한 인증 토큰을 생성합니다.

설치가 완료되면 브라우저에서 `http://127.0.0.1:18789/`에 접속하여 대시보드를 사용할 수 있습니다.

## 2. 에이전트 도구 샌드박스 (Sandboxing)

게이트웨이는 내 컴퓨터(Host)에서 실행하되, 에이전트가 터미널 명령을 내리거나 파일을 수정하는 작업만 별도의 Docker 컨테이너 안에서 안전하게 처리하도록 설정할 수 있습니다.

### 설정 방법
`openclaw.json` 설정 파일에서 샌드박스를 활성화합니다:

```json5
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // 메인 대화 이외의 세션은 샌드박스 사용
        "docker": {
          "image": "openclaw-sandbox:bookworm-slim"
        }
      }
    }
  }
}
```

샌드박스 전용 이미지를 만들려면 다음 스크립트를 실행하세요:
```bash
scripts/sandbox-setup.sh
```

## 문제 해결 (Troubleshooting)

- **이미지를 찾을 수 없음**: `scripts/sandbox-setup.sh`를 실행하여 샌드박스 이미지를 먼저 빌드해야 합니다.
- **권한 오류 (EACCES)**: 컨테이너 내부 사용자와 호스트 파일의 소유자(UID)가 일치하지 않을 때 발생합니다. 호스트의 워크스페이스 폴더 권한을 확인하세요.

---
상세한 보안 격리 설정은 [샌드박스 심화 가이드](/gateway/sandboxing)를 참고하세요.
---
