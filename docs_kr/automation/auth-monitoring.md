---
summary: "AI 모델 제공업체의 OAuth 인증 만료 모니터링 가이드"
read_when:
  - 인증 만료 알림이나 자동 확인 설정을 하고 싶을 때
  - 클로드(Claude)나 코덱스(Codex)의 인증 갱신 상태를 자동화하고 싶을 때
title: "인증 모니터링"
---

# 인증 모니터링 (Auth Monitoring)

OpenClaw는 `openclaw models status` 명령어를 통해 OAuth 인증 만료 상태를 확인할 수 있습니다. 이를 활용해 자동화된 알림 시스템을 구축할 수 있습니다.

## 추천 방법: CLI 체크 (이식성 높음)

터미널에서 다음 명령어를 실행하면 인증 상태를 즉시 확인할 수 있습니다:

```bash
openclaw models status --check
```

**종료 코드(Exit Codes):**
- `0`: 정상(OK)
- `1`: 인증 만료 또는 자격 증명 누락
- `2`: 곧 만료 예정 (24시간 이내)

이 명령어는 크론(cron) 작업이나 스크립트 내에서 조건식으로 활용하기 좋습니다.

## 보조 스크립트 (선택 사항)

`scripts/` 디렉토리에는 특정 환경(예: 휴대전화 Termux, 시스템 타이머 등)에서 사용하기 좋은 스크립트들이 포함되어 있습니다.

- **`scripts/auth-monitor.sh`**: 주기적으로 인증 상태를 확인하고 알림(ntfy 등)을 보냅니다.
- **`scripts/mobile-reauth.sh`**: 휴대전화에서 SSH를 통해 접속했을 때 가이드에 따라 재인증을 진행합니다.
- **`scripts/termux-quick-auth.sh`**: 안드로이드 Termux 환경에서 원터치로 인증 상태를 확인하고 인증 URL을 엽니다.

휴대전화 자동화나 별도의 시스템 타이머가 필요하지 않다면 이 스크립트들은 건너뛰어도 무방합니다.

---
인증 권한에 대한 일반적인 내용은 [인증 프로필 가이드](/concepts/model-providers#인증-프로필)를 참고하세요.
---
