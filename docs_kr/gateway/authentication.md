---
summary: "모델 인증: OAuth, API 키 및 설정 토큰 활용 가이드"
read_when:
  - 모델 인증이나 OAuth 만료 문제를 디버깅할 때
  - 인증 정보 저장 방식을 확인할 때
title: "인증"
---

# 인증 (Authentication)

OpenClaw는 모델 제공자를 위해 OAuth 및 API 키 인증 방식을 지원합니다. 일반적인 Anthropic 계정은 **API 키** 사용을 권장하며, Claude 유료 구독 사용자는 `claude setup-token`으로 생성된 토큰을 사용할 수 있습니다.

상세한 동작 원리는 [OAuth 컨셉 문서](/concepts/oauth)를 참고하십시오.

## 권장 경로: Anthropic API 키 설정

Anthropic을 직접 사용하는 경우 API 키 방식이 가장 안정적입니다.

1. Anthropic 콘솔에서 API 키를 생성합니다.
2. 게이트웨이가 실행되는 장비(호스트)에 키를 설정합니다.

```bash
export ANTHROPIC_API_KEY="..."
openclaw models status
```

3. 시스템 데몬(systemd/launchd)으로 실행 중이라면 `~/.openclaw/.env` 파일에 저장하는 것을 권장합니다:

```bash
cat >> ~/.openclaw/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

이후 게이트웨이를 재시작하고 `openclaw models status` 또는 `openclaw doctor` 명령어로 정상 작동 여부를 확인하세요. `openclaw onboard` 명령어를 통해서도 마법사 방식으로 설정할 수 있습니다.

## Anthropic: 설정 토큰 (구독 인증)

Claude Pro/Max 구독 사용자는 `claude setup-token` 흐름을 사용할 수 있습니다.

```bash
# 게이트웨이 기기에서 실행
claude setup-token
```

발급받은 토큰을 OpenClaw에 등록합니다:

```bash
openclaw models auth setup-token --provider anthropic
```

만약 다른 기기에서 생성한 토큰이라면 수동으로 붙여넣을 수 있습니다:

```bash
openclaw models auth paste-token --provider anthropic
```

## 에러 문제 해결: "This credential is only authorized for use with Claude Code..."

위와 같은 에러가 발생한다면, 해당 인증 정보는 Claude Code 전용이므로 일반 API 호출에 사용할 수 없습니다. 이 경우 Anthropic API 키를 별도로 발급받아 설정하십시오.

## API 키 로테이션 (Gateway)

OpenClaw는 속도 제한(Rate limit) 발생 시 자동으로 다른 API 키로 교체하여 재시도하는 기능을 지원합니다.

- **우선순위**:
  1. `OPENCLAW_LIVE_<PROVIDER>_KEY` (최우선 일시 오버라이드)
  2. `<PROVIDER>_API_KEYS` (여러 키 목록)
  3. `<PROVIDER>_API_KEY` (기본 키)
- **동작**: 429(속도 제한) 오류 발생 시에만 다음 키로 교체하며, 다른 일반 오류 시에는 즉시 종료됩니다.

상태 확인 및 트러블슈팅은 `openclaw models status`와 `openclaw doctor` 명령어를 활용하십시오.
---
