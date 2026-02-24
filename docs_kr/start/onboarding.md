---
summary: "OpenClaw(macOS 앱)의 초기 실행 온보딩 흐름"
read_when:
  - macOS 온보딩 어시스턴트를 설계할 때
  - 인증 또는 신원 설정을 구현할 때
title: "온보딩 (macOS 앱)"
sidebarTitle: "온보딩: macOS 앱"
---

# 온보딩 (macOS 앱)

이 문서는 **현재**의 초기 실행 온보딩 흐름을 설명합니다. 목표는 부드러운 "Day 0" 경험을 제공하는 것입니다. 게이트웨이 실행 위치 선택, 인증 연결, 위저드 실행, 그리고 에이전트의 자체 부트스트래핑 과정을 안내합니다.
온보딩 경로에 대한 일반적인 개요는 [온보딩 개요](/start/onboarding-overview)를 참고하세요.

<Steps>
<Step title="macOS 경고 승인">
<Frame>
<img src="/assets/macos-onboarding/01-macos-warning.jpeg" alt="" />
</Frame>
</Step>
<Step title="로컬 네트워크 찾기 권한 승인">
<Frame>
<img src="/assets/macos-onboarding/02-local-networks.jpeg" alt="" />
</Frame>
</Step>
<Step title="환영 메시지 및 보안 고지">
<Frame caption="표시된 보안 고지 내용을 확인하고 결정하세요.">
<img src="/assets/macos-onboarding/03-security-notice.png" alt="" />
</Frame>
</Step>
<Step title="로컬(Local) vs 원격(Remote)">
<Frame>
<img src="/assets/macos-onboarding/04-choose-gateway.png" alt="" />
</Frame>

**게이트웨이(Gateway)**를 어디에서 실행하시겠습니까?

- **이 Mac (로컬 전용):** 온보딩 과정에서 OAuth 흐름을 실행하고 인증 정보를 로컬에 저장할 수 있습니다.
- **원격 (SSH/Tailnet 경유):** 온보딩 과정에서 로컬 OAuth를 실행하지 않습니다. 인증 정보는 게이트웨이 호스트에 이미 존재해야 합니다.
- **나중에 설정:** 설정을 건너뛰고 앱을 구성되지 않은 상태로 둡니다.

<Tip>
**게이트웨이 인증 팁:**
- 이제 위저드는 루프백 호출에 대해서도 **토큰**을 생성하므로, 로컬 WebSocket 클라이언트도 인증이 필요합니다.
- 인증을 비활성화하면 모든 로컬 프로세스가 연결할 수 있습니다. 완전히 신뢰할 수 있는 머신에서만 사용하세요.
- 여러 머신에서 접속하거나 루프백이 아닌 바인딩을 사용하는 경우 반드시 **토큰**을 사용하세요.
</Tip>
</Step>
<Step title="권한 설정">
<Frame caption="OpenClaw에 부여할 권한을 선택하세요.">
<img src="/assets/macos-onboarding/05-permissions.png" alt="" />
</Frame>

온보딩 과정에서 다음 기능을 위해 TCC 권한을 요청합니다:

- 자동화 (AppleScript)
- 알림
- 손쉬운 사용 (Accessibility)
- 화면 기록
- 마이크
- 음성 인식
- 카메라
- 위치 정보

</Step>
<Step title="CLI 설치">
  <Info>이 단계는 선택 사항입니다.</Info>
  터미널 워크플로와 launchd 작업이 즉시 작동할 수 있도록 앱에서 npm/pnpm을 통해 글로벌 `openclaw` CLI를 설치할 수 있습니다.
</Step>
<Step title="온보딩 채팅 (전용 세션)">
  설정이 끝나면 에이전트가 자신을 소개하고 다음 단계를 안내할 수 있도록 전용 온보딩 채팅 세션이 열립니다. 이를 통해 초기 가이드 내용이 일반 대화와 섞이지 않도록 관리합니다. 에이전트의 첫 실행 시 게이트웨이 호스트에서 일어나는 일에 대해서는 [부트스트래핑](/start/bootstrapping) 문서를 참고하세요.
</Step>
</Steps>
