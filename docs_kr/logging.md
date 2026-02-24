---
summary: "OpenClaw 로깅 개요: 파일 로그, 콘솔 출력, 실시간 로그 확인 및 제어 UI 가이드"
read_when:
  - 로깅에 대한 초보자용 가이드가 필요할 때
  - 로그 레벨이나 형식을 설정하고 싶을 때
  - 트러블슈팅을 위해 로그 위치를 빨리 찾고 싶을 때
title: "로깅"
---

# 로깅 (Logging)

OpenClaw는 두 곳에 로그를 기록합니다:
- **파일 로그** (JSON 라인 형식): 게이트웨이에 의해 기록됩니다.
- **콘솔 출력**: 터미널 및 제어 UI에 표시됩니다.

## 로그 파일 위치

기본적으로 게이트웨이는 다음 경로에 일자별 로그 파일을 생성합니다:
`/tmp/openclaw/openclaw-YYYY-MM-DD.log`

경로를 변경하려면 `~/.openclaw/openclaw.json`에서 설정하세요:
```json
{
  "logging": {
    "file": "/원하는/경로/openclaw.log"
  }
}
```

## 로그 확인 방법

### 1. CLI 실시간 확인 (권장)
다음 명령어를 터미널에 입력하면 실시간으로 로그가 표시됩니다:
```bash
openclaw logs --follow
```
- `--json`: 각 로그를 낱개의 JSON 객체로 출력합니다.
- `--no-color`: 색상 강조를 끕니다.

### 2. 제어 UI (웹 대시보드)
제어 UI의 **Logs** 탭에서 동일한 내용을 확인할 수 있습니다.

### 3. 특정 채널 로그만 보기
WhatsApp이나 Telegram의 활동만 필터링해서 보려면:
```bash
openclaw channels logs --channel whatsapp
```

## 로깅 설정

모든 로깅 설정은 `openclaw.json`의 `logging` 섹션에서 관리합니다.

```json5
{
  "logging": {
    "level": "info", // 파일 로그 레벨 (info, debug, trace 등)
    "consoleLevel": "info", // 콘솔 출력 레벨
    "consoleStyle": "pretty", // 'pretty' (컬러), 'compact' (간략), 'json'
    "redactSensitive": "tools" // 민감한 정보(토큰 등) 가리기 활성
  }
}
```

### 로그 레벨 임시 변경
명령어 실행 시 `--log-level debug` 옵션을 붙이면 설정 파일을 수정하지 않고도 상세 로그를 볼 수 있습니다:
`openclaw --log-level debug gateway run`

## 진단 및 OpenTelemetry

OpenClaw는 모델 실행 정보나 메시지 흐름을 기계가 읽기 좋은 형식(Diagnostics)으로 생성할 수 있으며, 이를 데이터 분석 도구(OpenTelemetry 등)로 내보낼 수 있습니다.

상세한 진단 플래그 설정이나 원격 서버로의 로그 전송 방법은 [영문 원본 문서](/logging)를 참고하십시오.

## 문제 해결 팁
- **로그가 비어있나요?** 게이트웨이가 정상 실행 중인지 확인하고, `logging.file` 경로에 파일 쓰기 권한이 있는지 체크하세요.
- **더 자세한 정보가 필요하신가요?** 로그 레벨을 `debug` 또는 `trace`로 높여보세요.
---
