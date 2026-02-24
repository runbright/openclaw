---
summary: "`openclaw qr` (iOS 페어링 QR + 설정 코드 생성) CLI 레퍼런스"
read_when:
  - iOS 앱을 게이트웨이와 빠르게 페어링하고 싶을 때
  - 원격/수동 공유를 위한 설정 코드 출력이 필요할 때
title: "qr"
---

# `openclaw qr`

현재 Gateway 구성에서 iOS 페어링 QR 및 설정 코드를 생성합니다.

## 사용법

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## 옵션

- `--remote`: 구성에서 `gateway.remote.url` 및 원격 토큰/비밀번호 사용
- `--url <url>`: 페이로드에 사용되는 게이트웨이 URL 재정의
- `--public-url <url>`: 페이로드에 사용되는 공개 URL 재정의
- `--token <token>`: 페이로드의 게이트웨이 토큰 재정의
- `--password <password>`: 페이로드의 게이트웨이 비밀번호 재정의
- `--setup-code-only`: 설정 코드만 출력
- `--no-ascii`: ASCII QR 렌더링 건너뛰기
- `--json`: JSON 출력 (`setupCode`, `gatewayUrl`, `auth`, `urlSource`)

## 참고

- `--token`과 `--password`는 상호 배타적입니다.
- 스캔 후 다음으로 장치 페어링을 승인하세요:
  - `openclaw devices list`
  - `openclaw devices approve <requestId>`
