---
summary: "격리 또는 iMessage가 필요할 때 샌드박스된 macOS VM (로컬 또는 호스팅)에서 OpenClaw 실행"
read_when:
  - 메인 macOS 환경에서 OpenClaw을 격리하고 싶을 때
  - 샌드박스에서 iMessage 통합 (BlueBubbles)을 원할 때
  - 클론 가능한 재설정 가능한 macOS 환경을 원할 때
  - 로컬과 호스팅 macOS VM 옵션을 비교하고 싶을 때
title: "macOS VM"
---

# macOS VM에서 OpenClaw 실행 (샌드박싱)

## 권장 기본값 (대부분의 사용자)

- **소규모 Linux VPS**: 상시 가동 게이트웨이와 저비용을 위해. [VPS 호스팅](/vps)을 참조하세요.
- **전용 하드웨어** (Mac mini 또는 Linux 박스): 완전한 제어와 브라우저 자동화를 위한 **주거용 IP**를 원할 때. 많은 사이트가 데이터 센터 IP를 차단하므로 로컬 브라우징이 더 잘 작동하는 경우가 많습니다.
- **하이브리드:** 저렴한 VPS에 게이트웨이를 두고, 브라우저/UI 자동화가 필요할 때 Mac을 **노드**로 연결. [노드](/nodes) 및 [게이트웨이 원격](/gateway/remote)을 참조하세요.

macOS 전용 기능 (iMessage/BlueBubbles)이 특별히 필요하거나 일상적인 Mac과 엄격하게 격리하고 싶을 때 macOS VM을 사용하세요.

## macOS VM 옵션

### Apple Silicon Mac에서 로컬 VM (Lume)

