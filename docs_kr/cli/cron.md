---
summary: "`openclaw cron` CLI 참조 (백그라운드 작업 예약 및 실행)"
read_when:
  - 예약 작업 및 깨우기가 필요한 경우
  - cron 실행 및 로그를 디버깅하는 경우
title: "cron"
---

# `openclaw cron`

Gateway 스케줄러의 cron 작업을 관리합니다.

관련 항목:

- Cron 작업: [Cron jobs](/automation/cron-jobs)

팁: 전체 명령 목록은 `openclaw cron --help`를 실행하세요.

참고: 격리된 `cron add` 작업의 기본 전달 방식은 `--announce`입니다. 출력을 내부에 유지하려면 `--no-deliver`를 사용합니다. `--deliver`는 `--announce`의 더 이상 사용되지 않는 별칭으로 유지됩니다.

참고: 일회성(`--at`) 작업은 기본적으로 성공 후 삭제됩니다. 유지하려면 `--keep-after-run`을 사용합니다.

참고: 반복 작업은 이제 연속 오류 후 지수 재시도 백오프를 사용합니다 (30초 -> 1분 -> 5분 -> 15분 -> 60분). 다음 성공적인 실행 후 정상 일정으로 돌아갑니다.

참고: 보존/정리는 설정에서 제어됩니다:

- `cron.sessionRetention` (기본값 `24h`)은 완료된 격리 실행 세션을 정리합니다.
- `cron.runLog.maxBytes` + `cron.runLog.keepLines`는 `~/.openclaw/cron/runs/<jobId>.jsonl`을 정리합니다.

## 일반 편집

메시지를 변경하지 않고 전달 설정을 업데이트합니다:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

격리된 작업의 전달을 비활성화합니다:

```bash
openclaw cron edit <job-id> --no-deliver
```

특정 채널로 알림:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```
