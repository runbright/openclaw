---
summary: "OpenClaw 설치 가이드 — 스크립트, npm/pnpm, 소스 코드 빌드, Docker 등 다양한 방법 안내"
read_when:
  - 시작하기 퀵스타트 외에 다른 설치 방법이 필요할 때
  - 클라우드 플랫폼에 배포하고 싶을 때
  - 업데이트, 이전 또는 삭제 방법이 궁금할 때
title: "설치"
---

# 설치 (Install)

[시작하기](/start/getting-started) 가이드를 이미 완료하셨나요? 그렇다면 모든 준비가 끝난 것입니다. 이 페이지는 대체 설치 방법, 플랫폼별 지침 및 유지 관리에 대한 정보를 담고 있습니다.

## 시스템 요구 사항

- **[Node 22 이상](/install/node)** (설치 스크립트 실행 시 미설치된 경우 자동으로 설치를 시도합니다)
- macOS, Linux 또는 Windows
- 소스 코드에서 직접 빌드하는 경우 `pnpm` 필요

<Note>
Windows 사용자의 경우 [WSL2](https://learn.microsoft.com/ko-kr/windows/wsl/install) 환경에서 OpenClaw를 실행하는 것을 강력히 권장합니다.
</Note>

## 설치 방법

<Tip>
**설치 스크립트**를 사용하는 것이 가장 권장되는 방법입니다. Node.js 감지, 설치, 초기 설정(Onboarding)을 한 번에 처리합니다.
</Tip>

### 1. 설치 스크립트 실행 (권장)

터미널에서 다음 명령어를 실행하면 즉시 설치가 시작됩니다.

<Tabs>
  <Tab title="macOS / Linux / WSL2">
    ```bash
    curl -fsSL https://openclaw.ai/install.sh | bash
    ```
  </Tab>
  <Tab title="Windows (PowerShell)">
    ```powershell
    iwr -useb https://openclaw.ai/install.ps1 | iex
    ```
  </Tab>
</Tabs>

### 2. npm / pnpm 사용

Node 22 이상이 이미 설치되어 있고 직접 관리하고 싶다면:

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

### 3. 소스 코드에서 빌드 (개발자용)

직접 코드를 수정하거나 최신 개발 버전을 사용하고 싶다면:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
pnpm link --global
openclaw onboard --install-daemon
```

## 설치 후 확인 사항

설치가 잘 되었는지 확인하려면 다음 명령어들을 실행해 보세요:

```bash
openclaw doctor         # 설정 이슈 확인
openclaw status         # 게이트웨이 상태 확인
openclaw dashboard      # 웹 대시보드 열기
```

## 업데이트 및 삭제

<CardGroup cols={3}>
  <Card title="업데이트" href="/install/updating" icon="refresh-cw">
    최신 버전 유지 방법
  </Card>
  <Card title="이전하기" href="/install/migrating" icon="arrow-right">
    새로운 컴퓨터로 옮기기
  </Card>
  <Card title="삭제하기" href="/install/uninstall" icon="trash-2">
    완전 삭제 방법
  </Card>
</CardGroup>

---
설치 중 `openclaw` 명령어를 찾을 수 없다는 오류가 발생하면, npm 전역 바이너리 경로가 시스템 `PATH`에 추가되어 있는지 확인하세요.
---
