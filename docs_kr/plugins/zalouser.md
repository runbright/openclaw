---
summary: "Zalo Personal 플러그인: zca-cli를 통한 QR 로그인 + 메시징 (플러그인 설치 + 채널 설정 + CLI + 도구)"
read_when:
  - OpenClaw에서 Zalo Personal (비공식) 지원을 원하는 경우
  - zalouser 플러그인을 구성하거나 개발하는 경우
title: "Zalo Personal 플러그인"
---

# Zalo Personal (플러그인)

`zca-cli`를 사용하여 일반 Zalo 사용자 계정을 자동화하는 플러그인을 통한 OpenClaw의 Zalo Personal 지원.

> **경고:** 비공식 자동화는 계정 정지/차단을 초래할 수 있습니다. 자신의 책임하에 사용하세요.

## 명명

채널 ID는 `zalouser`로, 이것이 **개인 Zalo 사용자 계정** (비공식)을 자동화한다는 것을 명시적으로 나타냅니다. 향후 공식 Zalo API 통합을 위해 `zalo`를 예약해 둡니다.

## 실행 위치

이 플러그인은 **Gateway 프로세스 내부에서** 실행됩니다.

원격 Gateway를 사용하는 경우, **Gateway를 실행하는 머신**에 설치/구성한 다음 Gateway를 재시작하세요.

## 설치

### 옵션 A: npm에서 설치

```bash
openclaw plugins install @openclaw/zalouser
```

이후 Gateway를 재시작하세요.

### 옵션 B: 로컬 폴더에서 설치 (개발)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

이후 Gateway를 재시작하세요.

## 전제조건: zca-cli

Gateway 머신에 `zca`가 `PATH`에 있어야 합니다:

```bash
zca --version
```

## 설정

채널 설정은 `channels.zalouser` 아래에 있습니다 (`plugins.entries.*`가 아님):

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

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## 에이전트 도구

도구 이름: `zalouser`

액션: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`
