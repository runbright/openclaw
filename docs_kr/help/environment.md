---
summary: "OpenClaw가 환경 변수를 로드하는 방식과 적용 우선순위 가이드"
read_when:
  - 환경 변수가 어디서 어떤 순서로 로드되는지 확인해야 할 때
  - 게이트웨이에서 API 키가 누락되는 문제를 디버깅할 때
title: "환경 변수"
---

# 환경 변수 (Environment Variables)

OpenClaw는 여러 소스에서 환경 변수를 가져옵니다. 가장 중요한 원칙은 **"기존 값을 덮어쓰지 않는다"**는 것입니다.

## 우선순위 (높음 → 낮음)

1.  **프로세스 환경**: 게이트웨이 프로세스가 실행될 때 쉘이나 데몬으로부터 이미 가지고 있는 값.
2.  **현재 디렉토리의 `.env` 파일**: 실행 위치에 있는 설정 파일.
3.  **전역 `.env` 파일**: `~/.openclaw/.env` 위치에 있는 설정 파일.
4.  **설정 파일의 `env` 블록**: `openclaw.json` 파일 내에 정의된 값.
5.  **로그인 쉘(Shell) 임포트**: 쉘 설정 파일에서 누락된 키만 가져오는 방식.

## 설정 파일에서 환경 변수 정의

`openclaw.json` 파일의 `env` 블록에 환경 변수를 직접 작성할 수 있습니다:

```json5
{
  "env": {
    "OPENROUTER_API_KEY": "sk-or-...",
    "vars": {
      "GROQ_API_KEY": "gsk-..."
    }
  }
}
```

## 환경 변수 치환 사용

설정 파일의 문자열 값 내에서 `${변수명}` 형식을 사용하여 환경 변수를 참조할 수 있습니다:

```json5
{
  "models": {
    "providers": {
      "vercel-gateway": {
        "apiKey": "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

## 주요 경로 관련 환경 변수

| 변수명 | 용도 |
| :--- | :--- |
| `OPENCLAW_HOME` | 모든 내부 경로(세션, 자격 증명 등)의 기준이 되는 홈 디렉토리를 변경합니다. |
| `OPENCLAW_STATE_DIR` | 상태 저장 디렉토리(기본 `~/.openclaw`)를 변경합니다. |
| `OPENCLAW_CONFIG_PATH` | 설정 파일 경로(기본 `~/.openclaw/openclaw.json`)를 변경합니다. |

## 로깅 관련 환경 변수

| 변수명 | 용도 |
| :--- | :--- |
| `OPENCLAW_LOG_LEVEL` | 로그 레벨(`debug`, `trace` 등)을 강제로 지정합니다. 파일 및 콘솔 로그 모두에 적용됩니다. |

---
상세한 설정 옵션은 [게이트웨이 설정 가이드](/gateway/configuration)를 참고하세요.
---
