---
summary: "OpenClaw 문제 해결을 위한 증상별 점검 도구 및 가이드"
read_when:
  - OpenClaw가 제대로 작동하지 않아 빠른 해결 방법이 필요할 때
  - 상세 문서를 분석하기 전 시스템 상태를 빠르게 진단하고 싶을 때
title: "문제 해결"
---

# 문제 해결 (Troubleshooting)

시스템에 문제가 생겼을 때 가장 먼저 확인해야 할 사항들입니다.

## 60초 요약 진단

터미널에서 다음 명령어들을 순서대로 실행하여 상태를 점검하세요:

```bash
openclaw status
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

**정상적인 경우의 결과:**
- `openclaw status`: 설정된 채널들이 보이며 인증 오류가 없음.
- `openclaw gateway status`: `Runtime: running` 및 `RPC probe: ok` 표시됨.
- `openclaw doctor`: 설정이나 서비스에 치명적인 오류가 발견되지 않음.
- `openclaw channels status --probe`: 채널 상태가 `connected` 또는 `ready`로 표시됨.
- `openclaw logs --follow`: 실시간 로그가 흐르며 'Fatal' 오류가 반복되지 않음.

## 상황별 대처법 (FAQ)

### 1. 에이전트가 대답을 하지 않아요
- **체크리스트:**
  - `openclaw gateway status`에서 실행 중인지 확인하세요.
  - `channels status --probe`에서 메신저 채널이 연결되어 있는지 확인하세요.
  - 사용자가 **승인(Pairing)**되었는지 확인하세요. (`openclaw pairing list`)
- **로그 확인:** 로그에 `pairing request`나 `blocked`라는 단어가 보인다면 승인 문제일 가능성이 높습니다.

### 2. 대시보드나 제어 UI가 실행되지 않아요
- **체크리스트:**
  - `openclaw gateway status`에서 대시보드 주소(`http://...`)가 나오는지 확인하세요.
  - 네트워크 방화벽이 해당 포트를 차단하고 있지 않은지 확인하세요.
- **로그 확인:** `unauthorized` 오류가 반복된다면 토큰이나 비밀번호가 틀렸을 수 있습니다.

### 3. 게이트웨이가 시작되지 않아요
- **체크리스트:**
  - `EADDRINUSE` 오류가 발생한다면 이미 다른 게이트웨이가 작동 중이거나 다른 앱이 포트를 사용 중인 것입니다.
  - `openclaw doctor`를 실행하여 누락된 설정이 있는지 확인하세요.

### 4. 도구가 실행되지 않아요 (브라우저, 실행 등)
- **체크리스트:**
  - **브라우저**: 크롬이나 지원 브라우저가 설치되어 있고 경로가 맞는지 확인하세요. (`openclaw browser status`)
  - **명령어 실행**: `exec` 도구 권한이 승인되었는지 확인하세요.

---
추가적인 도움이 필요하다면 [Discord 커뮤니티](https://discord.gg/openclaw)에 문의해 주세요.
---
