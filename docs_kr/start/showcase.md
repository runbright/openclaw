---
title: "쇼케이스"
description: "커뮤니티에서 개발한 실제 OpenClaw 프로젝트 사례"
summary: "OpenClaw를 활용한 커뮤니티 제작 프로젝트 및 통합 사례"
---

# 쇼케이스

커뮤니티에서 진행 중인 실제 프로젝트들입니다. 사람들이 OpenClaw로 무엇을 만들고 있는지 확인해 보세요.

<Info>
**여러분의 프로젝트를 소개하고 싶으신가요?** [Discord의 #showcase 채널](https://discord.gg/clawd)에 공유하거나 [X(트위터)에서 @openclaw를 태그](https://x.com/openclaw)해 주세요.
</Info>

## 🎥 OpenClaw 사용 영상

VelvetShark가 제작한 전체 설정 가이드 (28분).

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/SaWSPZoPX34"
    title="OpenClaw: The self-hosted AI that Siri should have been (Full setup)"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube에서 보기](https://www.youtube.com/watch?v=SaWSPZoPX34)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/mMSKQvlmFuQ"
    title="OpenClaw showcase video"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube에서 보기](https://www.youtube.com/watch?v=mMSKQvlmFuQ)

<div
  style={{
    position: "relative",
    paddingBottom: "56.25%",
    height: 0,
    overflow: "hidden",
    borderRadius: 16,
  }}
>
  <iframe
    src="https://www.youtube-nocookie.com/embed/5kkIJNUGFho"
    title="OpenClaw community showcase"
    style={{ position: "absolute", top: 0, left: 0, width: "100%", height: "100%" }}
    frameBorder="0"
    loading="lazy"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowFullScreen
  />
</div>

[YouTube에서 보기](https://www.youtube.com/watch?v=5kkIJNUGFho)

## 🆕 Discord 최신 소식

<CardGroup cols={2}>

<Card title="PR 리뷰 → 텔레그램 피드백" icon="code-pull-request" href="https://x.com/i/status/2010878524543131691">
  **@bangnokia** • `review` `github` `telegram`

OpenCode가 코드 수정을 완료하고 PR을 생성하면, OpenClaw가 변경 사항을 리뷰한 뒤 텔레그램으로 "소소한 제안"과 함께 명확한 머지 판정(먼저 해결해야 할 치명적 수정 사항 포함)을 전달합니다.

  <img src="/assets/showcase/pr-review-telegram.jpg" alt="텔레그램으로 전달된 OpenClaw PR 리뷰 피드백" />
</Card>

<Card title="몇 분 만에 완성된 와인 셀러 스킬" icon="wine-glass" href="https://x.com/i/status/2010916352454791216">
  **@prades_maxime** • `skills` `local` `csv`

"로비"(OpenClaw 에이전트)에게 로컬 와인 셀러 관리 스킬 제작을 요청했습니다. 에이전트는 샘플 CSV와 저장 위치를 확인한 뒤, 962병의 방대한 데이터를 처리하는 스킬을 빠르게 제작하고 테스트까지 완료했습니다.

  <img src="/assets/showcase/wine-cellar-skill.jpg" alt="CSV로부터 로컬 와인 셀러 스킬을 제작하는 OpenClaw" />
</Card>

<Card title="Tesco 쇼핑 오토파일럿" icon="cart-shopping" href="https://x.com/i/status/2009724862470689131">
  **@marchattonhere** • `automation` `browser` `shopping`

주간 식단 계획 → 자주 사는 품목 선택 → 배송 시간 예약 → 주문 확정까지. API 없이 오직 브라우저 제어만으로 쇼핑을 자동화했습니다.

  <img src="/assets/showcase/tesco-shop.jpg" alt="채팅을 통한 Tesco 쇼핑 자동화" />
</Card>

<Card title="SNAG: 스크린샷에서 마크다운으로" icon="scissors" href="https://github.com/am-will/snag">
  **@am-will** • `devtools` `screenshots` `markdown`

화면 영역을 단축키로 캡처하면 Gemini vision이 이를 인식하여 즉시 마크다운 형식으로 클립보드에 복사해 줍니다.

  <img src="/assets/showcase/snag.png" alt="SNAG 스크린샷-마크다운 변환 도구" />
</Card>

<Card title="에이전트 UI" icon="window-maximize" href="https://releaseflow.net/kitze/agents-ui">
  **@kitze** • `ui` `skills` `sync`

에이전트, Claude, Codex 및 OpenClaw에서 사용하는 스킬과 명령을 통합 관리하는 데스크톱 앱입니다.

  <img src="/assets/showcase/agents-ui.jpg" alt="에이전트 UI 앱" />
</Card>

<Card title="텔레그램 음성 메시지 (papla.media)" icon="microphone" href="https://papla.media/docs">
  **커뮤니티** • `voice` `tts` `telegram`

papla.media TTS를 활용하여 결과물을 텔레그램 음성 메시지로 전송합니다 (자동 재생 없이 자연스러운 통합).

  <img src="/assets/showcase/papla-tts.jpg" alt="TTS 결과물의 텔레그램 음성 메시지 출력" />
</Card>

<Card title="CodexMonitor" icon="eye" href="https://clawhub.com/odrobnik/codexmonitor">
  **@odrobnik** • `devtools` `codex` `brew`

로컬 OpenAI Codex 세션을 목록으로 확인하고 모니터링할 수 있는 Homebrew 기반 도우미 도구입니다 (CLI 및 VS Code 지원).

  <img src="/assets/showcase/codexmonitor.png" alt="ClawHub의 CodexMonitor" />
</Card>

<Card title="Bambu 3D 프린터 제어" icon="print" href="https://clawhub.com/tobiasbischoff/bambu-cli">
  **@tobiasbischoff** • `hardware` `3d-printing` `skill`

BambuLab 프린터의 상태 확인, 작업 관리, 카메라 모니터링, AMS, 캘리브레이션 등 트러블슈팅 및 제어를 담당합니다.

  <img src="/assets/showcase/bambu-cli.png" alt="ClawHub의 Bambu CLI 스킬" />
</Card>

<Card title="빈 대중교통 정보 (Wiener Linien)" icon="train" href="https://clawhub.com/hjanuschka/wienerlinien">
  **@hjanuschka** • `travel` `transport` `skill`

빈(Vienna)의 대중교통 실시간 출발 시간, 지연 정보, 엘리베이터 상태 및 경로 안내 기능을 제공합니다.

  <img src="/assets/showcase/wienerlinien.png" alt="ClawHub의 Wiener Linien 스킬" />
</Card>

<Card title="ParentPay 학교 급식 예약" icon="utensils" href="#">
  **@George5562** • `automation` `browser` `parenting`

ParentPay를 통한 영국 학교 급식 예약 자동화입니다. 마우스 좌표를 활용하여 표의 셀을 정확하게 클릭하도록 설계되었습니다.
</Card>

<Card title="R2 업로드 (내 파일 보내기)" icon="cloud-arrow-up" href="https://clawhub.com/skills/r2-upload">
  **@julianengel** • `files` `r2` `presigned-urls`

Cloudflare R2/S3에 파일을 업로드하고 보안 다운로드 링크(presigned link)를 생성합니다. 원격 OpenClaw 인스턴스에서 사용하기 좋습니다.
</Card>

<Card title="텔레그램만으로 iOS 앱 개발 및 배포" icon="mobile" href="#">
  **@coard** • `ios` `xcode` `testflight`

지도와 음성 녹음 기능이 포함된 완전한 iOS 앱을 제작하고, 텔레그램 채팅만으로 TestFlight 배포까지 완료했습니다.

  <img src="/assets/showcase/ios-testflight.jpg" alt="TestFlight의 iOS 앱" />
</Card>

<Card title="Oura 링 건강 비서" icon="heart-pulse" href="#">
  **@AS** • `health` `oura` `calendar`

Oura 링의 건강 데이터와 캘린더, 일정, 운동 스케줄을 통합 관리하는 개인용 AI 건강 비서입니다.

  <img src="/assets/showcase/oura-health.png" alt="Oura 링 건강 비서" />
</Card>

<Card title="Kev's Dream Team (14개 이상의 에이전트)" icon="robot" href="https://github.com/adam91holt/orchestrated-ai-articles">
  **@adam91holt** • `multi-agent` `orchestration` `architecture` `manifesto`

Opus 4.5 오케스트레이터가 Codex 작업자들에게 업무를 할당하는, 하나의 게이트웨이 아래 14개 이상의 에이전트가 협업하는 구조입니다. 모델 선택, 샌드박싱, 웹훅, 하트비트 등을 다룬 상세한 [기술 리포트](https://github.com/adam91holt/orchestrated-ai-articles)가 공개되어 있습니다.
</Card>

<Card title="Linear CLI" icon="terminal" href="https://github.com/Finesssee/linear-cli">
  **@NessZerra** • `devtools` `linear` `cli` `issues`

에이전트 워크플로(Claude Code, OpenClaw)와 통합된 Linear용 CLI입니다. 업무, 프로젝트, 워크플로를 터미널에서 직접 관리하세요.
</Card>

<Card title="Beeper CLI" icon="message" href="https://github.com/blqke/beepcli">
  **@jules** • `messaging` `beeper` `cli` `automation`

Beeper Desktop을 통해 메시지를 읽고, 보내고, 저장합니다. Beeper의 로컬 MCP API를 활용하여 iMessage, WhatsApp 등 모든 채팅을 한곳에서 관리할 수 있습니다.
</Card>

</CardGroup>

## 🤖 자동화 및 워크플로

<CardGroup cols={2}>

<Card title="Winix 공기청정기 제어" icon="wind" href="https://x.com/antonplex/status/2010518442471006253">
  **@antonplex** • `automation` `hardware` `air-quality`

Claude Code가 청정기 제어 방식을 찾아내 확인하고, OpenClaw가 이를 이어받아 실내 공기질을 자동으로 관리합니다.

  <img src="/assets/showcase/winix-air-purifier.jpg" alt="OpenClaw를 통한 Winix 공기청정기 제어" />
</Card>

<Card title="예쁜 하늘 촬영" icon="camera" href="https://x.com/signalgaining/status/2010523120604746151">
  **@signalgaining** • `automation` `camera` `skill` `images`

옥상 카메라를 연동했습니다. 하늘이 예쁠 때마다 사진을 찍어달라고 요청하면, OpenClaw가 직접 스킬을 제작해 사진을 촬영합니다.

  <img src="/assets/showcase/roof-camera-sky.jpg" alt="OpenClaw가 촬영한 옥상 카메라 하늘 사진" />
</Card>

<Card title="시각화된 모닝 브리핑" icon="robot" href="https://x.com/buddyhadry/status/2010005331925954739">
  **@buddyhadry** • `automation` `briefing` `images` `telegram`

매일 아침 날씨, 할 일, 날짜, 명언 등이 포함된 하나의 요약 이미지(Scene)를 생성하여 전송하는 에이전트 페르소나입니다.
</Card>

<Card title="파델(Padel) 경기장 예약" icon="calendar-check" href="https://github.com/joshp123/padel-cli">
  **@joshp123** • `automation` `booking` `cli`
  
Playtomic 빈자리 확인 및 예약용 CLI입니다. 더 이상 빈 코트를 놓칠 염려가 없습니다.
  
  <img src="/assets/showcase/padel-screenshot.jpg" alt="padel-cli 스크린샷" />
</Card>

<Card title="회계 서류 처리" icon="file-invoice-dollar">
  **커뮤니티** • `automation` `email` `pdf`
  
이메일에서 PDF를 수집하고 세무사용 서류를 준비합니다. 매달 번거로운 회계 작업을 자동화했습니다.
</Card>

<Card title="넷플릭스 보며 개인 사이트 개편" icon="couch" href="https://davekiss.com">
  **@davekiss** • `telegram` `website` `migration` `astro`

넷플릭스를 보면서 텔레그램 대화만으로 전체 개인 사이트를 개편했습니다. Notion에서 Astro로 18개의 포스트를 이전하고 DNS 설정까지 마쳤습니다. 노트북을 한 번도 펴지 않았죠.
</Card>

<Card title="구인 정보 검색 에이전트" icon="briefcase">
  **@attol8** • `automation` `api` `skill`

구인 목록을 검색하고 이력서 키워드와 대조하여 관련 있는 기회만 링크와 함께 제공합니다. JSearch API를 사용하여 30분 만에 구축되었습니다.
</Card>

<Card title="Jira 스킬 빌더" icon="diagram-project" href="https://x.com/jdrhyne/status/2008336434827002232">
  **@jdrhyne** • `automation` `jira` `skill` `devtools`

OpenClaw가 Jira에 연결된 뒤 즉석에서 새로운 스킬을 생성해 냈습니다.
</Card>

<Card title="텔레그램 Todoist 스킬" icon="list-check" href="https://x.com/iamsubhrajyoti/status/2009949389884920153">
  **@iamsubhrajyoti** • `automation` `todoist` `skill` `telegram`

Todoist 작업을 자동화하고, 에이전트가 텔레그램 채팅 내에서 직접 스킬을 생성하도록 했습니다.
</Card>

<Card title="TradingView 분석" icon="chart-line">
  **@bheem1798** • `finance` `browser` `automation`

브라우저 자동화를 통해 TradingView에 로그인하고, 차트를 스캔하여 요청 시 기술적 분석을 수행합니다. API 없이 오직 브라우저 제어만 사용합니다.
</Card>

<Card title="Slack 자동 지원 에이전트" icon="slack">
  **@henrymascot** • `slack` `automation` `support`

회사 Slack 채널을 모니터링하여 답변하고 중요한 알림은 텔레그램으로 전달합니다. 요청하지 않았음에도 서버의 버그를 스스로 찾아 수정하기도 했습니다.
</Card>

</CardGroup>

## 🧠 지식 및 메모리

<CardGroup cols={2}>

<Card title="xuezh 중국어 학습" icon="language" href="https://github.com/joshp123/xuezh">
  **@joshp123** • `learning` `voice` `skill`
  
OpenClaw를 통한 발음 피드백 및 학습 흐름을 제공하는 중국어 학습 엔진입니다.
  
  <img src="/assets/showcase/xuezh-pronunciation.jpeg" alt="xuezh 발음 피드백" />
</Card>

<Card title="WhatsApp 메모리 보관함" icon="vault">
  **커뮤니티** • `memory` `transcription` `indexing`
  
WhatsApp 대화 내역 전체를 분석하고, 1,000개 이상의 음성 메시지를 텍스트로 변환한 뒤 Git 로그와 대조하여 연결된 마크다운 보고서를 작성합니다.
</Card>

<Card title="Karakeep 시맨틱 검색" icon="magnifying-glass" href="https://github.com/jamesbrooksco/karakeep-semantic-search">
  **@jamesbrooksco** • `search` `vector` `bookmarks`
  
Qdrant와 OpenAI/Ollama 임베딩을 사용하여 Karakeep 북마크에 벡터 검색 기능을 추가합니다.
</Card>

<Card title="에이전트 자아 모델 (Inside-Out-2)" icon="brain">
  **커뮤니티** • `memory` `beliefs` `self-model`
  
세션 파일을 메모리, 신념, 진화하는 자아 모델로 변환하는 별도의 메모리 관리자입니다.
</Card>

</CardGroup>

## 🎙️ 음성 및 전화

<CardGroup cols={2}>

<Card title="Clawdia 전화 브릿지" icon="phone" href="https://github.com/alejandroOPI/clawdia-bridge">
  **@alejandroOPI** • `voice` `vapi` `bridge`
  
Vapi 음성 비서와 OpenClaw HTTP 브릿지를 연동하여, 에이전트와 거의 실시간으로 전화 통화를 할 수 있습니다.
</Card>

<Card title="OpenRouter 음성 전사" icon="microphone" href="https://clawhub.com/obviyus/openrouter-transcribe">
  **@obviyus** • `transcription` `multilingual` `skill`

OpenRouter(Gemini 등)를 통한 다국어 오디오 전사 기능입니다. ClawHub에서 이용 가능합니다.
</Card>

</CardGroup>

## 🏗️ 인프라 및 배포

<CardGroup cols={2}>

<Card title="Home Assistant 애드온" icon="home" href="https://github.com/ngutman/openclaw-ha-addon">
  **@ngutman** • `homeassistant` `docker` `raspberry-pi`
  
ssh 터널링과 상태 유지를 지원하는 Home Assistant OS용 OpenClaw 게이트웨이입니다.
</Card>

<Card title="Home Assistant 스킬" icon="toggle-on" href="https://clawhub.com/skills/homeassistant">
  **ClawHub** • `homeassistant` `skill` `automation`
  
일상 언어로 Home Assistant 기기들을 제어하고 자동화하세요.
</Card>

<Card title="Nix 패키지" icon="snowflake" href="https://github.com/openclaw/nix-openclaw">
  **@openclaw** • `nix` `packaging` `deployment`
  
재현 가능한 배포를 위한 Nix 기반 OpenClaw 설정 패키지입니다.
</Card>

<Card title="CalDAV 캘린더" icon="calendar" href="https://clawhub.com/skills/caldav-calendar">
  **ClawHub** • `calendar` `caldav` `skill`
  
khal/vdirsyncer를 활용한 셀프 호스팅 캘린더 통합 스킬입니다.
</Card>

</CardGroup>

## 🌟 커뮤니티 프로젝트

<CardGroup cols={2}>

<Card title="StarSwap 마켓플레이스" icon="star" href="https://star-swap.com/">
  **커뮤니티** • `marketplace` `astronomy` `webapp`
  
OpenClaw 생태계를 활용해 구축된 방대한 천문 장비 마켓플레이스입니다.
</Card>

</CardGroup>

---

## 프로젝트 제출하기

공유하고 싶은 것이 있으신가요? 여러분의 프로젝트를 소개하고 싶습니다!

<Steps>
  <Step title="공유하기">
    [Discord의 #showcase 채널](https://discord.gg/clawd)에 올리거나 [X에서 @openclaw](https://x.com/openclaw)를 태그해 주세요.
  </Step>
  <Step title="상세 정보 포함">
    어떤 기능인지, 레포지토리/데모 링크 및 스크린샷이 있다면 함께 공유해 주세요.
  </Step>
  <Step title="소개되기">
    선정된 프로젝트는 이 페이지에 추가됩니다.
  </Step>
</Steps>
