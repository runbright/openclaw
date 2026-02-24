---
summary: "크론 및 하트비트 스케줄링 및 결과 전송 문제 해결 가이드"
read_when:
  - 크론 작업이 실행되지 않았을 때
  - 하트비트가 조용하거나 건너뛰어지는 것 같을 때
  - 예약된 대화가 채팅창에 나타나지 않을 때
title: "자동화 문제 해결"
---

# 자동화 문제 해결 (Automation Troubleshooting)

스케줄러(`cron`)나 주기적 실행(`heartbeat`) 관련 문제가 발생했을 때 확인해야 할 사항들입니다.

## 상태 확인 명령어 (Command Ladder)

문제가 생기면 아래 명령어들을 순서대로 실행하여 원인을 파악하세요:

```bash
openclaw status                # 전체 상태 확인
openclaw gateway status        # 게이트웨이 및 RPC 상태 확인
openclaw logs --follow         # 실시간 로그 확인
openclaw cron status           # 크론 스케줄러 상태 확인
openclaw cron list             # 등록된 작업 목록 확인
openclaw system heartbeat last # 마지막 하트비트 실행 결과 확인
```

## 크론 작업이 실행되지 않을 때 (Cron not firing)

1.  **스케줄러 활성화 확인**: `openclaw cron status` 에서 `enabled` 상태인지 확인하세요.
2.  **실행 기록 확인**: `openclaw cron runs --id <작업ID>` 명령어로 이전 실행 시도가 있었는지, 실패했다면 그 이유가 무엇인지 확인합니다.
3.  **로그 확인**: `openclaw logs --follow`를 켜둔 상태에서 작업 시간이 되었을 때 오류 메시지가 뜨는지 살핍니다.

## 실행은 되었는데 메시지가 안 올 때 (Cron fired but no delivery)

1.  **전송 모드 확인**: 작업 설정 중 `delivery mode`가 `none`으로 되어 있지 않은지 확인하세요. (`announce`여야 메시지가 전송됩니다.)
2.  **채널 상태 확인**: `openclaw channels status --probe` 명령어로 해당 채널(WhatsApp, Telegram 등)이 잘 연결되어 있는지 확인합니다.
3.  **대상(Target) 확인**: 메시지를 보낼 전화번호나 채널ID가 정확하게 설정되었는지 확인하세요.

## 하트비트가 작동하지 않을 때 (Heartbeat suppressed/skipped)

1.  **활동 시간 확인**: `activeHours` 설정 때문에 현재 시간이 "취침 시간(휴식 시간)"으로 취급되어 건너뛰어지고 있을 수 있습니다.
2.  **메인 세션 상태**: 현재 에이전트가 다른 작업을 수행 중이어서 하트비트가 밀려있을 수 있습니다.
3.  **내용 유무**: `HEARTBEAT.md` 파일에 에이전트가 수행할 구체적인 지침이 없으면 실행을 건너뜁니다.

## 시간대(Timezone) 관련 주의사항

- 크론 작업 예약 시 `--tz`로 시간대를 명시하지 않으면 **서버(Gateway host)의 시간대**를 기준으로 작동합니다.
- `HEARTBEAT.md`의 활동 시간(`activeHours`) 역시 설정된 시간대를 따릅니다. 서버를 해외 VPS로 옮겼다면 시간대 설정을 다시 확인해 보세요.

---
구체적인 실행 방식 차이는 [크론 vs 하트비트 비교](/automation/cron-vs-heartbeat)를 참고하세요.
---
