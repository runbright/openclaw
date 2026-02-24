---
summary: "`openclaw doctor` CLI 참조 (상태 점검 + 안내 수리)"
read_when:
  - 연결/인증 문제가 있어 안내에 따른 수리가 필요한 경우
  - 업데이트 후 정상 동작을 확인하려는 경우
title: "doctor"
---

# `openclaw doctor`

Gateway와 채널에 대한 상태 점검 + 빠른 수리입니다.

관련 항목:

- 문제 해결: [Troubleshooting](/gateway/troubleshooting)
- 보안 감사: [Security](/gateway/security)

## 예시

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

참고사항:

- 대화형 프롬프트(키체인/OAuth 수리 등)는 stdin이 TTY이고 `--non-interactive`가 설정되지 **않은** 경우에만 실행됩니다. 헤드리스 실행(cron, Telegram, 터미널 없음)에서는 프롬프트가 건너뛰어집니다.
- `--fix` (`--repair`의 별칭)는 `~/.openclaw/openclaw.json.bak`에 백업을 기록하고 알 수 없는 설정 키를 제거하며, 각 제거 항목을 나열합니다.
- 상태 무결성 검사는 이제 세션 디렉토리에서 고아 대화 기록 파일을 감지하고, 안전하게 공간을 확보하기 위해 `.deleted.<timestamp>`로 아카이브할 수 있습니다.

## macOS: `launchctl` 환경 재정의

이전에 `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (또는 `...PASSWORD`)를 실행한 경우, 해당 값이 설정 파일을 재정의하여 지속적인 "인증 실패" 오류를 유발할 수 있습니다.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