[Lume](https://cua.ai/docs/lume)를 사용하여 기존 Apple Silicon Mac에서 샌드박스된 macOS VM으로 OpenClaw을 실행합니다.

이것으로 얻을 수 있는 것:

- 격리된 완전한 macOS 환경 (호스트는 깨끗하게 유지)
- BlueBubbles를 통한 iMessage 지원 (Linux/Windows에서는 불가능)
- VM 클론으로 즉시 초기화
- 추가 하드웨어나 클라우드 비용 없음

### 호스팅 Mac 프로바이더 (클라우드)

클라우드에서 macOS를 원한다면 호스팅 Mac 프로바이더도 사용 가능합니다:

- [MacStadium](https://www.macstadium.com/) (호스팅 Mac)
- 다른 호스팅 Mac 벤더도 사용 가능; 해당 VM + SSH 문서를 참조하세요

macOS VM에 SSH 접근이 가능하면 아래 6단계부터 계속하세요.

---

## 빠른 경로 (Lume, 경험 있는 사용자)

1. Lume 설치
2. `lume create openclaw --os macos --ipsw latest`
3. 설정 도우미 완료, 원격 로그인 (SSH) 활성화
4. `lume run openclaw --no-display`
5. SSH 접속, OpenClaw 설치, 채널 구성
6. 완료

---

## 필요한 것 (Lume)

- Apple Silicon Mac (M1/M2/M3/M4)
- 호스트에 macOS Sequoia 이상
- VM당 약 60GB 여유 디스크 공간
- 약 20분

---

## 1) Lume 설치

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/trycua/cua/main/libs/lume/scripts/install.sh)"
```

`~/.local/bin`이 PATH에 없다면:

```bash
echo 'export PATH="$PATH:$HOME/.local/bin"' >> ~/.zshrc && source ~/.zshrc
```

확인:

```bash
lume --version
```

문서: [Lume 설치](https://cua.ai/docs/lume/guide/getting-started/installation)

---

## 2) macOS VM 생성

```bash
lume create openclaw --os macos --ipsw latest
```

macOS를 다운로드하고 VM을 생성합니다. VNC 창이 자동으로 열립니다.

참고: 인터넷 연결 속도에 따라 다운로드에 시간이 걸릴 수 있습니다.

---

## 3) 설정 도우미 완료

VNC 창에서:

1. 언어와 지역 선택
2. Apple ID 건너뛰기 (또는 나중에 iMessage를 위해 로그인)
3. 사용자 계정 생성 (사용자 이름과 비밀번호를 기억하세요)
4. 모든 선택적 기능 건너뛰기

설정 완료 후 SSH를 활성화합니다:

1. 시스템 설정 → 일반 → 공유 열기
2. "원격 로그인" 활성화

---

## 4) VM의 IP 주소 확인

```bash
lume get openclaw
```

IP 주소를 찾으세요 (보통 `192.168.64.x`).

---

## 5) VM에 SSH 접속

```bash
ssh youruser@192.168.64.X
```

`youruser`를 생성한 계정으로, IP를 VM의 IP로 바꾸세요.

---

## 6) OpenClaw 설치

VM 내부에서:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

온보딩 프롬프트에 따라 모델 프로바이더 (Anthropic, OpenAI 등)를 설정하세요.

---

## 7) 채널 구성

설정 파일을 편집합니다:

```bash
nano ~/.openclaw/openclaw.json
```

채널을 추가합니다:

```json
{
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+15551234567"]
    },
    "telegram": {
      "botToken": "YOUR_BOT_TOKEN"
    }
  }
}
```

그런 다음 WhatsApp에 로그인 (QR 스캔):

```bash
openclaw channels login
```

---

## 8) VM을 헤드리스로 실행

VM을 중지하고 디스플레이 없이 재시작:

```bash
lume stop openclaw
lume run openclaw --no-display
```

VM이 백그라운드에서 실행됩니다. OpenClaw의 데몬이 게이트웨이를 계속 실행합니다.

상태를 확인하려면:

```bash
ssh youruser@192.168.64.X "openclaw status"
```

---

## 보너스: iMessage 통합

macOS에서 실행하는 핵심 기능입니다. [BlueBubbles](https://bluebubbles.app)를 사용하여 OpenClaw에 iMessage를 추가하세요.

VM 내부에서:

1. bluebubbles.app에서 BlueBubbles 다운로드
2. Apple ID로 로그인
3. Web API를 활성화하고 비밀번호 설정
4. BlueBubbles 웹훅을 게이트웨이로 설정 (예: `https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`)

OpenClaw 설정에 추가:

```json
{
  "channels": {
    "bluebubbles": {
      "serverUrl": "http://localhost:1234",
      "password": "your-api-password",
      "webhookPath": "/bluebubbles-webhook"
    }
  }
}
```

게이트웨이를 재시작하세요. 이제 에이전트가 iMessage를 보내고 받을 수 있습니다.

전체 설정 세부 정보: [BlueBubbles 채널](/channels/bluebubbles)

---

## 골든 이미지 저장

추가 커스터마이즈 전에 깨끗한 상태를 스냅샷하세요:

```bash
lume stop openclaw
lume clone openclaw openclaw-golden
```

언제든 초기화:

```bash
lume stop openclaw && lume delete openclaw
lume clone openclaw-golden openclaw
lume run openclaw --no-display
```

---

## 24/7 실행

VM을 계속 실행하려면:

- Mac을 전원에 연결
- 시스템 설정 → 에너지 절약에서 잠자기 비활성화
- 필요 시 `caffeinate` 사용

진정한 상시 가동을 위해 전용 Mac mini 또는 소규모 VPS를 고려하세요. [VPS 호스팅](/vps)을 참조하세요.

---

## 문제 해결

| 문제                     | 해결 방법                                                                      |
| ------------------------ | ------------------------------------------------------------------------------ |
| VM에 SSH 접속 불가       | VM의 시스템 설정에서 "원격 로그인"이 활성화되어 있는지 확인                      |
| VM IP가 표시되지 않음    | VM이 완전히 부팅될 때까지 기다리고 `lume get openclaw`을 다시 실행               |
| Lume 명령을 찾을 수 없음 | `~/.local/bin`을 PATH에 추가                                                    |
| WhatsApp QR 스캔 안됨    | `openclaw channels login` 실행 시 호스트가 아닌 VM에 로그인되어 있는지 확인      |

---

## 관련 문서

- [VPS 호스팅](/vps)
- [노드](/nodes)
- [게이트웨이 원격](/gateway/remote)
- [BlueBubbles 채널](/channels/bluebubbles)
- [Lume 빠른 시작](https://cua.ai/docs/lume/guide/getting-started/quickstart)
- [Lume CLI 참조](https://cua.ai/docs/lume/reference/cli-reference)
- [무인 VM 설정](https://cua.ai/docs/lume/guide/fundamentals/unattended-setup) (고급)
- [Docker 샌드박싱](/install/docker) (대안적 격리 접근 방식)
