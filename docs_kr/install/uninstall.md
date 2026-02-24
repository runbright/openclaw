---
summary: "OpenClaw 완전 삭제 가이드: CLI, 서비스 설정, 상태 데이터 및 워크스페이스 제거 방법"
read_when:
  - 컴퓨터에서 OpenClaw를 완전히 제거하고 싶을 때
  - 프로그램 삭제 후에도 백그라운드 서비스가 계속 실행될 때
title: "삭제"
---

# 삭제 (Uninstall)

컴퓨터에서 OpenClaw를 완전히 제거하는 방법입니다.

## 추천 방법: 통합 삭제 명령어 사용

프로그램이 아직 설치되어 있다면, 내장된 삭제 기능을 사용하는 것이 가장 깔끔합니다.

```bash
openclaw uninstall
```

이 명령어는 백그라운드 서비스를 중지하고 등록을 해제한 뒤, 필요에 따라 설정 파일까지 함께 삭제할지 묻습니다.

## 수동 삭제 방법

명령어를 사용할 수 없거나 직접 삭제하고 싶다면 다음 순서를 따르세요:

### 1단계: 백그라운드 서비스 중지 및 삭제
- **macOS**: `launchctl bootout gui/$UID/bot.molt.gateway` 실행 후 `~/Library/LaunchAgents/bot.molt.gateway.plist` 파일 삭제
- **Linux**: `systemctl --user disable --now openclaw-gateway.service` 실행 후 `~/.config/systemd/user/openclaw-gateway.service` 파일 삭제

### 2단계: 데이터 및 설정 파일 삭제
OpenClaw의 데이터는 기본적으로 홈 디렉토리 아래에 숨겨져 있습니다.

```bash
rm -rf ~/.openclaw
```
*주의: 이 폴더를 삭제하면 모든 대화 세션 기록, 메모리, API 키 정보가 사라집니다.*

### 3단계: 프로그램(CLI) 삭제
설치 방식에 맞게 프로그램을 제거합니다.

```bash
npm rm -g openclaw
# 또는
pnpm remove -g openclaw
```

### 4단계: 워크스페이스 삭제 (선택 사항)
에이전트의 작업 공간(지침 파일들)을 삭제하려면 다음 폴더를 제거하세요.

```bash
rm -rf ~/.openclaw/workspace
```

---
새로운 기기로 설정을 옮기고 싶다면 [이전하기 가이드](/install/migrating)를 확인해 보세요.
---
