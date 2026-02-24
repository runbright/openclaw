---
summary: "OpenClaw 온보딩 옵션 및 흐름에 대한 개요"
read_when:
  - 온보딩 경로를 선택할 때
  - 새로운 환경을 설정할 때
title: "온보딩 개요"
sidebarTitle: "온보딩 개요"
---

# 온보딩 개요

OpenClaw는 게이트웨이가 실행되는 위치와 프로바이더 구성 방식에 따라 여러 온보딩 경로를 지원합니다.

## 온보딩 경로 선택하기

- **CLI 위저드**: macOS, Linux 및 Windows(WSL2 활용) 환경용.
- **macOS 앱**: Apple silicon 또는 Intel Mac에서의 가이드 기반 초기 실행용.

## CLI 온보딩 위저드

터미널에서 위저드를 실행하세요:

```bash
openclaw onboard
```

게이트웨이, 워크스페이스, 채널 및 스킬을 완벽하게 제어하고 싶을 때 CLI 위저드를 사용하세요. 관련 문서:

- [온보딩 위저드 (CLI)](/start/wizard)
- [`openclaw onboard` 명령](/cli/onboard)

## macOS 앱 온보딩

macOS에서 가이드에 따라 설정을 완료하고 싶을 때 OpenClaw 앱을 사용하세요. 관련 문서:

- [온보딩 (macOS 앱)](/start/onboarding)

## 커스텀 프로바이더 (Custom Provider)

목록에 없는 엔드포인트가 필요하거나, 표준 OpenAI 또는 Anthropic API를 제공하는 호스팅 프로바이더를 사용하려는 경우 CLI 위저드에서 **Custom Provider**를 선택하세요. 다음 정보가 필요합니다:

- OpenAI 호환, Anthropic 호환 또는 **Unknown**(자동 감지) 중 선택.
- 베이스 URL 및 API 키 (프로바이더가 요구하는 경우).
- 모델 ID 및 선택적 별칭(alias).
- 여러 커스텀 엔드포인트를 동시에 사용할 수 있도록 엔드포인트 ID 지정.

상세 단계는 위의 CLI 온보딩 문서를 참고하세요.
