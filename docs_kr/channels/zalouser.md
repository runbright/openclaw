---
summary: "zca-cli(QR 로그인)를 통한 Zalo 개인 계정 연동 가이드 및 주의사항"
read_when:
  - OpenClaw에서 Zalo 개인 계정 연동을 설정할 때
  - Zalo 개인 계정 로그인이나 메시지 흐름을 디버깅할 때
title: "Zalo Personal"
---

# Zalo Personal (비공식)

**상태**: 실험적 지원. 이 연동 방식은 `zca-cli` 도구를 통해 **개인 Zalo 계정**을 자동화합니다.

> **경고**: 이 방식은 비공식 연동이며 사용 시 계정이 정지되거나 차단될 수 있습니다. 모든 책임은 사용자에게 있으며 신중하게 사용하십시오.

## 플러그인 설치

Zalo Personal은 별도의 플러그인 설치가 필요합니다:

```bash
openclaw plugins install @openclaw/zalouser
```

## 사전 준비: zca-cli

게이트웨이가 실행되는 장비의 `PATH`에 `zca` 바이너리가 있어야 합니다.

- 확인 방법: `zca --version`
- 설치 가이드: `extensions/zalouser/README.md` 또는 zca-cli 공식 문서를 참고하십시오.

## 빠른 설정 가이드

1. **로그인 (QR 코드)**:
   - 게이트웨이 장비에서 `openclaw channels login --channel zalouser` 명령어를 실행합니다.
   - 터미널에 나타나는 QR 코드를 모바일 Zalo 앱으로 스캔합니다.
2. **OpenClaw 설정 (`openclaw.json`)**:

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

3. **게이트웨이 실행**: 설정 완료 후 게이트웨이를 시작하고, 첫 연락 시 페어링 코드로 승인합니다.

## 작동 방식 및 명칭

- 이 채널 ID는 `zalouser`입니다. 이는 공식 봇 API와 구분하여 **개인 사용자 계정**을 자동화함을 명확히 하기 위함입니다.
- `zca listen`을 통해 메시지를 수신하고 `zca msg`를 통해 답장(텍스트, 미디어, 링크 등)을 보냅니다.

## 접근 제어 (Access Control)

- **DM**: 기본적으로 `pairing` 모드입니다. `openclaw pairing approve zalouser <코드>`로 승인합니다.
- **그룹**: 기본적으로 허용(`open`)되어 있지만, `groupPolicy: "allowlist"` 설정을 통해 특정 그룹 ID나 이름만 허용하도록 제한할 수 있습니다.

## 주의사항

- **메시지 분할**: 텍스트는 약 2,000자 단위로 분할되어 전송됩니다.
- **스트리밍**: 현재는 기술적 제한으로 인해 스트리밍 기능이 차단되어 있습니다.
- **로그인 유지**: 로그인 세션이 끊긴 경우 `openclaw channels status --probe`로 상태를 확인하고 다시 로그인하십시오.

상세 설정값은 [게이트웨이 설정 참고 문서](/gateway/configuration)를 확인하세요.
---
