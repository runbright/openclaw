---
summary: "단일 호스트에서 여러 OpenClaw 게이트웨이 실행 가이드 (격리, 포트 및 프로필 설정)"
read_when:
  - 같은 기기에서 두 개 이상의 게이트웨이를 실행해야 할 때
  - 게이트웨이별로 설정, 상태, 포트를 완전히 분리하고 싶을 때
title: "다중 게이트웨이 운영"
---

# 다중 게이트웨이 운영 (동일 호스트)

하나의 게이트웨이가 여러 채널과 에이전트를 동시에 관리할 수 있기 때문에 대부분의 사용자는 하나의 게이트웨이만 실행하면 됩니다. 하지만 **강력한 격리**가 필요하거나, 메인 봇 장애 시 복구용으로 사용할 **레스큐 봇(Rescue Bot)**이 필요한 경우 별도의 게이트웨이를 실행할 수 있습니다.

## 격리 필수 체크리스트

각 게이트웨이 인스턴스는 다음 항목들이 서로 겹치지 않도록 완전히 분리되어야 합니다:

1. **설정 파일 경로** (`OPENCLAW_CONFIG_PATH`)
2. **상태 데이터 디렉토리** (`OPENCLAW_STATE_DIR`): 세션, 인증 정보, 캐시 등
3. **에이전트 워크스페이스** (`agents.defaults.workspace`): 에이전트의 작업 공간
4. **게이트웨이 포트** (`gateway.port`): 중복되지 않는 포트 번호
5. **파생 포트** (브라우저 제어, 캔버스 등): 베이스 포트 충돌 방지

## 권장 방법: 프로필(`--profile`) 사용

프로필 기능을 사용하면 상태 디렉토리와 설정 파일 경로가 자동으로 구분되며, 서비스 이름도 프로필명에 맞춰 지정됩니다.

```bash
# 메인 게이트웨이 설정 및 실행
openclaw --profile main onboard
openclaw --profile main gateway --port 18789

# 레스큐(복구용) 게이트웨이 설정 및 실행
openclaw --profile rescue onboard
openclaw --profile rescue gateway --port 19789
```

각 프로필별로 별도의 시스템 서비스로 등록하여 관리할 수 있습니다:
```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## 포트 할당 규칙

충돌을 피하기 위해 베이스 포트 사이에 최소 20개 이상의 여유를 두는 것이 좋습니다.

- **게이트웨이 포트**: 기본 18789
- **브라우저 제어 포트**: 게이트웨이 포트 + 2 (내부 전용)
- **캔버스 호스트**: 게이트웨이 포트와 동일

예를 들어, 메인 봇이 18789를 쓴다면 다음 봇은 **19789**나 **20789**와 같이 멀리 떨어진 포트를 사용하는 것이 안전합니다.

## 수동 환경 변수 설정 (참고)

프로필 기능을 쓰지 않고 직접 환경 변수를 지정하여 실행할 수도 있습니다:

```bash
# 첫 번째 게이트웨이
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

# 두 번째 게이트웨이
OPENCLAW_CONFIG_PATH=~/.openclaw/second.json \
OPENCLAW_STATE_DIR=~/.openclaw-second \
openclaw gateway --port 19001
```

상세한 프로토콜 및 브라우저 관련 설정은 [영문 원본 문서](/gateway/multiple-gateways)를 참고하십시오.
---
