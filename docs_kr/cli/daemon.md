---
summary: "`openclaw daemon` CLI 참조 (Gateway 서비스 관리의 레거시 별칭)"
read_when:
  - 스크립트에서 아직 `openclaw daemon ...`을 사용하는 경우
  - 서비스 수명 주기 명령이 필요한 경우 (install/start/stop/restart/status)
title: "daemon"
---

# `openclaw daemon`

Gateway 서비스 관리 명령의 레거시 별칭입니다.

`openclaw daemon ...`은 `openclaw gateway ...` 서비스 명령과 동일한 서비스 제어 인터페이스에 매핑됩니다.

## 사용법

```bash
openclaw daemon status
openclaw daemon install
openclaw daemon start
openclaw daemon stop
openclaw daemon restart
openclaw daemon uninstall
```

## 하위 명령

- `status`: 서비스 설치 상태를 표시하고 Gateway 상태를 프로브합니다
- `install`: 서비스를 설치합니다 (`launchd`/`systemd`/`schtasks`)
- `uninstall`: 서비스를 제거합니다
- `start`: 서비스를 시작합니다
- `stop`: 서비스를 중지합니다
- `restart`: 서비스를 다시 시작합니다

## 공통 옵션

- `status`: `--url`, `--token`, `--password`, `--timeout`, `--no-probe`, `--deep`, `--json`
- `install`: `--port`, `--runtime <node|bun>`, `--token`, `--force`, `--json`
- 수명 주기 (`uninstall|start|stop|restart`): `--json`

## 권장 사항

최신 문서와 예시는 [`openclaw gateway`](/cli/gateway)를 사용하세요.
