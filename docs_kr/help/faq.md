---
summary: "OpenClaw 설정, 구성 및 사용에 대한 자주 묻는 질문"
read_when:
  - Answering common setup, install, onboarding, or runtime support questions
  - Triaging user-reported issues before deeper debugging
title: "FAQ"
---

# FAQ

실제 환경 설정(로컬 개발, VPS, 멀티 에이전트, OAuth/API 키, 모델 장애 조치)에 대한 빠른 답변과 심층 문제 해결. 런타임 진단은 [문제 해결](/gateway/troubleshooting)을 참조하세요. 전체 구성 참조는 [구성](/gateway/configuration)을 참조하세요.

## 목차

- [빠른 시작 및 최초 실행 설정]
  - [막혔는데 가장 빠르게 해결하는 방법은?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [OpenClaw를 설치하고 설정하는 권장 방법은?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [온보딩 후 대시보드를 어떻게 여나요?](#how-do-i-open-the-dashboard-after-onboarding)
  - [localhost와 원격에서 대시보드 토큰 인증은 어떻게 하나요?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [어떤 런타임이 필요한가요?](#what-runtime-do-i-need)
  - [Raspberry Pi에서 실행되나요?](#does-it-run-on-raspberry-pi)
  - [Raspberry Pi 설치 팁이 있나요?](#any-tips-for-raspberry-pi-installs)
  - ["wake up my friend"에서 멈춤 / 온보딩이 시작되지 않음. 어떻게 하나요?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [온보딩을 다시 하지 않고 새 머신(Mac mini)으로 설정을 마이그레이션할 수 있나요?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [최신 버전의 변경 사항은 어디서 볼 수 있나요?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [docs.openclaw.ai에 접속할 수 없습니다 (SSL 오류). 어떻게 하나요?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [stable과 beta의 차이점은 무엇인가요?](#whats-the-difference-between-stable-and-beta)
  - [beta 버전은 어떻게 설치하고, beta와 dev의 차이점은 무엇인가요?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [최신 빌드를 사용해보려면 어떻게 하나요?](#how-do-i-try-the-latest-bits)
  - [설치와 온보딩은 보통 얼마나 걸리나요?](#how-long-does-install-and-onboarding-usually-take)
  - [설치가 멈췄나요? 더 많은 피드백을 받으려면?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows 설치 시 git을 찾을 수 없거나 openclaw가 인식되지 않음](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [문서에서 답을 찾지 못했는데 - 더 나은 답변을 얻으려면?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Linux에 OpenClaw를 어떻게 설치하나요?](#how-do-i-install-openclaw-on-linux)
  - [VPS에 OpenClaw를 어떻게 설치하나요?](#how-do-i-install-openclaw-on-a-vps)
  - [클라우드/VPS 설치 가이드는 어디에 있나요?](#where-are-the-cloudvps-install-guides)
  - [OpenClaw에게 자체 업데이트를 요청할 수 있나요?](#can-i-ask-openclaw-to-update-itself)
  - [온보딩 마법사는 실제로 무엇을 하나요?](#what-does-the-onboarding-wizard-actually-do)
  - [이것을 실행하려면 Claude나 OpenAI 구독이 필요한가요?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [API 키 없이 Claude Max 구독을 사용할 수 있나요?](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Anthropic "setup-token" 인증은 어떻게 작동하나요?](#how-does-anthropic-setuptoken-auth-work)
  - [Anthropic setup-token은 어디서 찾나요?](#where-do-i-find-an-anthropic-setuptoken)
  - [Claude 구독 인증(Claude Pro 또는 Max)을 지원하나요?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Anthropic에서 `HTTP 429: rate_limit_error`가 보이는 이유는?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [AWS Bedrock이 지원되나요?](#is-aws-bedrock-supported)
  - [Codex 인증은 어떻게 작동하나요?](#how-does-codex-auth-work)
  - [OpenAI 구독 인증(Codex OAuth)을 지원하나요?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Gemini CLI OAuth를 어떻게 설정하나요?](#how-do-i-set-up-gemini-cli-oauth)
  - [일상적인 대화에 로컬 모델을 사용해도 괜찮나요?](#is-a-local-model-ok-for-casual-chats)
  - [호스팅 모델 트래픽을 특정 지역으로 유지하려면?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [이것을 설치하려면 Mac Mini를 구매해야 하나요?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [iMessage 지원을 위해 Mac mini가 필요한가요?](#do-i-need-a-mac-mini-for-imessage-support)
  - [OpenClaw을 실행하기 위해 Mac mini를 구매하면 MacBook Pro에 연결할 수 있나요?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Bun을 사용할 수 있나요?](#can-i-use-bun)
  - [Telegram: `allowFrom`에 무엇을 넣나요?](#telegram-what-goes-in-allowfrom)
  - [여러 사람이 하나의 WhatsApp 번호를 다른 OpenClaw 인스턴스로 사용할 수 있나요?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - ["빠른 채팅" 에이전트와 "코딩용 Opus" 에이전트를 실행할 수 있나요?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Linux에서 Homebrew가 작동하나요?](#does-homebrew-work-on-linux)
  - [hackable(git) 설치와 npm 설치의 차이점은?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [나중에 npm과 git 설치 간에 전환할 수 있나요?](#can-i-switch-between-npm-and-git-installs-later)
  - [Gateway를 노트북에서 실행해야 하나요, VPS에서 실행해야 하나요?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [OpenClaw를 전용 머신에서 실행하는 것이 얼마나 중요한가요?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [최소 VPS 요구 사항과 권장 OS는?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [VM에서 OpenClaw를 실행할 수 있나요, 요구 사항은?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [OpenClaw란 무엇인가요?](#what-is-openclaw)
  - [OpenClaw란, 한 문단으로?](#what-is-openclaw-in-one-paragraph)
  - [가치 제안은 무엇인가요?](#whats-the-value-proposition)
  - [방금 설정했는데 먼저 무엇을 해야 하나요?](#i-just-set-it-up-what-should-i-do-first)
  - [OpenClaw의 일상적 사용 사례 상위 다섯 가지는?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [OpenClaw가 SaaS의 리드 생성, 아웃리치, 광고 및 블로그에 도움이 될 수 있나요?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [웹 개발에서 Claude Code 대비 장점은?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [스킬 및 자동화](#skills-and-automation)
  - [저장소를 더럽히지 않고 스킬을 커스터마이즈하려면?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [커스텀 폴더에서 스킬을 로드할 수 있나요?](#can-i-load-skills-from-a-custom-folder)
  - [다른 작업에 다른 모델을 사용하려면?](#how-can-i-use-different-models-for-different-tasks)
  - [봇이 무거운 작업 중 멈춥니다. 어떻게 오프로드하나요?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron이나 리마인더가 실행되지 않습니다. 무엇을 확인해야 하나요?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Linux에서 스킬을 어떻게 설치하나요?](#how-do-i-install-skills-on-linux)
  - [OpenClaw가 일정에 따라 또는 백그라운드에서 지속적으로 작업을 실행할 수 있나요?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Linux에서 Apple macOS 전용 스킬을 실행할 수 있나요?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Notion이나 HeyGen 통합이 있나요?](#do-you-have-a-notion-or-heygen-integration)
  - [브라우저 테이크오버를 위한 Chrome 확장을 어떻게 설치하나요?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [샌드박싱 및 메모리](#sandboxing-and-memory)
  - [전용 샌드박싱 문서가 있나요?](#is-there-a-dedicated-sandboxing-doc)
  - [호스트 폴더를 샌드박스에 바인드하려면?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [메모리는 어떻게 작동하나요?](#how-does-memory-work)
  - [메모리가 계속 잊어버립니다. 어떻게 유지시키나요?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [메모리는 영구적으로 유지되나요? 제한은 무엇인가요?](#does-memory-persist-forever-what-are-the-limits)
  - [시맨틱 메모리 검색에 OpenAI API 키가 필요한가요?](#does-semantic-memory-search-require-an-openai-api-key)
- [디스크의 데이터 위치](#where-things-live-on-disk)
  - [OpenClaw와 함께 사용되는 모든 데이터가 로컬에 저장되나요?](#is-all-data-used-with-openclaw-saved-locally)
  - [OpenClaw는 데이터를 어디에 저장하나요?](#where-does-openclaw-store-its-data)
  - [AGENTS.md / SOUL.md / USER.md / MEMORY.md는 어디에 있어야 하나요?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [권장 백업 전략은?](#whats-the-recommended-backup-strategy)
  - [OpenClaw를 완전히 제거하려면?](#how-do-i-completely-uninstall-openclaw)
  - [에이전트가 워크스페이스 밖에서 작업할 수 있나요?](#can-agents-work-outside-the-workspace)
  - [원격 모드에서 세션 저장소는 어디에 있나요?](#im-in-remote-mode-where-is-the-session-store)
- [구성 기본 사항](#config-basics)
  - [구성 형식은 무엇이고 어디에 있나요?](#what-format-is-the-config-where-is-it)
  - [`gateway.bind: "lan"` (또는 `"tailnet"`)을 설정했더니 아무것도 리슨하지 않음 / UI가 unauthorized라고 함](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [왜 localhost에서도 토큰이 필요한가요?](#why-do-i-need-a-token-on-localhost-now)
  - [구성 변경 후 재시작해야 하나요?](#do-i-have-to-restart-after-changing-config)
  - [웹 검색(및 웹 fetch)을 어떻게 활성화하나요?](#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply가 내 구성을 지웠습니다. 복구하고 방지하려면?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [여러 장치에 걸쳐 전문화된 워커가 있는 중앙 Gateway를 어떻게 실행하나요?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [OpenClaw 브라우저를 headless로 실행할 수 있나요?](#can-the-openclaw-browser-run-headless)
  - [브라우저 제어에 Brave를 어떻게 사용하나요?](#how-do-i-use-brave-for-browser-control)
- [원격 게이트웨이 및 노드](#remote-gateways-and-nodes)
  - [Telegram, 게이트웨이, 노드 간에 명령이 어떻게 전파되나요?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [Gateway가 원격에 호스팅되어 있을 때 에이전트가 내 컴퓨터에 접근하려면?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale이 연결되었지만 응답이 없습니다. 어떻게 하나요?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [두 OpenClaw 인스턴스가 서로 통신할 수 있나요 (로컬 + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [여러 에이전트에 별도의 VPS가 필요한가요?](#do-i-need-separate-vpses-for-multiple-agents)
  - [VPS에서 SSH 대신 개인 노트북에서 노드를 사용하는 이점이 있나요?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [노드는 게이트웨이 서비스를 실행하나요?](#do-nodes-run-a-gateway-service)
  - [구성을 적용하는 API / RPC 방법이 있나요?](#is-there-an-api-rpc-way-to-apply-config)
  - [최초 설치를 위한 최소한의 "합리적인" 구성은?](#whats-a-minimal-sane-config-for-a-first-install)
  - [VPS에 Tailscale을 설정하고 Mac에서 연결하려면?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [Mac 노드를 원격 Gateway에 연결하려면 (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [두 번째 노트북에 설치해야 하나요, 노드만 추가하면 되나요?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [환경 변수 및 .env 로딩](#env-vars-and-env-loading)
  - [OpenClaw는 환경 변수를 어떻게 로드하나요?](#how-does-openclaw-load-environment-variables)
  - ["서비스를 통해 Gateway를 시작했더니 환경 변수가 사라졌습니다." 어떻게 하나요?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [`COPILOT_GITHUB_TOKEN`을 설정했는데 models status에 "Shell env: off."라고 나옵니다. 왜 그런가요?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [세션 및 다중 채팅](#sessions-and-multiple-chats)
  - [새 대화를 시작하려면?](#how-do-i-start-a-fresh-conversation)
  - [`/new`를 보내지 않으면 세션이 자동으로 초기화되나요?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [OpenClaw 인스턴스 팀을 만들 수 있나요 - CEO 하나와 많은 에이전트?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [작업 중 컨텍스트가 잘린 이유는? 어떻게 방지하나요?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [OpenClaw를 완전히 초기화하되 설치는 유지하려면?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - ["context too large" 오류가 발생합니다 - 초기화하거나 압축하려면?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - ["LLM request rejected: messages.content.tool_use.input field required"가 보이는 이유는?](#why-am-i-seeing-llm-request-rejected-messagescontenttool_useinput-field-required)
  - [왜 30분마다 하트비트 메시지를 받나요?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [WhatsApp 그룹에 "봇 계정"을 추가해야 하나요?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [WhatsApp 그룹의 JID를 어떻게 얻나요?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [왜 OpenClaw가 그룹에서 응답하지 않나요?](#why-doesnt-openclaw-reply-in-a-group)
  - [그룹/스레드가 DM과 컨텍스트를 공유하나요?](#do-groupsthreads-share-context-with-dms)
  - [워크스페이스와 에이전트를 몇 개까지 만들 수 있나요?](#how-many-workspaces-and-agents-can-i-create)
  - [동시에 여러 봇이나 채팅을 실행할 수 있나요 (Slack), 어떻게 설정하나요?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [모델: 기본값, 선택, 별칭, 전환](#models-defaults-selection-aliases-switching)
  - ["기본 모델"이란 무엇인가요?](#what-is-the-default-model)
  - [어떤 모델을 추천하나요?](#what-model-do-you-recommend)
  - [구성을 지우지 않고 모델을 전환하려면?](#how-do-i-switch-models-without-wiping-my-config)
  - [자체 호스팅 모델(llama.cpp, vLLM, Ollama)을 사용할 수 있나요?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [OpenClaw, Flawd, Krill은 어떤 모델을 사용하나요?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [재시작 없이 즉시 모델을 전환하려면?](#how-do-i-switch-models-on-the-fly-without-restarting)
  - [일상 작업에 GPT 5.2를, 코딩에 Codex 5.3을 사용할 수 있나요?](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - ["Model ... is not allowed"가 나타나고 응답이 없는 이유는?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - ["Unknown model: minimax/MiniMax-M2.1"이 보이는 이유는?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [MiniMax를 기본으로, 복잡한 작업에는 OpenAI를 사용할 수 있나요?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [opus / sonnet / gpt는 내장 단축키인가요?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [모델 단축키(별칭)를 정의/재정의하려면?](#how-do-i-defineoverride-model-shortcuts-aliases)
  - [OpenRouter나 Z.AI 같은 다른 프로바이더의 모델을 추가하려면?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [모델 장애 조치 및 "All models failed"](#model-failover-and-all-models-failed)
  - [장애 조치는 어떻게 작동하나요?](#how-does-failover-work)
  - [이 오류는 무엇을 의미하나요?](#what-does-this-error-mean)
  - [`No credentials found for profile "anthropic:default"` 수정 체크리스트](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [왜 Google Gemini도 시도하고 실패했나요?](#why-did-it-also-try-google-gemini-and-fail)
- [인증 프로필: 정의와 관리 방법](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [인증 프로필이란 무엇인가요?](#what-is-an-auth-profile)
  - [일반적인 프로필 ID는 무엇인가요?](#what-are-typical-profile-ids)
  - [어떤 인증 프로필을 먼저 시도할지 제어할 수 있나요?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth vs API 키: 차이점은?](#oauth-vs-api-key-whats-the-difference)
- [Gateway: 포트, "already running", 원격 모드](#gateway-ports-already-running-and-remote-mode)
  - [Gateway는 어떤 포트를 사용하나요?](#what-port-does-the-gateway-use)
  - [`openclaw gateway status`가 `Runtime: running`이지만 `RPC probe: failed`인 이유는?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [`openclaw gateway status`에서 `Config (cli)`와 `Config (service)`가 다른 이유는?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - ["another gateway instance is already listening"은 무슨 의미인가요?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [원격 모드로 OpenClaw를 실행하려면 (클라이언트가 다른 곳의 Gateway에 연결)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [Control UI가 "unauthorized"라고 하거나 계속 재연결합니다. 어떻게 하나요?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [`gateway.bind: "tailnet"`을 설정했는데 바인드할 수 없음 / 아무것도 리슨하지 않음](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [같은 호스트에서 여러 Gateway를 실행할 수 있나요?](#can-i-run-multiple-gateways-on-the-same-host)
  - ["invalid handshake" / code 1008은 무슨 의미인가요?](#what-does-invalid-handshake-code-1008-mean)
- [로깅 및 디버깅](#logging-and-debugging)
  - [로그는 어디에 있나요?](#where-are-logs)
  - [Gateway 서비스를 시작/중지/재시작하려면?](#how-do-i-startstoprestart-the-gateway-service)
  - [Windows에서 터미널을 닫았습니다 - OpenClaw를 재시작하려면?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [Gateway가 실행 중인데 응답이 도착하지 않습니다. 무엇을 확인해야 하나요?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - ["Disconnected from gateway: no reason" - 어떻게 하나요?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands가 네트워크 오류와 함께 실패합니다. 무엇을 확인해야 하나요?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [TUI에 출력이 표시되지 않습니다. 무엇을 확인해야 하나요?](#tui-shows-no-output-what-should-i-check)
  - [Gateway를 완전히 중지한 후 시작하려면?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [무언가 실패할 때 더 많은 세부 정보를 얻는 가장 빠른 방법은?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [미디어 및 첨부 파일](#media-and-attachments)
  - [스킬이 이미지/PDF를 생성했는데 아무것도 전송되지 않았습니다](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [보안 및 접근 제어](#security-and-access-control)
  - [인바운드 DM에 OpenClaw를 노출하는 것이 안전한가요?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [프롬프트 인젝션은 공개 봇에만 해당되는 문제인가요?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [봇에게 별도의 이메일 GitHub 계정이나 전화번호가 있어야 하나요?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [텍스트 메시지에 대한 자율권을 줄 수 있고, 안전한가요?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [개인 어시스턴트 작업에 저렴한 모델을 사용할 수 있나요?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  - [Telegram에서 `/start`를 실행했는데 페어링 코드를 받지 못했습니다](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp: 내 연락처에 메시지를 보내나요? 페어링은 어떻게 작동하나요?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [채팅 명령, 작업 중단, "멈추지 않습니다"](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [내부 시스템 메시지가 채팅에 표시되지 않게 하려면?](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [실행 중인 작업을 중지/취소하려면?](#how-do-i-stopcancel-a-running-task)
  - [Telegram에서 Discord 메시지를 보내려면? ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [봇이 빠른 연속 메시지를 "무시"하는 것 같은 이유는?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## 문제 발생 시 첫 60초

1. **빠른 상태 확인 (첫 번째 점검)**

   ```bash
   openclaw status
   ```

   빠른 로컬 요약: OS + 업데이트, 게이트웨이/서비스 접근성, 에이전트/세션, 프로바이더 구성 + 런타임 이슈 (게이트웨이가 접근 가능한 경우).

2. **붙여넣기 가능한 보고서 (공유해도 안전)**

   ```bash
   openclaw status --all
   ```

   읽기 전용 진단 + 로그 tail (토큰이 수정됨).

3. **데몬 + 포트 상태**

   ```bash
   openclaw gateway status
   ```

   슈퍼바이저 런타임 vs RPC 접근성, 프로브 대상 URL, 서비스가 사용한 것으로 보이는 구성을 보여줍니다.

4. **심층 프로브**

   ```bash
   openclaw status --deep
   ```

   게이트웨이 상태 확인 + 프로바이더 프로브를 실행합니다 (접근 가능한 게이트웨이 필요). [Health](/gateway/health)를 참조하세요.

5. **최신 로그 tail**

   ```bash
   openclaw logs --follow
   ```

   RPC가 다운된 경우 다음으로 대체:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   파일 로그는 서비스 로그와 별도입니다; [로깅](/logging) 및 [문제 해결](/gateway/troubleshooting)을 참조하세요.

6. **doctor 실행 (수리)**

   ```bash
   openclaw doctor
   ```

   구성/상태 수리/마이그레이션 + 상태 확인을 실행합니다. [Doctor](/gateway/doctor)를 참조하세요.

7. **Gateway 스냅샷**

   ```bash
   openclaw health --json
   openclaw health --verbose   # 오류 시 대상 URL + 구성 경로를 표시
   ```

   실행 중인 게이트웨이에 전체 스냅샷을 요청합니다 (WS 전용). [Health](/gateway/health)를 참조하세요.

## 빠른 시작 및 최초 실행 설정

### 막혔는데 가장 빠르게 해결하는 방법은

**당신의 머신을 볼 수 있는** 로컬 AI 에이전트를 사용하세요. Discord에서 질문하는 것보다 훨씬 효과적입니다. 대부분의 "막혔다" 사례는 원격 도우미가 검사할 수 없는 **로컬 구성 또는 환경 이슈**이기 때문입니다.

- **Claude Code**: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
- **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

이러한 도구들은 저장소를 읽고, 명령을 실행하고, 로그를 검사하고, 머신 수준 설정(PATH, 서비스, 권한, 인증 파일)을 수정하는 데 도움을 줄 수 있습니다. **전체 소스 체크아웃**을 hackable(git) 설치를 통해 제공하세요:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

이렇게 하면 OpenClaw가 **git 체크아웃에서** 설치되므로 에이전트가 코드 + 문서를 읽고 실행 중인 정확한 버전에 대해 추론할 수 있습니다. 나중에 `--install-method git` 없이 설치 프로그램을 다시 실행하면 언제든지 stable로 전환할 수 있습니다.

팁: 에이전트에게 수정을 **계획하고 감독**하도록 요청한 다음(단계별), 필요한 명령만 실행하세요. 이렇게 하면 변경 사항이 작아지고 감사하기 쉬워집니다.

실제 버그나 수정을 발견하면 GitHub 이슈를 제출하거나 PR을 보내주세요:
[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues)
[https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls)

다음 명령으로 시작하세요 (도움을 요청할 때 출력을 공유하세요):

```bash
openclaw status
openclaw models status
openclaw doctor
```

각 명령의 역할:

- `openclaw status`: 게이트웨이/에이전트 상태 + 기본 구성의 빠른 스냅샷.
- `openclaw models status`: 프로바이더 인증 + 모델 가용성을 확인.
- `openclaw doctor`: 일반적인 구성/상태 이슈를 검증하고 수리.

기타 유용한 CLI 확인: `openclaw status --all`, `openclaw logs --follow`,
`openclaw gateway status`, `openclaw health --verbose`.

빠른 디버그 루프: [문제 발생 시 첫 60초](#first-60-seconds-if-somethings-broken).
설치 문서: [설치](/install), [설치 프로그램 플래그](/install/installer), [업데이트](/install/updating).

### OpenClaw를 설치하고 설정하는 권장 방법은

저장소에서는 소스에서 실행하고 온보딩 마법사를 사용하는 것을 권장합니다:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

마법사는 UI 에셋도 자동으로 빌드할 수 있습니다. 온보딩 후 일반적으로 포트 **18789**에서 Gateway를 실행합니다.

소스에서 (기여자/개발):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # 첫 실행 시 UI 의존성을 자동 설치
openclaw onboard
```

아직 전역 설치가 없으면 `pnpm openclaw onboard`로 실행하세요.

### 온보딩 후 대시보드를 어떻게 여나요

마법사는 온보딩 직후 깨끗한(토큰이 없는) 대시보드 URL로 브라우저를 열고 요약에 링크를 출력합니다. 해당 탭을 열어두세요; 열리지 않았다면 같은 머신에서 출력된 URL을 복사/붙여넣기하세요.

### localhost와 원격에서 대시보드 토큰 인증은 어떻게 하나요

**Localhost (같은 머신):**

- `http://127.0.0.1:18789/`를 엽니다.
- 인증을 요청하면, `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`)의 토큰을 Control UI 설정에 붙여넣으세요.
- 게이트웨이 호스트에서 가져오기: `openclaw config get gateway.auth.token` (또는 생성: `openclaw doctor --generate-gateway-token`).

**Localhost가 아닌 경우:**

- **Tailscale Serve** (권장): bind를 loopback으로 유지하고, `openclaw gateway --tailscale serve`를 실행하고, `https://<magicdns>/`를 엽니다. `gateway.auth.allowTailscale`이 `true`이면 identity 헤더가 Control UI/WebSocket 인증을 충족합니다 (토큰 불필요, 신뢰할 수 있는 게이트웨이 호스트 가정); HTTP API는 여전히 토큰/비밀번호가 필요합니다.
- **Tailnet bind**: `openclaw gateway --bind tailnet --token "<token>"`을 실행하고, `http://<tailscale-ip>:18789/`를 열고, 대시보드 설정에 토큰을 붙여넣으세요.
- **SSH 터널**: `ssh -N -L 18789:127.0.0.1:18789 user@host` 그런 다음 `http://127.0.0.1:18789/`를 열고 Control UI 설정에 토큰을 붙여넣으세요.

[대시보드](/web/dashboard) 및 [웹 서피스](/web)에서 bind 모드와 인증 세부 사항을 참조하세요.

### 어떤 런타임이 필요한가요

Node **>= 22**가 필요합니다. `pnpm`을 권장합니다. Bun은 Gateway에 **권장하지 않습니다**.

### Raspberry Pi에서 실행되나요

네. Gateway는 가벼워서 - 문서에 **512MB-1GB RAM**, **1 코어**, 약 **500MB** 디스크면 개인 사용에 충분하다고 기재되어 있으며, **Raspberry Pi 4에서 실행할 수 있다**고 언급합니다.

추가 여유 공간(로그, 미디어, 다른 서비스)이 필요하면 **2GB를 권장**하지만, 하드 최소값은 아닙니다.

팁: 작은 Pi/VPS가 Gateway를 호스팅할 수 있고, 로컬 화면/카메라/캔버스 또는 명령 실행을 위해 노트북/폰에 **노드**를 페어링할 수 있습니다. [노드](/nodes)를 참조하세요.

### Raspberry Pi 설치 팁이 있나요

짧은 버전: 작동하지만 거친 부분이 있을 수 있습니다.

- **64비트** OS를 사용하고 Node >= 22를 유지하세요.
- **hackable(git) 설치**를 선호하여 로그를 보고 빠르게 업데이트할 수 있습니다.
- 채널/스킬 없이 시작한 다음 하나씩 추가하세요.
- 이상한 바이너리 이슈가 발생하면 보통 **ARM 호환성** 문제입니다.

문서: [Linux](/platforms/linux), [설치](/install).

### "wake up my friend"에서 멈춤 / 온보딩이 시작되지 않음 어떻게 하나요

해당 화면은 Gateway가 접근 가능하고 인증되어야 합니다. TUI도 첫 번째 hatch 시 자동으로 "Wake up, my friend!"를 보냅니다. **응답 없이** 그 줄이 보이고 토큰이 0에 머물러 있으면 에이전트가 실행되지 않은 것입니다.

1. Gateway를 재시작합니다:

```bash
openclaw gateway restart
```

2. 상태 + 인증을 확인합니다:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

3. 여전히 멈추면 실행합니다:

```bash
openclaw doctor
```

Gateway가 원격이면 터널/Tailscale 연결이 살아있는지, UI가 올바른 Gateway를 가리키는지 확인하세요. [원격 접근](/gateway/remote)을 참조하세요.

### 온보딩을 다시 하지 않고 새 머신(Mac mini)으로 설정을 마이그레이션할 수 있나요

네. **상태 디렉토리**와 **워크스페이스**를 복사한 다음 Doctor를 한 번 실행하세요. 이렇게 하면 **양쪽** 위치를 복사하는 한 봇이 "정확히 같은 상태"(메모리, 세션 기록, 인증, 채널 상태)를 유지합니다:

1. 새 머신에 OpenClaw를 설치합니다.
2. 이전 머신에서 `$OPENCLAW_STATE_DIR` (기본값: `~/.openclaw`)를 복사합니다.
3. 워크스페이스(기본값: `~/.openclaw/workspace`)를 복사합니다.
4. `openclaw doctor`를 실행하고 Gateway 서비스를 재시작합니다.

이렇게 하면 구성, 인증 프로필, WhatsApp 자격 증명, 세션, 메모리가 보존됩니다. 원격 모드인 경우 게이트웨이 호스트가 세션 저장소와 워크스페이스를 소유한다는 점을 기억하세요.

**중요:** 워크스페이스만 GitHub에 커밋/푸시하면 **메모리 + 부트스트랩 파일**은 백업되지만, 세션 기록이나 인증은 **백업되지 않습니다**. 그것들은 `~/.openclaw/` 아래에 있습니다 (예: `~/.openclaw/agents/<agentId>/sessions/`).

관련: [마이그레이션](/install/migrating), [디스크의 데이터 위치](/help/faq#where-does-openclaw-store-its-data),
[에이전트 워크스페이스](/concepts/agent-workspace), [Doctor](/gateway/doctor),
[원격 모드](/gateway/remote).

### 최신 버전의 변경 사항은 어디서 볼 수 있나요

GitHub 변경 로그를 확인하세요:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

최신 항목이 맨 위에 있습니다. 맨 위 섹션이 **Unreleased**로 표시되어 있으면 다음 날짜가 있는 섹션이 최신 배포 버전입니다. 항목은 **Highlights**, **Changes**, **Fixes** (필요 시 docs/기타 섹션 추가)로 그룹화됩니다.

### docs.openclaw.ai에 접속할 수 없습니다 (SSL 오류) 어떻게 하나요

일부 Comcast/Xfinity 연결이 Xfinity Advanced Security를 통해 `docs.openclaw.ai`를 잘못 차단합니다. 비활성화하거나 `docs.openclaw.ai`를 허용 목록에 추가한 다음 재시도하세요. 자세한 내용: [문제 해결](/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
차단 해제를 도와주시려면 여기에 신고해 주세요: [https://spa.xfinity.com/check_url_status](https://spa.xfinity.com/check_url_status).

여전히 사이트에 접속할 수 없으면 문서가 GitHub에 미러링되어 있습니다:
[https://github.com/openclaw/openclaw/tree/main/docs](https://github.com/openclaw/openclaw/tree/main/docs)

### stable과 beta의 차이점은 무엇인가요

**Stable**과 **beta**는 별도의 코드 라인이 아닌 **npm dist-tag**입니다:

- `latest` = stable
- `beta` = 테스트용 초기 빌드

빌드를 **beta**에 배포하고, 테스트한 후, 빌드가 안정적이면 **같은 버전을 `latest`로 프로모션**합니다. 그래서 beta와 stable이 **같은 버전**을 가리킬 수 있습니다.

변경 사항 확인:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

### beta 버전은 어떻게 설치하고 beta와 dev의 차이점은 무엇인가요

**Beta**는 npm dist-tag `beta`입니다 (`latest`와 같을 수 있음).
**Dev**는 `main`의 움직이는 HEAD(git)이며; 배포되면 npm dist-tag `dev`를 사용합니다.

원라이너 (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Windows 설치 프로그램 (PowerShell):
[https://openclaw.ai/install.ps1](https://openclaw.ai/install.ps1)

자세한 내용: [개발 채널](/install/development-channels) 및 [설치 프로그램 플래그](/install/installer).

### 설치와 온보딩은 보통 얼마나 걸리나요

대략적인 가이드:

- **설치:** 2-5분
- **온보딩:** 구성하는 채널/모델 수에 따라 5-15분

멈추면 [설치 프로그램 멈춤](/help/faq#installer-stuck-how-do-i-get-more-feedback)과 [막혔을 때](/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck)의 빠른 디버그 루프를 사용하세요.

### 최신 빌드를 사용해보려면 어떻게 하나요

두 가지 옵션:

1. **Dev 채널 (git checkout):**

```bash
openclaw update --channel dev
```

이렇게 하면 `main` 브랜치로 전환하고 소스에서 업데이트합니다.

2. **Hackable 설치 (설치 프로그램 사이트에서):**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

이렇게 하면 편집할 수 있는 로컬 저장소가 제공되며, git을 통해 업데이트합니다.

수동으로 클린 클론을 선호하면 다음을 사용하세요:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
```

문서: [업데이트](/cli/update), [개발 채널](/install/development-channels),
[설치](/install).

### 설치 프로그램이 멈췄나요 더 많은 피드백을 받으려면

**verbose 출력**으로 설치 프로그램을 다시 실행하세요:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

verbose로 beta 설치:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

hackable(git) 설치:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --verbose
```

Windows (PowerShell) 동등한 방법:

```powershell
# install.ps1에는 아직 전용 -Verbose 플래그가 없습니다.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

추가 옵션: [설치 프로그램 플래그](/install/installer).

### Windows 설치 시 git을 찾을 수 없거나 openclaw가 인식되지 않음

두 가지 일반적인 Windows 이슈:

**1) npm error spawn git / git not found**

- **Git for Windows**를 설치하고 `git`이 PATH에 있는지 확인하세요.
- PowerShell을 닫고 다시 연 다음 설치 프로그램을 다시 실행하세요.

**2) 설치 후 openclaw가 인식되지 않음**

- npm 전역 bin 폴더가 PATH에 없습니다.
- 경로를 확인하세요:

  ```powershell
  npm config get prefix
  ```

- `<prefix>\\bin`이 PATH에 있는지 확인하세요 (대부분의 시스템에서 `%AppData%\\npm`입니다).
- PATH 업데이트 후 PowerShell을 닫고 다시 여세요.

가장 매끄러운 Windows 설정을 원하면 네이티브 Windows 대신 **WSL2**를 사용하세요.
문서: [Windows](/platforms/windows).

### 문서에서 답을 찾지 못했는데 더 나은 답변을 얻으려면

**hackable(git) 설치**를 사용하여 전체 소스와 문서를 로컬에 보유한 다음, **해당 폴더에서** 봇(또는 Claude/Codex)에게 물어보세요. 그러면 저장소를 읽고 정확하게 답변할 수 있습니다.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

자세한 내용: [설치](/install) 및 [설치 프로그램 플래그](/install/installer).

### Linux에 OpenClaw를 어떻게 설치하나요

짧은 답변: Linux 가이드를 따른 다음 온보딩 마법사를 실행하세요.

- Linux 빠른 경로 + 서비스 설치: [Linux](/platforms/linux).
- 전체 안내: [시작하기](/start/getting-started).
- 설치 프로그램 + 업데이트: [설치 및 업데이트](/install/updating).

### VPS에 OpenClaw를 어떻게 설치하나요

모든 Linux VPS에서 작동합니다. 서버에 설치한 다음 SSH/Tailscale을 사용하여 Gateway에 접근하세요.

가이드: [exe.dev](/install/exe-dev), [Hetzner](/install/hetzner), [Fly.io](/install/fly).
원격 접근: [Gateway 원격](/gateway/remote).

### 클라우드/VPS 설치 가이드는 어디에 있나요

일반 프로바이더가 포함된 **호스팅 허브**가 있습니다. 하나를 선택하고 가이드를 따르세요:

- [VPS 호스팅](/vps) (모든 프로바이더를 한 곳에서)
- [Fly.io](/install/fly)
- [Hetzner](/install/hetzner)
- [exe.dev](/install/exe-dev)

클라우드에서의 작동 방식: **Gateway가 서버에서 실행**되고, 노트북/폰에서 Control UI (또는 Tailscale/SSH)를 통해 접근합니다. 상태 + 워크스페이스는 서버에 있으므로 호스트를 신뢰할 수 있는 소스로 취급하고 백업하세요.

클라우드 Gateway에 **노드** (Mac/iOS/Android/headless)를 페어링하여 노트북에서 로컬 화면/카메라/캔버스에 접근하거나 명령을 실행할 수 있으며, Gateway는 클라우드에 유지할 수 있습니다.

허브: [플랫폼](/platforms). 원격 접근: [Gateway 원격](/gateway/remote).
노드: [노드](/nodes), [노드 CLI](/cli/nodes).

### OpenClaw에게 자체 업데이트를 요청할 수 있나요

짧은 답변: **가능하지만 권장하지 않습니다**. 업데이트 흐름은 Gateway를 재시작할 수 있고 (활성 세션이 끊어짐), 깨끗한 git checkout이 필요할 수 있으며, 확인을 요청할 수 있습니다. 더 안전한 방법: 운영자로서 셸에서 업데이트를 실행하세요.

CLI 사용:

```bash
openclaw update
openclaw update status
openclaw update --channel stable|beta|dev
openclaw update --tag <dist-tag|version>
openclaw update --no-restart
```

에이전트에서 자동화해야 하는 경우:

```bash
openclaw update --yes --no-restart
openclaw gateway restart
```

문서: [업데이트](/cli/update), [업데이트하기](/install/updating).

### 온보딩 마법사는 실제로 무엇을 하나요

`openclaw onboard`는 권장 설정 경로입니다. **로컬 모드**에서는 다음을 안내합니다:

- **모델/인증 설정** (Claude 구독에는 Anthropic **setup-token** 권장, OpenAI Codex OAuth 지원, API 키 선택 사항, LM Studio 로컬 모델 지원)
- **워크스페이스** 위치 + 부트스트랩 파일
- **Gateway 설정** (bind/port/auth/tailscale)
- **프로바이더** (WhatsApp, Telegram, Discord, Mattermost (플러그인), Signal, iMessage)
- **데몬 설치** (macOS에서 LaunchAgent; Linux/WSL2에서 systemd user unit)
- **상태 확인** 및 **스킬** 선택

구성된 모델이 알 수 없거나 인증이 누락된 경우에도 경고합니다.

### 이것을 실행하려면 Claude나 OpenAI 구독이 필요한가요

아닙니다. **API 키** (Anthropic/OpenAI/기타)로 OpenClaw를 실행하거나 데이터가 장치에 머무르도록 **로컬 전용 모델**로 실행할 수 있습니다. 구독(Claude Pro/Max 또는 OpenAI Codex)은 해당 프로바이더를 인증하는 선택적 방법입니다.

문서: [Anthropic](/providers/anthropic), [OpenAI](/providers/openai),
[로컬 모델](/gateway/local-models), [모델](/concepts/models).

### API 키 없이 Claude Max 구독을 사용할 수 있나요

네. API 키 대신 **setup-token**으로 인증할 수 있습니다. 이것이 구독 경로입니다.

Claude Pro/Max 구독에는 **API 키가 포함되지 않으므로**, 이것이 구독 계정에 올바른 접근 방식입니다. 중요: 이 사용이 Anthropic의 구독 정책 및 약관에 따라 허용되는지 Anthropic에 확인해야 합니다. 가장 명시적이고 지원되는 경로를 원하면 Anthropic API 키를 사용하세요.

### Anthropic setup-token 인증은 어떻게 작동하나요

`claude setup-token`은 Claude Code CLI를 통해 **토큰 문자열**을 생성합니다 (웹 콘솔에서는 사용할 수 없음). **모든 머신**에서 실행할 수 있습니다. 마법사에서 **Anthropic token (paste setup-token)**을 선택하거나 `openclaw models auth paste-token --provider anthropic`으로 붙여넣으세요. 토큰은 **anthropic** 프로바이더의 인증 프로필로 저장되며 API 키처럼 사용됩니다 (자동 갱신 없음). 자세한 내용: [OAuth](/concepts/oauth).

### Anthropic setup-token은 어디서 찾나요

Anthropic Console에는 **없습니다**. setup-token은 **모든 머신**에서 **Claude Code CLI**로 생성됩니다:

```bash
claude setup-token
```

출력된 토큰을 복사한 다음 마법사에서 **Anthropic token (paste setup-token)**을 선택하세요. 게이트웨이 호스트에서 실행하려면 `openclaw models auth setup-token --provider anthropic`을 사용하세요. 다른 곳에서 `claude setup-token`을 실행했다면 게이트웨이 호스트에서 `openclaw models auth paste-token --provider anthropic`으로 붙여넣으세요. [Anthropic](/providers/anthropic)을 참조하세요.

### Claude 구독 인증(Claude Pro 또는 Max)을 지원하나요

네 - **setup-token**을 통해 지원합니다. OpenClaw는 더 이상 Claude Code CLI OAuth 토큰을 재사용하지 않습니다; setup-token 또는 Anthropic API 키를 사용하세요. 어디서든 토큰을 생성하고 게이트웨이 호스트에 붙여넣으세요. [Anthropic](/providers/anthropic) 및 [OAuth](/concepts/oauth)를 참조하세요.

참고: Claude 구독 접근은 Anthropic의 약관에 의해 관리됩니다. 프로덕션 또는 멀티 사용자 워크로드에는 API 키가 보통 더 안전한 선택입니다.

### Anthropic에서 HTTP 429 rate_limit_error가 보이는 이유는

현재 기간에 대한 **Anthropic 할당량/속도 제한**이 소진되었다는 의미입니다. **Claude 구독** (setup-token 또는 Claude Code OAuth)을 사용하는 경우 기간이 재설정될 때까지 기다리거나 플랜을 업그레이드하세요. **Anthropic API 키**를 사용하는 경우 Anthropic Console에서 사용량/결제를 확인하고 필요에 따라 제한을 높이세요.

팁: **대체 모델**을 설정하면 프로바이더가 속도 제한된 동안에도 OpenClaw가 계속 응답할 수 있습니다. [모델](/cli/models) 및 [OAuth](/concepts/oauth)를 참조하세요.

### AWS Bedrock이 지원되나요

네 - pi-ai의 **Amazon Bedrock (Converse)** 프로바이더를 통해 **수동 구성**으로 지원됩니다. 게이트웨이 호스트에 AWS 자격 증명/리전을 제공하고 모델 구성에 Bedrock 프로바이더 항목을 추가해야 합니다. [Amazon Bedrock](/providers/bedrock) 및 [모델 프로바이더](/providers/models)를 참조하세요. 관리형 키 흐름을 선호하면 Bedrock 앞에 OpenAI 호환 프록시를 사용하는 것도 유효한 옵션입니다.

### Codex 인증은 어떻게 작동하나요

OpenClaw는 OAuth (ChatGPT 로그인)를 통해 **OpenAI Code (Codex)**를 지원합니다. 마법사는 OAuth 흐름을 실행할 수 있으며 적절한 경우 기본 모델을 `openai-codex/gpt-5.3-codex`로 설정합니다. [모델 프로바이더](/concepts/model-providers) 및 [마법사](/start/wizard)를 참조하세요.

### OpenAI 구독 인증(Codex OAuth)을 지원하나요

네. OpenClaw는 **OpenAI Code (Codex) 구독 OAuth**를 완전히 지원합니다. 온보딩 마법사가 OAuth 흐름을 실행할 수 있습니다.

[OAuth](/concepts/oauth), [모델 프로바이더](/concepts/model-providers), [마법사](/start/wizard)를 참조하세요.

### Gemini CLI OAuth를 어떻게 설정하나요

Gemini CLI는 `openclaw.json`의 client id나 secret이 아닌 **플러그인 인증 흐름**을 사용합니다.

단계:

1. 플러그인 활성화: `openclaw plugins enable google-gemini-cli-auth`
2. 로그인: `openclaw models auth login --provider google-gemini-cli --set-default`

이렇게 하면 게이트웨이 호스트의 인증 프로필에 OAuth 토큰이 저장됩니다. 자세한 내용: [모델 프로바이더](/concepts/model-providers).

### 일상적인 대화에 로컬 모델을 사용해도 괜찮나요

보통은 아닙니다. OpenClaw는 큰 컨텍스트 + 강력한 안전성이 필요합니다; 작은 카드는 잘라내기와 누출이 발생합니다. 꼭 사용해야 하면 로컬에서 실행할 수 있는 **가장 큰** MiniMax M2.1 빌드를 실행하고(LM Studio) [/gateway/local-models](/gateway/local-models)를 참조하세요. 더 작은/양자화된 모델은 프롬프트 인젝션 위험을 증가시킵니다 - [보안](/gateway/security)을 참조하세요.

### 호스팅 모델 트래픽을 특정 지역으로 유지하려면

리전 고정 엔드포인트를 선택하세요. OpenRouter는 MiniMax, Kimi, GLM에 대한 미국 호스팅 옵션을 제공합니다; 데이터를 특정 지역에 유지하려면 미국 호스팅 변형을 선택하세요. 대체가 계속 사용 가능하도록 `models.mode: "merge"`를 사용하여 Anthropic/OpenAI를 병렬로 나열할 수 있으며, 선택한 리전 고정 프로바이더를 존중합니다.

### 이것을 설치하려면 Mac Mini를 구매해야 하나요

아닙니다. OpenClaw는 macOS 또는 Linux (Windows는 WSL2를 통해)에서 실행됩니다. Mac mini는 선택 사항입니다 - 일부 사람들은 항상 켜져 있는 호스트로 구매하지만, 작은 VPS, 홈 서버 또는 Raspberry Pi급 박스도 작동합니다.

Mac이 **macOS 전용 도구**에만 필요합니다. iMessage의 경우 [BlueBubbles](/channels/bluebubbles) (권장)를 사용하세요 - BlueBubbles 서버는 모든 Mac에서 실행되며, Gateway는 Linux 또는 다른 곳에서 실행할 수 있습니다. 다른 macOS 전용 도구를 원하면 Mac에서 Gateway를 실행하거나 macOS 노드를 페어링하세요.

문서: [BlueBubbles](/channels/bluebubbles), [노드](/nodes), [Mac 원격 모드](/platforms/mac/remote).

### iMessage 지원을 위해 Mac mini가 필요한가요

Messages에 로그인된 **어떤 macOS 장치**가 필요합니다. Mac mini일 **필요는 없습니다** - 모든 Mac이 작동합니다. iMessage에는 **[BlueBubbles](/channels/bluebubbles)** (권장)를 사용하세요 - BlueBubbles 서버는 macOS에서 실행되며 Gateway는 Linux 또는 다른 곳에서 실행할 수 있습니다.

일반적인 설정:

- Gateway를 Linux/VPS에서 실행하고, Messages에 로그인된 모든 Mac에서 BlueBubbles 서버를 실행합니다.
- 가장 간단한 단일 머신 설정을 원하면 모든 것을 Mac에서 실행합니다.

문서: [BlueBubbles](/channels/bluebubbles), [노드](/nodes),
[Mac 원격 모드](/platforms/mac/remote).

### OpenClaw를 실행하기 위해 Mac mini를 구매하면 MacBook Pro에 연결할 수 있나요

네. **Mac mini가 Gateway를 실행**할 수 있고, MacBook Pro가 **노드** (컴패니언 장치)로 연결할 수 있습니다. 노드는 Gateway를 실행하지 않습니다 - 해당 장치에서 화면/카메라/캔버스 및 `system.run` 같은 추가 기능을 제공합니다.

일반적인 패턴:

- Mac mini에서 Gateway (항상 켜짐).
- MacBook Pro에서 macOS 앱 또는 노드 호스트를 실행하고 Gateway에 페어링.
- `openclaw nodes status` / `openclaw nodes list`로 확인.

문서: [노드](/nodes), [노드 CLI](/cli/nodes).

### Bun을 사용할 수 있나요

Bun은 **권장하지 않습니다**. 특히 WhatsApp과 Telegram에서 런타임 버그가 발생합니다. 안정적인 게이트웨이에는 **Node**를 사용하세요.

그래도 Bun으로 실험하려면 WhatsApp/Telegram 없이 비프로덕션 게이트웨이에서 하세요.

### Telegram allowFrom에 무엇을 넣나요

`channels.telegram.allowFrom`은 **보내는 사람의 Telegram 사용자 ID** (숫자)입니다. 봇 사용자 이름이 아닙니다.

온보딩 마법사는 `@username` 입력을 받아 숫자 ID로 변환하지만, OpenClaw 인증은 숫자 ID만 사용합니다.

더 안전한 방법 (타사 봇 없음):

- 봇에게 DM을 보낸 다음 `openclaw logs --follow`를 실행하고 `from.id`를 읽으세요.

공식 Bot API:

- 봇에게 DM을 보낸 다음 `https://api.telegram.org/bot<bot_token>/getUpdates`를 호출하고 `message.from.id`를 읽으세요.

타사 (덜 프라이빗):

- `@userinfobot` 또는 `@getidsbot`에 DM을 보내세요.

[/channels/telegram](/channels/telegram#access-control-dms--groups)을 참조하세요.

### 여러 사람이 하나의 WhatsApp 번호를 다른 OpenClaw 인스턴스로 사용할 수 있나요

네, **멀티 에이전트 라우팅**을 통해 가능합니다. 각 보내는 사람의 WhatsApp **DM** (peer `kind: "direct"`, sender E.164 형식 `+15551234567`)을 다른 `agentId`에 바인딩하면 각 사람이 자체 워크스페이스와 세션 저장소를 얻습니다. 응답은 여전히 **같은 WhatsApp 계정**에서 발송되며, DM 접근 제어(`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`)는 WhatsApp 계정별로 전역입니다. [멀티 에이전트 라우팅](/concepts/multi-agent) 및 [WhatsApp](/channels/whatsapp)을 참조하세요.

### "빠른 채팅" 에이전트와 "코딩용 Opus" 에이전트를 실행할 수 있나요

네. 멀티 에이전트 라우팅을 사용하세요: 각 에이전트에 자체 기본 모델을 지정한 다음, 인바운드 라우트(프로바이더 계정 또는 특정 피어)를 각 에이전트에 바인딩하세요. 예제 구성은 [멀티 에이전트 라우팅](/concepts/multi-agent)에 있습니다. [모델](/concepts/models) 및 [구성](/gateway/configuration)도 참조하세요.

### Linux에서 Homebrew가 작동하나요

네. Homebrew는 Linux를 지원합니다 (Linuxbrew). 빠른 설정:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

systemd를 통해 OpenClaw를 실행하는 경우 서비스 PATH에 `/home/linuxbrew/.linuxbrew/bin` (또는 brew prefix)이 포함되어야 비로그인 셸에서 `brew`로 설치된 도구가 해결됩니다.
최근 빌드는 Linux systemd 서비스에서 일반적인 사용자 bin 디렉토리(예: `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/.bun/bin`)도 앞에 추가하며, 설정된 경우 `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR`, `FNM_DIR`을 존중합니다.

### hackable(git) 설치와 npm 설치의 차이점은

- **Hackable(git) 설치:** 전체 소스 체크아웃, 편집 가능, 기여자에게 최적.
  빌드를 로컬에서 실행하고 코드/문서를 패치할 수 있습니다.
- **npm 설치:** 전역 CLI 설치, 저장소 없음, "그냥 실행"에 최적.
  업데이트는 npm dist-tag에서 옵니다.

문서: [시작하기](/start/getting-started), [업데이트하기](/install/updating).

### 나중에 npm과 git 설치 간에 전환할 수 있나요

네. 다른 방식을 설치한 다음 Doctor를 실행하면 게이트웨이 서비스가 새 진입점을 가리킵니다. 이것은 **데이터를 삭제하지 않습니다** - OpenClaw 코드 설치만 변경합니다. 상태 (`~/.openclaw`)와 워크스페이스 (`~/.openclaw/workspace`)는 그대로 유지됩니다.

npm에서 git으로:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw doctor
openclaw gateway restart
```

git에서 npm으로:

```bash
npm install -g openclaw@latest
openclaw doctor
openclaw gateway restart
```

Doctor는 게이트웨이 서비스 진입점 불일치를 감지하고 현재 설치에 맞게 서비스 구성을 다시 작성하도록 제안합니다 (자동화에서는 `--repair` 사용).

백업 팁: [백업 전략](/help/faq#whats-the-recommended-backup-strategy)을 참조하세요.

### Gateway를 노트북에서 실행해야 하나요 VPS에서 실행해야 하나요

짧은 답변: **24/7 안정성을 원하면 VPS를 사용하세요**. 가장 낮은 마찰을 원하고 절전/재시작에 괜찮다면 로컬에서 실행하세요.

**노트북 (로컬 Gateway)**

- **장점:** 서버 비용 없음, 로컬 파일에 직접 접근, 라이브 브라우저 창.
- **단점:** 절전/네트워크 끊김 = 연결 해제, OS 업데이트/재부팅으로 중단, 깨어있어야 함.

**VPS / 클라우드**

- **장점:** 항상 켜짐, 안정적인 네트워크, 노트북 절전 이슈 없음, 지속 실행 유지가 더 쉬움.
- **단점:** 주로 headless로 실행 (스크린샷 사용), 원격 파일 접근만 가능, 업데이트를 위해 SSH 필요.

**OpenClaw 관련 참고:** WhatsApp/Telegram/Slack/Mattermost (플러그인)/Discord 모두 VPS에서 잘 작동합니다. 유일한 실질적 트레이드오프는 **headless 브라우저** vs 보이는 창입니다. [브라우저](/tools/browser)를 참조하세요.

**권장 기본값:** 이전에 게이트웨이 연결 끊김이 있었다면 VPS. 로컬은 Mac을 적극적으로 사용하고 로컬 파일 접근이나 보이는 브라우저로 UI 자동화를 원할 때 좋습니다.

### OpenClaw를 전용 머신에서 실행하는 것이 얼마나 중요한가요

필수는 아니지만, **안정성과 격리를 위해 권장**합니다.

- **전용 호스트 (VPS/Mac mini/Pi):** 항상 켜짐, 절전/재부팅 중단이 적음, 더 깨끗한 권한, 지속 실행 유지가 더 쉬움.
- **공유 노트북/데스크탑:** 테스트와 적극적 사용에는 괜찮지만, 머신이 절전하거나 업데이트될 때 일시 정지가 예상됨.

두 가지의 장점을 모두 원하면, Gateway를 전용 호스트에 유지하고 노트북을 로컬 화면/카메라/exec 도구를 위한 **노드**로 페어링하세요. [노드](/nodes)를 참조하세요.
보안 가이드는 [보안](/gateway/security)을 읽으세요.

### 최소 VPS 요구 사항과 권장 OS는

OpenClaw는 가볍습니다. 기본 Gateway + 하나의 채팅 채널:

- **절대 최소:** 1 vCPU, 1GB RAM, ~500MB 디스크.
- **권장:** 여유 공간을 위해 1-2 vCPU, 2GB RAM 이상 (로그, 미디어, 여러 채널). 노드 도구와 브라우저 자동화는 리소스를 많이 사용할 수 있습니다.

OS: **Ubuntu LTS** (또는 최신 Debian/Ubuntu)를 사용하세요. Linux 설치 경로가 거기서 가장 잘 테스트되었습니다.

문서: [Linux](/platforms/linux), [VPS 호스팅](/vps).

### VM에서 OpenClaw를 실행할 수 있나요 요구 사항은

네. VM을 VPS와 같이 취급하세요: 항상 켜져 있어야 하고, 접근 가능해야 하며, Gateway와 활성화한 채널에 충분한 RAM이 있어야 합니다.

기본 가이드:

- **절대 최소:** 1 vCPU, 1GB RAM.
- **권장:** 여러 채널, 브라우저 자동화 또는 미디어 도구를 실행하면 2GB RAM 이상.
- **OS:** Ubuntu LTS 또는 다른 최신 Debian/Ubuntu.

Windows를 사용하는 경우 **WSL2가 가장 쉬운 VM 스타일 설정**이며 도구 호환성이 가장 좋습니다. [Windows](/platforms/windows), [VPS 호스팅](/vps)을 참조하세요.
macOS를 VM에서 실행하는 경우 [macOS VM](/install/macos-vm)을 참조하세요.

## OpenClaw란 무엇인가요?

### OpenClaw란 한 문단으로

OpenClaw는 자신의 장치에서 실행하는 개인 AI 어시스턴트입니다. 이미 사용하는 메시징 서피스(WhatsApp, Telegram, Slack, Mattermost (플러그인), Discord, Google Chat, Signal, iMessage, WebChat)에서 응답하며 지원되는 플랫폼에서 음성 + 라이브 Canvas도 사용할 수 있습니다. **Gateway**는 항상 켜져 있는 컨트롤 플레인이며; 어시스턴트가 제품입니다.

### 가치 제안은 무엇인가요

OpenClaw는 "Claude 래퍼가 아닙니다." **자체 하드웨어**에서 유능한 어시스턴트를 실행할 수 있는 **로컬 우선 컨트롤 플레인**으로, 이미 사용하는 채팅 앱에서 접근 가능하며 상태 저장 세션, 메모리, 도구를 갖추고 있습니다 - 워크플로우의 제어를 호스팅 SaaS에 넘기지 않습니다.

하이라이트:

- **당신의 장치, 당신의 데이터:** 원하는 곳(Mac, Linux, VPS)에서 Gateway를 실행하고 워크스페이스 + 세션 기록을 로컬에 유지.
- **실제 채널, 웹 샌드박스가 아님:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/등, 그리고 지원되는 플랫폼에서 모바일 음성과 Canvas.
- **모델 불가지론적:** Anthropic, OpenAI, MiniMax, OpenRouter 등을 에이전트별 라우팅과 장애 조치로 사용.
- **로컬 전용 옵션:** 로컬 모델을 실행하면 원하면 **모든 데이터가 장치에 머무를 수 있음**.
- **멀티 에이전트 라우팅:** 채널, 계정 또는 작업별로 별도의 에이전트, 각각 자체 워크스페이스와 기본값.
- **오픈 소스와 해킹 가능:** 벤더 종속 없이 검사, 확장, 자체 호스팅.

문서: [Gateway](/gateway), [채널](/channels), [멀티 에이전트](/concepts/multi-agent),
[메모리](/concepts/memory).

### 방금 설정했는데 먼저 무엇을 해야 하나요

좋은 첫 프로젝트:

- 웹사이트 구축 (WordPress, Shopify, 또는 간단한 정적 사이트).
- 모바일 앱 프로토타이핑 (개요, 화면, API 계획).
- 파일과 폴더 정리 (정리, 네이밍, 태깅).
- Gmail 연결 및 요약이나 후속 조치 자동화.

큰 작업을 처리할 수 있지만, 단계로 분할하고 병렬 작업에 서브 에이전트를 사용할 때 가장 잘 작동합니다.

### OpenClaw의 일상적 사용 사례 상위 다섯 가지는

일상적인 성과는 보통 다음과 같습니다:

- **개인 브리핑:** 관심 있는 받은 편지함, 캘린더, 뉴스의 요약.
- **리서치 및 초안:** 빠른 리서치, 요약, 이메일이나 문서의 첫 번째 초안.
- **리마인더 및 후속 조치:** cron 또는 하트비트 기반 알림과 체크리스트.
- **브라우저 자동화:** 양식 작성, 데이터 수집, 웹 작업 반복.
- **크로스 디바이스 조정:** 폰에서 작업을 보내고, Gateway가 서버에서 실행하고, 결과를 채팅으로 돌려받음.

### OpenClaw가 SaaS의 리드 생성 아웃리치 광고 및 블로그에 도움이 될 수 있나요

네, **리서치, 검증, 초안 작성**에 도움이 됩니다. 사이트를 스캔하고, 후보 목록을 작성하고, 잠재 고객을 요약하고, 아웃리치 또는 광고 카피 초안을 작성할 수 있습니다.

**아웃리치나 광고 실행**에는 사람이 루프에 있어야 합니다. 스팸을 피하고, 현지 법률과 플랫폼 정책을 따르고, 보내기 전에 모든 것을 검토하세요. 가장 안전한 패턴은 OpenClaw가 초안을 작성하고 당신이 승인하는 것입니다.

문서: [보안](/gateway/security).

### 웹 개발에서 Claude Code 대비 장점은

OpenClaw는 **개인 어시스턴트**이자 조정 레이어이며, IDE 대체가 아닙니다. 저장소 내에서 가장 빠른 직접 코딩 루프를 위해서는 Claude Code나 Codex를 사용하세요. 내구성 있는 메모리, 크로스 디바이스 접근, 도구 오케스트레이션이 필요할 때 OpenClaw를 사용하세요.

장점:

- **세션 간 영구 메모리 + 워크스페이스**
- **멀티 플랫폼 접근** (WhatsApp, Telegram, TUI, WebChat)
- **도구 오케스트레이션** (브라우저, 파일, 스케줄링, 훅)
- **항상 켜진 Gateway** (VPS에서 실행, 어디서든 상호작용)
- 로컬 브라우저/화면/카메라/exec을 위한 **노드**

쇼케이스: [https://openclaw.ai/showcase](https://openclaw.ai/showcase)

## 스킬 및 자동화

### 저장소를 더럽히지 않고 스킬을 커스터마이즈하려면

저장소 복사본을 편집하는 대신 관리형 오버라이드를 사용하세요. 변경 사항을 `~/.openclaw/skills/<name>/SKILL.md`에 넣거나 (`~/.openclaw/openclaw.json`의 `skills.load.extraDirs`를 통해 폴더 추가). 우선순위는 `<workspace>/skills` > `~/.openclaw/skills` > 번들이므로, 관리형 오버라이드가 git을 건드리지 않고 우선합니다. 업스트림에 적합한 편집만 저장소에 있어야 하고 PR로 나가야 합니다.

### 커스텀 폴더에서 스킬을 로드할 수 있나요

네. `~/.openclaw/openclaw.json`의 `skills.load.extraDirs`를 통해 추가 디렉토리를 추가하세요 (가장 낮은 우선순위). 기본 우선순위는 유지: `<workspace>/skills` -> `~/.openclaw/skills` -> 번들 -> `skills.load.extraDirs`. `clawhub`은 기본적으로 `./skills`에 설치하며, OpenClaw는 이를 `<workspace>/skills`로 취급합니다.

### 다른 작업에 다른 모델을 사용하려면

현재 지원되는 패턴:

- **Cron 작업**: 격리된 작업에서 작업별 `model` 오버라이드를 설정할 수 있습니다.
- **서브 에이전트**: 다른 기본 모델을 가진 별도의 에이전트에 작업을 라우팅합니다.
- **온디맨드 전환**: `/model`을 사용하여 현재 세션 모델을 언제든 전환합니다.

[Cron 작업](/automation/cron-jobs), [멀티 에이전트 라우팅](/concepts/multi-agent), [슬래시 명령](/tools/slash-commands)을 참조하세요.

### 봇이 무거운 작업 중 멈춥니다 어떻게 오프로드하나요

길거나 병렬 작업에는 **서브 에이전트**를 사용하세요. 서브 에이전트는 자체 세션에서 실행되고 요약을 반환하며 메인 채팅의 반응성을 유지합니다.

봇에게 "이 작업을 위한 서브 에이전트를 생성해"라고 요청하거나 `/subagents`를 사용하세요.
채팅에서 `/status`를 사용하여 Gateway가 현재 수행 중인 작업(바쁜지 여부)을 확인하세요.

토큰 팁: 긴 작업과 서브 에이전트 모두 토큰을 소비합니다. 비용이 걱정이면 `agents.defaults.subagents.model`을 통해 서브 에이전트에 더 저렴한 모델을 설정하세요.

문서: [서브 에이전트](/tools/subagents).

### Discord에서 스레드 바인딩 서브에이전트 세션은 어떻게 작동하나요

스레드 바인딩을 사용하세요. Discord 스레드를 서브에이전트 또는 세션 대상에 바인딩하여 해당 스레드의 후속 메시지가 바인딩된 세션에 유지되도록 할 수 있습니다.

기본 흐름:

- `sessions_spawn`을 `thread: true` (선택적으로 영구 후속을 위해 `mode: "session"`)로 생성합니다.
- 또는 `/focus <target>`으로 수동으로 바인딩합니다.
- `/agents`로 바인딩 상태를 검사합니다.
- `/session ttl <duration|off>`로 자동 언포커스를 제어합니다.
- `/unfocus`로 스레드를 분리합니다.

필수 구성:

- 전역 기본값: `session.threadBindings.enabled`, `session.threadBindings.ttlHours`.
- Discord 오버라이드: `channels.discord.threadBindings.enabled`, `channels.discord.threadBindings.ttlHours`.
- 생성 시 자동 바인딩: `channels.discord.threadBindings.spawnSubagentSessions: true`로 설정.

문서: [서브 에이전트](/tools/subagents), [Discord](/channels/discord), [구성 참조](/gateway/configuration-reference), [슬래시 명령](/tools/slash-commands).

### Cron이나 리마인더가 실행되지 않습니다 무엇을 확인해야 하나요

Cron은 Gateway 프로세스 내에서 실행됩니다. Gateway가 지속적으로 실행되지 않으면 예약된 작업이 실행되지 않습니다.

체크리스트:

- cron이 활성화되어 있는지 확인 (`cron.enabled`) 하고 `OPENCLAW_SKIP_CRON`이 설정되어 있지 않은지.
- Gateway가 24/7 실행 중인지 확인 (절전/재시작 없음).
- 작업의 시간대 설정 확인 (`--tz` vs 호스트 시간대).

디버그:

```bash
openclaw cron run <jobId> --force
openclaw cron runs --id <jobId> --limit 50
```

문서: [Cron 작업](/automation/cron-jobs), [Cron vs 하트비트](/automation/cron-vs-heartbeat).

### Linux에서 스킬을 어떻게 설치하나요

**ClawHub** (CLI)를 사용하거나 워크스페이스에 스킬을 드롭하세요. macOS 스킬 UI는 Linux에서 사용할 수 없습니다.
스킬은 [https://clawhub.com](https://clawhub.com)에서 탐색하세요.

ClawHub CLI 설치 (패키지 관리자 선택):

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

### OpenClaw가 일정에 따라 또는 백그라운드에서 지속적으로 작업을 실행할 수 있나요

네. Gateway 스케줄러를 사용하세요:

- **Cron 작업**: 예약 또는 반복 작업 (재시작 간 유지).
- **하트비트**: "메인 세션" 주기적 확인.
- **격리 작업**: 요약을 게시하거나 채팅에 전달하는 자율 에이전트.

문서: [Cron 작업](/automation/cron-jobs), [Cron vs 하트비트](/automation/cron-vs-heartbeat),
[하트비트](/gateway/heartbeat).

### Linux에서 Apple macOS 전용 스킬을 실행할 수 있나요?

직접적으로는 불가능합니다. macOS 스킬은 `metadata.openclaw.os`와 필수 바이너리로 게이팅되며, 스킬은 **Gateway 호스트**에서 적격한 경우에만 시스템 프롬프트에 나타납니다. Linux에서는 게이팅을 오버라이드하지 않는 한 `darwin` 전용 스킬(`apple-notes`, `apple-reminders`, `things-mac` 등)이 로드되지 않습니다.

세 가지 지원 패턴이 있습니다:

**옵션 A - Mac에서 Gateway 실행 (가장 간단).**
macOS 바이너리가 있는 곳에서 Gateway를 실행한 다음, [원격 모드](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)나 Tailscale을 통해 Linux에서 연결합니다. Gateway 호스트가 macOS이므로 스킬이 정상적으로 로드됩니다.

**옵션 B - macOS 노드 사용 (SSH 없음).**
Linux에서 Gateway를 실행하고, macOS 노드(메뉴바 앱)를 페어링하고, Mac에서 **Node Run Commands**를 "Always Ask" 또는 "Always Allow"로 설정합니다. OpenClaw는 필수 바이너리가 노드에 존재할 때 macOS 전용 스킬을 적격으로 취급할 수 있습니다. 에이전트는 `nodes` 도구를 통해 해당 스킬을 실행합니다. "Always Ask"를 선택하면 프롬프트에서 "Always Allow"를 승인하면 해당 명령이 허용 목록에 추가됩니다.

**옵션 C - SSH를 통한 macOS 바이너리 프록시 (고급).**
Linux에서 Gateway를 유지하되, 필수 CLI 바이너리가 Mac에서 실행되는 SSH 래퍼로 해결되도록 합니다. 그런 다음 Linux를 허용하도록 스킬을 오버라이드하여 적격 상태를 유지합니다.

1. 바이너리용 SSH 래퍼 생성 (예: Apple Notes의 `memo`):

   ```bash
   #!/usr/bin/env bash
   set -euo pipefail
   exec ssh -T user@mac-host /opt/homebrew/bin/memo "$@"
   ```

2. Linux 호스트의 `PATH`에 래퍼를 넣습니다 (예: `~/bin/memo`).
3. 스킬 메타데이터(워크스페이스 또는 `~/.openclaw/skills`)를 오버라이드하여 Linux를 허용:

   ```markdown
   ---
   name: apple-notes
   description: Manage Apple Notes via the memo CLI on macOS.
   metadata: { "openclaw": { "os": ["darwin", "linux"], "requires": { "bins": ["memo"] } } }
   ---
   ```

4. 새 세션을 시작하면 스킬 스냅샷이 새로고침됩니다.

### Notion이나 HeyGen 통합이 있나요

현재 내장되어 있지 않습니다.

옵션:

- **커스텀 스킬 / 플러그인:** 안정적인 API 접근에 최적 (Notion/HeyGen 모두 API가 있음).
- **브라우저 자동화:** 코드 없이 작동하지만 느리고 취약함.

클라이언트별 컨텍스트를 유지하려면(에이전시 워크플로우) 간단한 패턴:

- 클라이언트당 하나의 Notion 페이지 (컨텍스트 + 선호도 + 진행 중인 작업).
- 세션 시작 시 에이전트에게 해당 페이지를 가져오도록 요청.

네이티브 통합을 원하면 기능 요청을 열거나 해당 API를 대상으로 하는 스킬을 빌드하세요.

스킬 설치:

```bash
clawhub install <skill-slug>
clawhub update --all
```

ClawHub은 현재 디렉토리의 `./skills` 아래에 설치합니다 (또는 구성된 OpenClaw 워크스페이스로 폴백); OpenClaw는 다음 세션에서 이를 `<workspace>/skills`로 취급합니다. 에이전트 간 공유 스킬의 경우 `~/.openclaw/skills/<name>/SKILL.md`에 넣으세요. 일부 스킬은 Homebrew로 설치된 바이너리를 기대합니다; Linux에서는 Linuxbrew를 의미합니다 (위의 Homebrew Linux FAQ 항목 참조). [스킬](/tools/skills) 및 [ClawHub](/tools/clawhub)을 참조하세요.

### 브라우저 테이크오버를 위한 Chrome 확장을 어떻게 설치하나요

내장 설치 프로그램을 사용한 다음 Chrome에서 unpacked 확장을 로드하세요:

```bash
openclaw browser extension install
openclaw browser extension path
```

그런 다음 Chrome -> `chrome://extensions` -> "개발자 모드" 활성화 -> "압축해제된 확장 프로그램을 로드합니다" -> 해당 폴더를 선택.

전체 가이드 (원격 Gateway + 보안 참고 사항 포함): [Chrome 확장](/tools/chrome-extension)

Gateway가 Chrome과 같은 머신에서 실행되면 (기본 설정), 보통 추가 작업이 **필요하지 않습니다**.
Gateway가 다른 곳에서 실행되면, 브라우저 머신에서 노드 호스트를 실행하여 Gateway가 브라우저 작업을 프록시할 수 있도록 합니다.
여전히 제어하려는 탭에서 확장 버튼을 클릭해야 합니다 (자동 연결되지 않음).

## 샌드박싱 및 메모리

### 전용 샌드박싱 문서가 있나요

네. [샌드박싱](/gateway/sandboxing)을 참조하세요. Docker 관련 설정(전체 게이트웨이를 Docker에서 또는 샌드박스 이미지)은 [Docker](/install/docker)를 참조하세요.

### Docker가 제한적으로 느껴집니다 전체 기능을 활성화하려면

기본 이미지는 보안 우선이며 `node` 사용자로 실행되므로 시스템 패키지, Homebrew 또는 번들 브라우저가 포함되지 않습니다. 더 완전한 설정:

- `OPENCLAW_HOME_VOLUME`으로 `/home/node`를 영속화하여 캐시가 유지되도록 합니다.
- `OPENCLAW_DOCKER_APT_PACKAGES`로 시스템 의존성을 이미지에 빌드합니다.
- 번들 CLI를 통해 Playwright 브라우저 설치:
  `node /app/node_modules/playwright-core/cli.js install chromium`
- `PLAYWRIGHT_BROWSERS_PATH`를 설정하고 해당 경로가 영속화되도록 합니다.

문서: [Docker](/install/docker), [브라우저](/tools/browser).

**DM은 개인적으로 유지하면서 그룹은 하나의 에이전트로 공개/샌드박스로 만들 수 있나요**

네 - 개인 트래픽이 **DM**이고 공개 트래픽이 **그룹**인 경우.

`agents.defaults.sandbox.mode: "non-main"`을 사용하면 그룹/채널 세션(메인이 아닌 키)이 Docker에서 실행되고 메인 DM 세션은 호스트에 유지됩니다. 그런 다음 `tools.sandbox.tools`를 통해 샌드박스 세션에서 사용 가능한 도구를 제한하세요.

설정 안내 + 예제 구성: [그룹: 개인 DM + 공개 그룹](/channels/groups#pattern-personal-dms-public-groups-single-agent)

핵심 구성 참조: [Gateway 구성](/gateway/configuration#agentsdefaultssandbox)

### 호스트 폴더를 샌드박스에 바인드하려면

`agents.defaults.sandbox.docker.binds`를 `["host:path:mode"]`로 설정하세요 (예: `"/home/user/src:/src:ro"`). 전역 + 에이전트별 바인드가 병합됩니다; `scope: "shared"`일 때 에이전트별 바인드는 무시됩니다. 민감한 것에는 `:ro`를 사용하고 바인드가 샌드박스 파일시스템 벽을 우회한다는 점을 기억하세요. 예제와 안전 참고 사항은 [샌드박싱](/gateway/sandboxing#custom-bind-mounts) 및 [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check)를 참조하세요.

### 메모리는 어떻게 작동하나요

OpenClaw 메모리는 에이전트 워크스페이스의 Markdown 파일입니다:

- `memory/YYYY-MM-DD.md`에 일일 노트
- `MEMORY.md`에 큐레이션된 장기 노트 (메인/비공개 세션만)

OpenClaw는 자동 압축 전에 모델이 내구성 있는 노트를 작성하도록 상기시키는 **무음 사전 압축 메모리 플러시**도 실행합니다. 이는 워크스페이스가 쓰기 가능한 경우에만 실행됩니다 (읽기 전용 샌드박스에서는 건너뜀). [메모리](/concepts/memory)를 참조하세요.

### 메모리가 계속 잊어버립니다 어떻게 유지시키나요

봇에게 **사실을 메모리에 작성**하도록 요청하세요. 장기 노트는 `MEMORY.md`에, 단기 컨텍스트는 `memory/YYYY-MM-DD.md`에 넣습니다.

이 부분은 아직 개선 중입니다. 모델에게 메모리를 저장하도록 상기시키면 도움이 됩니다; 모델은 무엇을 해야 하는지 알고 있습니다. 계속 잊어버리면, 매 실행마다 Gateway가 같은 워크스페이스를 사용하는지 확인하세요.

문서: [메모리](/concepts/memory), [에이전트 워크스페이스](/concepts/agent-workspace).

### 시맨틱 메모리 검색에 OpenAI API 키가 필요한가요

**OpenAI 임베딩**을 사용하는 경우에만. Codex OAuth는 chat/completions를 커버하며 임베딩 접근 권한을 **부여하지 않으므로**, **Codex로 로그인(OAuth 또는 Codex CLI 로그인)**은 시맨틱 메모리 검색에 도움이 되지 않습니다. OpenAI 임베딩은 여전히 실제 API 키(`OPENAI_API_KEY` 또는 `models.providers.openai.apiKey`)가 필요합니다.

프로바이더를 명시적으로 설정하지 않으면, OpenClaw는 API 키(인증 프로필, `models.providers.*.apiKey`, 또는 환경 변수)를 해결할 수 있을 때 프로바이더를 자동 선택합니다. OpenAI 키가 해결되면 OpenAI를 선호하고, 그렇지 않으면 Gemini 키가 해결되면 Gemini, 그 다음 Voyage, Mistral 순입니다. 원격 키가 없으면 구성할 때까지 메모리 검색이 비활성화됩니다. 로컬 모델 경로가 구성되고 존재하면 OpenClaw는 `local`을 선호합니다.

로컬을 유지하고 싶으면 `memorySearch.provider = "local"` (선택적으로 `memorySearch.fallback = "none"`)을 설정하세요. Gemini 임베딩을 원하면 `memorySearch.provider = "gemini"`로 설정하고 `GEMINI_API_KEY` (또는 `memorySearch.remote.apiKey`)를 제공하세요. **OpenAI, Gemini, Voyage, Mistral, 또는 local** 임베딩 모델을 지원합니다 - 설정 세부 사항은 [메모리](/concepts/memory)를 참조하세요.

### 메모리는 영구적으로 유지되나요 제한은 무엇인가요

메모리 파일은 디스크에 있으며 삭제할 때까지 유지됩니다. 제한은 모델이 아닌 스토리지입니다. **세션 컨텍스트**는 여전히 모델 컨텍스트 윈도우에 의해 제한되므로, 긴 대화는 압축되거나 잘릴 수 있습니다. 이것이 메모리 검색이 존재하는 이유입니다 - 관련 부분만 컨텍스트로 다시 가져옵니다.

문서: [메모리](/concepts/memory), [컨텍스트](/concepts/context).

## 디스크의 데이터 위치

### OpenClaw와 함께 사용되는 모든 데이터가 로컬에 저장되나요

아니요 - **OpenClaw의 상태는 로컬**이지만, **외부 서비스는 여전히 전송한 내용을 볼 수 있습니다**.

- **기본적으로 로컬:** 세션, 메모리 파일, 구성, 워크스페이스는 Gateway 호스트에 있습니다 (`~/.openclaw` + 워크스페이스 디렉토리).
- **필연적으로 원격:** 모델 프로바이더(Anthropic/OpenAI/등)에 보내는 메시지는 해당 API로 가고, 채팅 플랫폼(WhatsApp/Telegram/Slack/등)은 서버에 메시지 데이터를 저장합니다.
- **사용 범위를 제어합니다:** 로컬 모델을 사용하면 프롬프트가 머신에 유지되지만, 채널 트래픽은 여전히 채널 서버를 통과합니다.

관련: [에이전트 워크스페이스](/concepts/agent-workspace), [메모리](/concepts/memory).

### OpenClaw는 데이터를 어디에 저장하나요

모든 것이 `$OPENCLAW_STATE_DIR` (기본값: `~/.openclaw`) 아래에 있습니다:

| 경로                                                            | 목적                                                         |
| --------------------------------------------------------------- | ------------------------------------------------------------ |
| `$OPENCLAW_STATE_DIR/openclaw.json`                             | 메인 구성 (JSON5)                                            |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json`                    | 레거시 OAuth import (첫 사용 시 인증 프로필로 복사)          |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth-profiles.json` | 인증 프로필 (OAuth + API 키)                                 |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/agent/auth.json`          | 런타임 인증 캐시 (자동 관리)                                 |
| `$OPENCLAW_STATE_DIR/credentials/`                              | 프로바이더 상태 (예: `whatsapp/<accountId>/creds.json`)      |
| `$OPENCLAW_STATE_DIR/agents/`                                   | 에이전트별 상태 (agentDir + 세션)                            |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/`                | 대화 기록 및 상태 (에이전트별)                               |
| `$OPENCLAW_STATE_DIR/agents/<agentId>/sessions/sessions.json`   | 세션 메타데이터 (에이전트별)                                 |

레거시 단일 에이전트 경로: `~/.openclaw/agent/*` (`openclaw doctor`로 마이그레이션).

**워크스페이스** (AGENTS.md, 메모리 파일, 스킬 등)는 별도이며 `agents.defaults.workspace`를 통해 구성됩니다 (기본값: `~/.openclaw/workspace`).

### AGENTS.md SOUL.md USER.md MEMORY.md는 어디에 있어야 하나요

이 파일들은 `~/.openclaw`가 아닌 **에이전트 워크스페이스**에 있습니다.

- **워크스페이스 (에이전트별)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
  `MEMORY.md` (또는 `memory.md`), `memory/YYYY-MM-DD.md`, 선택적 `HEARTBEAT.md`.
- **상태 디렉토리 (`~/.openclaw`)**: 구성, 자격 증명, 인증 프로필, 세션, 로그,
  공유 스킬 (`~/.openclaw/skills`).

기본 워크스페이스는 `~/.openclaw/workspace`이며, 다음을 통해 구성 가능합니다:

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

재시작 후 봇이 "잊어버리면", 매 실행마다 Gateway가 같은 워크스페이스를 사용하는지 확인하세요 (그리고 기억하세요: 원격 모드는 로컬 노트북이 아닌 **게이트웨이 호스트의** 워크스페이스를 사용합니다).

팁: 내구성 있는 행동이나 선호도를 원하면, 채팅 기록에 의존하기보다 봇에게 **AGENTS.md 또는 MEMORY.md에 작성하도록** 요청하세요.

[에이전트 워크스페이스](/concepts/agent-workspace) 및 [메모리](/concepts/memory)를 참조하세요.

### 권장 백업 전략은

**에이전트 워크스페이스**를 **비공개** git 저장소에 넣고 비공개로 백업하세요 (예: GitHub 비공개). 이렇게 하면 메모리 + AGENTS/SOUL/USER 파일이 캡처되며, 나중에 어시스턴트의 "마음"을 복원할 수 있습니다.

`~/.openclaw` 아래의 것은 커밋하지 **마세요** (자격 증명, 세션, 토큰). 전체 복원이 필요하면 워크스페이스와 상태 디렉토리를 별도로 백업하세요 (위의 마이그레이션 질문 참조).

문서: [에이전트 워크스페이스](/concepts/agent-workspace).

### OpenClaw를 완전히 제거하려면

전용 가이드를 참조하세요: [제거](/install/uninstall).

### 에이전트가 워크스페이스 밖에서 작업할 수 있나요

네. 워크스페이스는 **기본 cwd**이자 메모리 앵커이며, 하드 샌드박스가 아닙니다. 상대 경로는 워크스페이스 내에서 해결되지만, 절대 경로는 샌드박싱이 활성화되지 않는 한 다른 호스트 위치에 접근할 수 있습니다. 격리가 필요하면 [`agents.defaults.sandbox`](/gateway/sandboxing) 또는 에이전트별 샌드박스 설정을 사용하세요. 저장소를 기본 작업 디렉토리로 만들려면 해당 에이전트의 `workspace`를 저장소 루트로 지정하세요. OpenClaw 저장소는 단지 소스 코드입니다; 에이전트가 의도적으로 그 안에서 작업하기를 원하지 않는 한 워크스페이스를 별도로 유지하세요.

예제 (저장소를 기본 cwd로):

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo",
    },
  },
}
```

### 원격 모드에서 세션 저장소는 어디에 있나요

세션 상태는 **게이트웨이 호스트**가 소유합니다. 원격 모드인 경우 관심 있는 세션 저장소는 로컬 노트북이 아닌 원격 머신에 있습니다. [세션 관리](/concepts/session)를 참조하세요.

## 구성 기본 사항

### 구성 형식은 무엇이고 어디에 있나요

OpenClaw는 `$OPENCLAW_CONFIG_PATH` (기본값: `~/.openclaw/openclaw.json`)에서 선택적 **JSON5** 구성을 읽습니다:

```
$OPENCLAW_CONFIG_PATH
```

파일이 없으면 안전한 기본값을 사용합니다 (기본 워크스페이스 `~/.openclaw/workspace` 포함).

### gateway.bind를 lan 또는 tailnet으로 설정했더니 아무것도 리슨하지 않음 UI가 unauthorized라고 함

비루프백 바인드는 **인증이 필요합니다**. `gateway.auth.mode` + `gateway.auth.token`을 구성하세요 (또는 `OPENCLAW_GATEWAY_TOKEN` 사용).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me",
    },
  },
}
```

참고:

- `gateway.remote.token`은 **원격 CLI 호출** 전용입니다; 로컬 게이트웨이 인증을 활성화하지 않습니다.
- Control UI는 `connect.params.auth.token` (앱/UI 설정에 저장)을 통해 인증합니다. URL에 토큰을 넣지 마세요.

### 왜 localhost에서도 토큰이 필요한가요

OpenClaw는 루프백을 포함하여 기본적으로 토큰 인증을 시행합니다. 토큰이 구성되지 않으면 게이트웨이 시작 시 자동 생성하여 `gateway.auth.token`에 저장하므로, **로컬 WS 클라이언트도 인증해야 합니다**. 이렇게 하면 다른 로컬 프로세스가 Gateway를 호출하는 것을 차단합니다.

루프백을 **정말** 열려면 구성에서 `gateway.auth.mode: "none"`을 명시적으로 설정하세요. Doctor가 언제든 토큰을 생성할 수 있습니다: `openclaw doctor --generate-gateway-token`.

### 구성 변경 후 재시작해야 하나요

Gateway는 구성을 감시하고 핫 리로드를 지원합니다:

- `gateway.reload.mode: "hybrid"` (기본값): 안전한 변경은 핫 적용, 중요한 변경은 재시작
- `hot`, `restart`, `off`도 지원

### 웹 검색(및 웹 fetch)을 어떻게 활성화하나요

`web_fetch`는 API 키 없이 작동합니다. `web_search`에는 Brave Search API 키가 필요합니다. **권장:** `openclaw configure --section web`을 실행하여 `tools.web.search.apiKey`에 저장하세요. 환경 대안: Gateway 프로세스에 `BRAVE_API_KEY`를 설정.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
      },
      fetch: {
        enabled: true,
      },
    },
  },
}
```

참고:

- 허용 목록을 사용하면 `web_search`/`web_fetch` 또는 `group:web`을 추가하세요.
- `web_fetch`는 기본적으로 활성화됩니다 (명시적으로 비활성화하지 않는 한).
- 데몬은 `~/.openclaw/.env` (또는 서비스 환경)에서 환경 변수를 읽습니다.

문서: [웹 도구](/tools/web).

### config.apply가 내 구성을 지웠습니다 복구하고 방지하려면

`config.apply`는 **전체 구성**을 교체합니다. 부분 객체를 보내면 나머지가 모두 제거됩니다.

복구:

- 백업에서 복원 (git 또는 복사된 `~/.openclaw/openclaw.json`).
- 백업이 없으면 `openclaw doctor`를 다시 실행하고 채널/모델을 재구성하세요.
- 예상치 못한 경우 버그를 제출하고 마지막으로 알려진 구성이나 백업을 포함하세요.
- 로컬 코딩 에이전트가 로그나 기록에서 작동하는 구성을 재구성할 수 있는 경우가 많습니다.

방지:

- 작은 변경에는 `openclaw config set`을 사용하세요.
- 대화형 편집에는 `openclaw configure`를 사용하세요.

문서: [Config](/cli/config), [Configure](/cli/configure), [Doctor](/gateway/doctor).

### 여러 장치에 걸쳐 전문화된 워커가 있는 중앙 Gateway를 어떻게 실행하나요

일반적인 패턴은 **하나의 Gateway** (예: Raspberry Pi)와 **노드** 및 **에이전트**입니다:

- **Gateway (중앙):** 채널(Signal/WhatsApp), 라우팅, 세션을 소유.
- **노드 (장치):** Mac/iOS/Android가 주변 장치로 연결되어 로컬 도구(`system.run`, `canvas`, `camera`)를 노출.
- **에이전트 (워커):** 특수 역할을 위한 별도의 두뇌/워크스페이스 (예: "Hetzner ops", "개인 데이터").
- **서브 에이전트:** 병렬성이 필요할 때 메인 에이전트에서 백그라운드 작업을 생성.
- **TUI:** Gateway에 연결하여 에이전트/세션을 전환.

문서: [노드](/nodes), [원격 접근](/gateway/remote), [멀티 에이전트 라우팅](/concepts/multi-agent), [서브 에이전트](/tools/subagents), [TUI](/web/tui).

### OpenClaw 브라우저를 headless로 실행할 수 있나요

네. 구성 옵션입니다:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } },
    },
  },
}
```

기본값은 `false` (headful)입니다. Headless는 일부 사이트에서 안티봇 검사를 더 잘 트리거할 수 있습니다. [브라우저](/tools/browser)를 참조하세요.

Headless는 **동일한 Chromium 엔진**을 사용하며 대부분의 자동화(양식, 클릭, 스크래핑, 로그인)에서 작동합니다. 주요 차이점:

- 보이는 브라우저 창 없음 (시각적 확인이 필요하면 스크린샷 사용).
- 일부 사이트는 headless 모드에서 자동화에 더 엄격합니다 (CAPTCHA, 안티봇).
  예를 들어, X/Twitter는 종종 headless 세션을 차단합니다.

### 브라우저 제어에 Brave를 어떻게 사용하나요

`browser.executablePath`를 Brave 바이너리 (또는 다른 Chromium 기반 브라우저)로 설정하고 Gateway를 재시작하세요.
전체 구성 예제는 [브라우저](/tools/browser#use-brave-or-another-chromium-based-browser)를 참조하세요.

## 원격 게이트웨이 및 노드

### Telegram 게이트웨이 노드 간에 명령이 어떻게 전파되나요

Telegram 메시지는 **게이트웨이**에서 처리됩니다. 게이트웨이는 에이전트를 실행하고 노드 도구가 필요할 때만 **Gateway WebSocket**을 통해 노드를 호출합니다:

Telegram -> Gateway -> Agent -> `node.*` -> Node -> Gateway -> Telegram

노드는 인바운드 프로바이더 트래픽을 보지 않습니다; 노드 RPC 호출만 수신합니다.

### Gateway가 원격에 호스팅되어 있을 때 에이전트가 내 컴퓨터에 접근하려면

짧은 답변: **컴퓨터를 노드로 페어링하세요**. Gateway는 다른 곳에서 실행되지만, Gateway WebSocket을 통해 로컬 머신의 `node.*` 도구(화면, 카메라, 시스템)를 호출할 수 있습니다.

일반적인 설정:

1. 항상 켜진 호스트(VPS/홈 서버)에서 Gateway를 실행합니다.
2. Gateway 호스트 + 컴퓨터를 같은 tailnet에 넣습니다.
3. Gateway WS가 접근 가능한지 확인합니다 (tailnet bind 또는 SSH 터널).
4. 로컬에서 macOS 앱을 열고 **Remote over SSH** 모드 (또는 직접 tailnet)로 연결하여 노드로 등록할 수 있습니다.
5. Gateway에서 노드를 승인합니다:

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

별도의 TCP 브리지가 필요하지 않습니다; 노드는 Gateway WebSocket을 통해 연결됩니다.

보안 알림: macOS 노드를 페어링하면 해당 머신에서 `system.run`이 허용됩니다. 신뢰하는 장치만 페어링하고 [보안](/gateway/security)을 검토하세요.

문서: [노드](/nodes), [Gateway 프로토콜](/gateway/protocol), [macOS 원격 모드](/platforms/mac/remote), [보안](/gateway/security).

### Tailscale이 연결되었지만 응답이 없습니다 어떻게 하나요

기본 사항을 확인하세요:

- Gateway가 실행 중: `openclaw gateway status`
- Gateway 상태: `openclaw status`
- 채널 상태: `openclaw channels status`

그런 다음 인증과 라우팅을 확인하세요:

- Tailscale Serve를 사용하면 `gateway.auth.allowTailscale`이 올바르게 설정되어 있는지 확인하세요.
- SSH 터널을 통해 연결하면 로컬 터널이 올바른 포트를 가리키는지 확인하세요.
- 허용 목록(DM 또는 그룹)에 계정이 포함되어 있는지 확인하세요.

문서: [Tailscale](/gateway/tailscale), [원격 접근](/gateway/remote), [채널](/channels).

### 두 OpenClaw 인스턴스가 서로 통신할 수 있나요 (로컬 + VPS)

네. 내장된 "봇 대 봇" 브리지는 없지만 몇 가지 안정적인 방법으로 연결할 수 있습니다:

**가장 간단:** 두 봇이 접근할 수 있는 일반 채팅 채널(Telegram/Slack/WhatsApp)을 사용합니다. Bot A가 Bot B에게 메시지를 보내면 Bot B가 평소처럼 응답합니다.

**CLI 브리지 (범용):** 다른 Gateway에 `openclaw agent --message ... --deliver`를 호출하는 스크립트를 실행하여 다른 봇이 듣고 있는 채팅을 대상으로 합니다. 한 봇이 원격 VPS에 있으면 SSH/Tailscale을 통해 CLI가 해당 원격 Gateway를 가리키도록 합니다 ([원격 접근](/gateway/remote) 참조).

예제 패턴 (대상 Gateway에 접근할 수 있는 머신에서 실행):

```bash
openclaw agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

팁: 두 봇이 무한 루프를 돌지 않도록 가드레일을 추가하세요 (멘션 전용, 채널 허용 목록, 또는 "봇 메시지에 응답하지 않음" 규칙).

문서: [원격 접근](/gateway/remote), [Agent CLI](/cli/agent), [Agent send](/tools/agent-send).

### 여러 에이전트에 별도의 VPS가 필요한가요

아닙니다. 하나의 Gateway가 여러 에이전트를 호스팅할 수 있으며, 각각 자체 워크스페이스, 모델 기본값, 라우팅을 가집니다. 이것이 정상적인 설정이며 에이전트당 하나의 VPS를 실행하는 것보다 훨씬 저렴하고 간단합니다.

하드 격리(보안 경계)가 필요하거나 공유하고 싶지 않은 매우 다른 구성이 있을 때만 별도의 VPS를 사용하세요. 그렇지 않으면 하나의 Gateway를 유지하고 여러 에이전트 또는 서브 에이전트를 사용하세요.

### VPS에서 SSH 대신 개인 노트북에서 노드를 사용하는 이점이 있나요

네 - 노드는 원격 Gateway에서 노트북에 접근하는 1급 방법이며, 셸 접근 이상의 것을 잠금 해제합니다. Gateway는 macOS/Linux (Windows는 WSL2를 통해)에서 실행되며 가볍습니다 (작은 VPS나 Raspberry Pi급 박스면 충분; 4 GB RAM이면 충분), 따라서 일반적인 설정은 항상 켜진 호스트와 노트북을 노드로 사용하는 것입니다.

- **인바운드 SSH 불필요.** 노드는 Gateway WebSocket으로 아웃바운드 연결하고 장치 페어링을 사용합니다.
- **더 안전한 실행 제어.** `system.run`은 해당 노트북의 노드 허용 목록/승인으로 게이팅됩니다.
- **더 많은 장치 도구.** 노드는 `system.run` 외에 `canvas`, `camera`, `screen`을 노출합니다.
- **로컬 브라우저 자동화.** Gateway는 VPS에 유지하되, 로컬에서 Chrome을 실행하고 Chrome 확장 + 노트북의 노드 호스트로 제어를 릴레이합니다.

SSH는 임시 셸 접근에 적합하지만, 노드는 지속적인 에이전트 워크플로우와 장치 자동화에 더 간단합니다.

문서: [노드](/nodes), [노드 CLI](/cli/nodes), [Chrome 확장](/tools/chrome-extension).

### 두 번째 노트북에 설치해야 하나요 노드만 추가하면 되나요

두 번째 노트북에서 **로컬 도구** (화면/카메라/exec)만 필요하면 **노드**로 추가하세요. 이렇게 하면 단일 Gateway를 유지하고 구성 중복을 피할 수 있습니다. 로컬 노드 도구는 현재 macOS 전용이지만, 다른 OS로 확장할 계획입니다.

**하드 격리**가 필요하거나 두 개의 완전히 별도의 봇이 필요할 때만 두 번째 Gateway를 설치하세요.

문서: [노드](/nodes), [노드 CLI](/cli/nodes), [여러 게이트웨이](/gateway/multiple-gateways).

### 노드는 게이트웨이 서비스를 실행하나요

아닙니다. 의도적으로 격리된 프로필을 실행하지 않는 한 **호스트당 하나의 게이트웨이**만 실행해야 합니다 ([여러 게이트웨이](/gateway/multiple-gateways) 참조). 노드는 게이트웨이에 연결하는 주변 장치입니다 (iOS/Android 노드, 또는 메뉴바 앱의 macOS "노드 모드"). headless 노드 호스트와 CLI 제어는 [노드 호스트 CLI](/cli/node)를 참조하세요.

`gateway`, `discovery`, `canvasHost` 변경에는 전체 재시작이 필요합니다.

### 구성을 적용하는 API / RPC 방법이 있나요

네. `config.apply`는 전체 구성을 검증 + 작성하고 작업의 일부로 Gateway를 재시작합니다.

### 최초 설치를 위한 최소한의 합리적인 구성은

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

이렇게 하면 워크스페이스를 설정하고 봇을 트리거할 수 있는 사람을 제한합니다.

### VPS에 Tailscale을 설정하고 Mac에서 연결하려면

최소 단계:

1. **VPS에 설치 + 로그인**

   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   sudo tailscale up
   ```

2. **Mac에 설치 + 로그인**
   - Tailscale 앱을 사용하고 같은 tailnet에 로그인하세요.
3. **MagicDNS 활성화 (권장)**
   - Tailscale 관리자 콘솔에서 MagicDNS를 활성화하면 VPS에 안정적인 이름이 부여됩니다.
4. **tailnet 호스트 이름 사용**
   - SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   - Gateway WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

SSH 없이 Control UI를 원하면 VPS에서 Tailscale Serve를 사용하세요:

```bash
openclaw gateway --tailscale serve
```

이렇게 하면 게이트웨이가 루프백에 바인딩되고 Tailscale을 통해 HTTPS를 노출합니다. [Tailscale](/gateway/tailscale)을 참조하세요.

### Mac 노드를 원격 Gateway에 연결하려면 (Tailscale Serve)

Serve는 **Gateway Control UI + WS**를 노출합니다. 노드는 같은 Gateway WS 엔드포인트를 통해 연결됩니다.

권장 설정:

1. **VPS + Mac이 같은 tailnet에 있는지 확인**.
2. **macOS 앱을 원격 모드로 사용** (SSH 대상은 tailnet 호스트 이름 가능).
   앱이 Gateway 포트를 터널링하고 노드로 연결합니다.
3. **게이트웨이에서 노드 승인**:

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

문서: [Gateway 프로토콜](/gateway/protocol), [Discovery](/gateway/discovery), [macOS 원격 모드](/platforms/mac/remote).

## 환경 변수 및 .env 로딩

### OpenClaw는 환경 변수를 어떻게 로드하나요

OpenClaw는 부모 프로세스(셸, launchd/systemd, CI 등)에서 환경 변수를 읽고 추가로 로드합니다:

- 현재 작업 디렉토리의 `.env`
- `~/.openclaw/.env` (일명 `$OPENCLAW_STATE_DIR/.env`)의 전역 대체 `.env`

어떤 `.env` 파일도 기존 환경 변수를 덮어쓰지 않습니다.

구성에서 인라인 환경 변수를 정의할 수도 있습니다 (프로세스 환경에서 누락된 경우에만 적용):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

전체 우선순위와 소스는 [/environment](/help/environment)를 참조하세요.

### 서비스를 통해 Gateway를 시작했더니 환경 변수가 사라졌습니다 어떻게 하나요

두 가지 일반적인 수정:

1. 누락된 키를 `~/.openclaw/.env`에 넣으면 서비스가 셸 환경을 상속하지 않을 때도 로드됩니다.
2. 셸 import 활성화 (선택적 편의):

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

이렇게 하면 로그인 셸을 실행하고 누락된 예상 키만 import합니다 (덮어쓰지 않음). 환경 변수 동등물:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

### COPILOT_GITHUB_TOKEN을 설정했는데 models status에 Shell env off라고 나옵니다 왜 그런가요

`openclaw models status`는 **셸 환경 import**가 활성화되어 있는지 보고합니다. "Shell env: off"는 환경 변수가 누락되었다는 의미가 **아닙니다** - OpenClaw가 로그인 셸을 자동으로 로드하지 않는다는 것입니다.

Gateway가 서비스(launchd/systemd)로 실행되면 셸 환경을 상속하지 않습니다. 다음 중 하나를 수행하여 수정:

1. `~/.openclaw/.env`에 토큰을 넣습니다:

   ```
   COPILOT_GITHUB_TOKEN=...
   ```

2. 또는 셸 import를 활성화합니다 (`env.shellEnv.enabled: true`).
3. 또는 구성 `env` 블록에 추가합니다 (누락된 경우에만 적용).

그런 다음 게이트웨이를 재시작하고 다시 확인:

```bash
openclaw models status
```

Copilot 토큰은 `COPILOT_GITHUB_TOKEN` (또한 `GH_TOKEN` / `GITHUB_TOKEN`)에서 읽힙니다.
[/concepts/model-providers](/concepts/model-providers) 및 [/environment](/help/environment)를 참조하세요.

## 세션 및 다중 채팅

### 새 대화를 시작하려면

독립 메시지로 `/new` 또는 `/reset`을 보내세요. [세션 관리](/concepts/session)를 참조하세요.

### /new를 보내지 않으면 세션이 자동으로 초기화되나요

네. 세션은 `session.idleMinutes` (기본값 **60**) 후 만료됩니다. **다음** 메시지는 해당 채팅 키에 대한 새 세션 id를 시작합니다. 이것은 대화록을 삭제하지 않습니다 - 새 세션을 시작할 뿐입니다.

```json5
{
  session: {
    idleMinutes: 240,
  },
}
```

### OpenClaw 인스턴스 팀을 만들 수 있나요 CEO 하나와 많은 에이전트

네, **멀티 에이전트 라우팅**과 **서브 에이전트**를 통해 가능합니다. 하나의 코디네이터 에이전트와 각각 자체 워크스페이스와 모델을 가진 여러 워커 에이전트를 만들 수 있습니다.

그러나 이것은 **재미있는 실험**으로 보는 것이 가장 좋습니다. 토큰을 많이 소비하고 종종 별도의 세션이 있는 하나의 봇을 사용하는 것보다 덜 효율적입니다. 우리가 구상하는 일반적인 모델은 대화하는 하나의 봇이며, 병렬 작업을 위한 다른 세션이 있는 것입니다. 해당 봇은 필요할 때 서브 에이전트도 생성할 수 있습니다.

문서: [멀티 에이전트 라우팅](/concepts/multi-agent), [서브 에이전트](/tools/subagents), [Agents CLI](/cli/agents).

### 작업 중 컨텍스트가 잘린 이유는 어떻게 방지하나요

세션 컨텍스트는 모델 윈도우에 의해 제한됩니다. 긴 채팅, 큰 도구 출력, 많은 파일이 압축이나 잘림을 트리거할 수 있습니다.

도움이 되는 것:

- 봇에게 현재 상태를 요약하고 파일에 작성하도록 요청하세요.
- 긴 작업 전에 `/compact`를 사용하고 주제를 전환할 때 `/new`를 사용하세요.
- 중요한 컨텍스트를 워크스페이스에 유지하고 봇에게 다시 읽도록 요청하세요.
- 메인 채팅이 더 작게 유지되도록 길거나 병렬 작업에 서브 에이전트를 사용하세요.
- 자주 발생하면 더 큰 컨텍스트 윈도우를 가진 모델을 선택하세요.

### OpenClaw를 완전히 초기화하되 설치는 유지하려면

reset 명령을 사용하세요:

```bash
openclaw reset
```

비대화형 전체 초기화:

```bash
openclaw reset --scope full --yes --non-interactive
```

그런 다음 온보딩을 다시 실행:

```bash
openclaw onboard --install-daemon
```

참고:

- 온보딩 마법사는 기존 구성이 있으면 **Reset**도 제공합니다. [마법사](/start/wizard)를 참조하세요.
- 프로필(`--profile` / `OPENCLAW_PROFILE`)을 사용했다면 각 상태 디렉토리를 초기화하세요 (기본값 `~/.openclaw-<profile>`).
- 개발 초기화: `openclaw gateway --dev --reset` (dev 전용; dev 구성 + 자격 증명 + 세션 + 워크스페이스를 삭제).

### "context too large" 오류가 발생합니다 초기화하거나 압축하려면

다음 중 하나를 사용하세요:

- **Compact** (대화를 유지하되 이전 턴을 요약):

  ```
  /compact
  ```

  또는 요약을 안내하기 위해 `/compact <instructions>`.

- **Reset** (같은 채팅 키에 대한 새 세션 ID):

  ```
  /new
  /reset
  ```

계속 발생하면:

- **세션 프루닝** (`agents.defaults.contextPruning`)을 활성화하거나 조정하여 오래된 도구 출력을 제거하세요.
- 더 큰 컨텍스트 윈도우를 가진 모델을 사용하세요.

문서: [압축](/concepts/compaction), [세션 프루닝](/concepts/session-pruning), [세션 관리](/concepts/session).

### "LLM request rejected: messages.content.tool_use.input field required"가 보이는 이유는?

이것은 프로바이더 검증 오류입니다: 모델이 필수 `input`이 없는 `tool_use` 블록을 내보냈습니다. 보통 세션 기록이 오래되었거나 손상되었음을 의미합니다 (종종 긴 스레드나 도구/스키마 변경 후).

수정: 독립 메시지로 `/new`를 보내 새 세션을 시작하세요.

### 왜 30분마다 하트비트 메시지를 받나요

하트비트는 기본적으로 **30분**마다 실행됩니다. 조정하거나 비활성화하세요:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "2h", // 또는 비활성화하려면 "0m"
      },
    },
  },
}
```

`HEARTBEAT.md`가 존재하지만 사실상 비어 있으면 (빈 줄과 `# Heading` 같은 마크다운 헤더만), OpenClaw는 API 호출을 절약하기 위해 하트비트 실행을 건너뜁니다.
파일이 없으면 하트비트는 여전히 실행되고 모델이 무엇을 할지 결정합니다.

에이전트별 오버라이드는 `agents.list[].heartbeat`를 사용합니다. 문서: [하트비트](/gateway/heartbeat).

### WhatsApp 그룹에 봇 계정을 추가해야 하나요

아닙니다. OpenClaw는 **자신의 계정**에서 실행되므로, 그룹에 있으면 OpenClaw가 볼 수 있습니다. 기본적으로 그룹 응답은 보내는 사람을 허용할 때까지 차단됩니다 (`groupPolicy: "allowlist"`).

**당신**만 그룹 응답을 트리거할 수 있게 하려면:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

### WhatsApp 그룹의 JID를 어떻게 얻나요

옵션 1 (가장 빠름): 로그를 tail하고 그룹에 테스트 메시지를 보내세요:

```bash
openclaw logs --follow --json
```

`@g.us`로 끝나는 `chatId` (또는 `from`)를 찾으세요:
`1234567890-1234567890@g.us`.

옵션 2 (이미 구성/허용된 경우): 구성에서 그룹을 나열:

```bash
openclaw directory groups list --channel whatsapp
```

문서: [WhatsApp](/channels/whatsapp), [Directory](/cli/directory), [Logs](/cli/logs).

### 왜 OpenClaw가 그룹에서 응답하지 않나요

두 가지 일반적인 원인:

- 멘션 게이팅이 켜져 있음 (기본값). 봇을 @멘션하거나 (`mentionPatterns`와 일치해야) 합니다.
- `"*"` 없이 `channels.whatsapp.groups`를 구성했고 그룹이 허용 목록에 없습니다.

[그룹](/channels/groups) 및 [그룹 메시지](/channels/group-messages)를 참조하세요.

### 그룹/스레드가 DM과 컨텍스트를 공유하나요

직접 채팅은 기본적으로 메인 세션으로 축소됩니다. 그룹/채널은 자체 세션 키를 가지며, Telegram 토픽 / Discord 스레드는 별도의 세션입니다. [그룹](/channels/groups) 및 [그룹 메시지](/channels/group-messages)를 참조하세요.

### 워크스페이스와 에이전트를 몇 개까지 만들 수 있나요

하드 제한 없음. 수십 개(심지어 수백 개)도 괜찮지만 주의할 점:

- **디스크 증가:** 세션 + 대화록이 `~/.openclaw/agents/<agentId>/sessions/` 아래에 있습니다.
- **토큰 비용:** 더 많은 에이전트는 더 많은 동시 모델 사용을 의미합니다.
- **운영 오버헤드:** 에이전트별 인증 프로필, 워크스페이스, 채널 라우팅.

팁:

- 에이전트당 하나의 **활성** 워크스페이스를 유지하세요 (`agents.defaults.workspace`).
- 디스크가 증가하면 오래된 세션을 정리하세요 (JSONL 또는 저장소 항목 삭제).
- `openclaw doctor`를 사용하여 누락된 워크스페이스와 프로필 불일치를 발견하세요.

### 동시에 여러 봇이나 채팅을 실행할 수 있나요 (Slack) 어떻게 설정하나요

네. **멀티 에이전트 라우팅**을 사용하여 여러 격리된 에이전트를 실행하고 채널/계정/피어별로 인바운드 메시지를 라우팅하세요. Slack은 채널로 지원되며 특정 에이전트에 바인딩할 수 있습니다.

브라우저 접근은 강력하지만 "사람이 할 수 있는 모든 것"은 아닙니다 - 안티봇, CAPTCHA, MFA가 여전히 자동화를 차단할 수 있습니다. 가장 안정적인 브라우저 제어를 위해 브라우저를 실행하는 머신에서 Chrome 확장 릴레이를 사용하세요 (Gateway는 어디에든 가능).

모범 사례 설정:

- 항상 켜진 Gateway 호스트 (VPS/Mac mini).
- 역할별 하나의 에이전트 (바인딩).
- 해당 에이전트에 바인딩된 Slack 채널.
- 필요할 때 확장 릴레이(또는 노드)를 통한 로컬 브라우저.

문서: [멀티 에이전트 라우팅](/concepts/multi-agent), [Slack](/channels/slack),
[브라우저](/tools/browser), [Chrome 확장](/tools/chrome-extension), [노드](/nodes).

## 모델: 기본값, 선택, 별칭, 전환

### 기본 모델이란 무엇인가요

OpenClaw의 기본 모델은 다음으로 설정한 것입니다:

```
agents.defaults.model.primary
```

모델은 `provider/model`로 참조됩니다 (예: `anthropic/claude-opus-4-6`). 프로바이더를 생략하면 OpenClaw는 현재 임시 사용 중단 대체로 `anthropic`을 가정합니다 - 하지만 **명시적으로** `provider/model`을 설정해야 합니다.

### 어떤 모델을 추천하나요

**권장 기본값:** `anthropic/claude-opus-4-6`.
**좋은 대안:** `anthropic/claude-sonnet-4-5`.
**안정적 (캐릭터 약함):** `openai/gpt-5.2` - Opus만큼 좋지만 개성이 적음.
**저예산:** `zai/glm-4.7`.

MiniMax M2.1은 자체 문서가 있습니다: [MiniMax](/providers/minimax) 및
[로컬 모델](/gateway/local-models).

경험 법칙: 고위험 작업에는 **감당할 수 있는 가장 좋은 모델**을 사용하고, 일상적인 채팅이나 요약에는 더 저렴한 모델을 사용하세요. 에이전트별로 모델을 라우팅하고 서브 에이전트를 사용하여 긴 작업을 병렬화할 수 있습니다 (각 서브 에이전트는 토큰을 소비). [모델](/concepts/models) 및 [서브 에이전트](/tools/subagents)를 참조하세요.

강한 경고: 약하거나 과도하게 양자화된 모델은 프롬프트 인젝션과 안전하지 않은 동작에 더 취약합니다. [보안](/gateway/security)을 참조하세요.

추가 컨텍스트: [모델](/concepts/models).

### 자체 호스팅 모델(llama.cpp, vLLM, Ollama)을 사용할 수 있나요

네. 로컬 서버가 OpenAI 호환 API를 노출하면 커스텀 프로바이더로 가리킬 수 있습니다. Ollama는 직접 지원되며 가장 쉬운 경로입니다.

보안 참고: 더 작거나 심하게 양자화된 모델은 프롬프트 인젝션에 더 취약합니다. 도구를 사용할 수 있는 봇에는 **큰 모델**을 강력히 권장합니다. 여전히 작은 모델을 원하면 샌드박싱과 엄격한 도구 허용 목록을 활성화하세요.

문서: [Ollama](/providers/ollama), [로컬 모델](/gateway/local-models),
[모델 프로바이더](/concepts/model-providers), [보안](/gateway/security),
[샌드박싱](/gateway/sandboxing).

### 구성을 지우지 않고 모델을 전환하려면

**모델 명령**을 사용하거나 **모델** 필드만 편집하세요. 전체 구성 교체를 피하세요.

안전한 옵션:

- 채팅에서 `/model` (빠른, 세션별)
- `openclaw models set ...` (모델 구성만 업데이트)
- `openclaw configure --section model` (대화형)
- `~/.openclaw/openclaw.json`에서 `agents.defaults.model` 편집

부분 객체로 전체 구성을 교체할 의도가 아니면 `config.apply`를 피하세요.
구성을 덮어썼다면 백업에서 복원하거나 `openclaw doctor`를 실행하여 수리하세요.

문서: [모델](/concepts/models), [Configure](/cli/configure), [Config](/cli/config), [Doctor](/gateway/doctor).

### OpenClaw, Flawd, Krill은 어떤 모델을 사용하나요

- **OpenClaw + Flawd:** Anthropic Opus (`anthropic/claude-opus-4-6`) - [Anthropic](/providers/anthropic) 참조.
- **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - [MiniMax](/providers/minimax) 참조.

### 재시작 없이 즉시 모델을 전환하려면

독립 메시지로 `/model` 명령을 사용하세요:

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

`/model`, `/model list`, 또는 `/model status`로 사용 가능한 모델을 나열할 수 있습니다.

`/model` (및 `/model list`)은 컴팩트한 번호 선택기를 보여줍니다. 번호로 선택:

```
/model 3
```

세션별로 프로바이더의 특정 인증 프로필을 강제할 수도 있습니다:

```
/model opus@anthropic:default
/model opus@anthropic:work
```

팁: `/model status`는 어떤 에이전트가 활성인지, 어떤 `auth-profiles.json` 파일이 사용되는지, 다음에 시도될 인증 프로필을 보여줍니다.
사용 가능한 경우 구성된 프로바이더 엔드포인트(`baseUrl`)와 API 모드(`api`)도 표시합니다.

**@profile로 설정한 프로필을 어떻게 해제하나요**

`@profile` 접미사 **없이** `/model`을 다시 실행하세요:

```
/model anthropic/claude-opus-4-6
```

기본값으로 돌아가려면 `/model`에서 선택하세요 (또는 `/model <default provider/model>`을 보내세요).
`/model status`를 사용하여 어떤 인증 프로필이 활성인지 확인하세요.

### 일상 작업에 GPT 5.2를 코딩에 Codex 5.3을 사용할 수 있나요

네. 하나를 기본값으로 설정하고 필요에 따라 전환하세요:

- **빠른 전환 (세션별):** 일상 작업에 `/model gpt-5.2`, 코딩에 `/model gpt-5.3-codex`.
- **기본값 + 전환:** `agents.defaults.model.primary`를 `openai/gpt-5.2`로 설정한 다음 코딩 시 `openai-codex/gpt-5.3-codex`로 전환 (또는 반대).
- **서브 에이전트:** 다른 기본 모델을 가진 서브 에이전트에 코딩 작업을 라우팅.

[모델](/concepts/models) 및 [슬래시 명령](/tools/slash-commands)을 참조하세요.

### "Model is not allowed"가 나타나고 응답이 없는 이유는

`agents.defaults.models`가 설정되면 `/model` 및 모든 세션 오버라이드에 대한 **허용 목록**이 됩니다. 해당 목록에 없는 모델을 선택하면 다음이 반환됩니다:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

해당 오류는 일반 응답 **대신** 반환됩니다. 수정: 모델을 `agents.defaults.models`에 추가하거나, 허용 목록을 제거하거나, `/model list`에서 모델을 선택하세요.

### "Unknown model: minimax/MiniMax-M2.1"이 보이는 이유는

이것은 **프로바이더가 구성되지 않았음**을 의미합니다 (MiniMax 프로바이더 구성이나 인증 프로필을 찾을 수 없음), 따라서 모델을 해결할 수 없습니다. 이 감지에 대한 수정이 **2026.1.12**에 있습니다 (작성 시점에 미출시).

수정 체크리스트:

1. **2026.1.12**로 업그레이드 (또는 소스 `main`에서 실행), 그런 다음 게이트웨이를 재시작합니다.
2. MiniMax가 구성되어 있는지 (마법사 또는 JSON), 또는 MiniMax API 키가 환경/인증 프로필에 있어 프로바이더가 주입될 수 있는지 확인합니다.
3. 정확한 모델 id (대소문자 구분)를 사용: `minimax/MiniMax-M2.1` 또는
   `minimax/MiniMax-M2.1-lightning`.
4. 실행:

   ```bash
   openclaw models list
   ```

   목록에서 선택하세요 (또는 채팅에서 `/model list`).

[MiniMax](/providers/minimax) 및 [모델](/concepts/models)을 참조하세요.

### MiniMax를 기본으로 복잡한 작업에는 OpenAI를 사용할 수 있나요

네. **MiniMax를 기본값**으로 사용하고 필요할 때 **세션별로** 모델을 전환하세요. 장애 조치는 "어려운 작업"이 아닌 **오류**를 위한 것이므로 `/model` 또는 별도의 에이전트를 사용하세요.

**옵션 A: 세션별 전환**

```json5
{
  env: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "minimax" },
        "openai/gpt-5.2": { alias: "gpt" },
      },
    },
  },
}
```

그런 다음:

```
/model gpt
```

**옵션 B: 별도의 에이전트**

- Agent A 기본값: MiniMax
- Agent B 기본값: OpenAI
- 에이전트별로 라우팅하거나 `/agent`를 사용하여 전환

문서: [모델](/concepts/models), [멀티 에이전트 라우팅](/concepts/multi-agent), [MiniMax](/providers/minimax), [OpenAI](/providers/openai).

### opus sonnet gpt는 내장 단축키인가요

네. OpenClaw는 몇 가지 기본 약칭을 제공합니다 (모델이 `agents.defaults.models`에 존재하는 경우에만 적용):

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

같은 이름으로 자체 별칭을 설정하면 해당 값이 우선합니다.

### 모델 단축키(별칭)를 정의/재정의하려면

별칭은 `agents.defaults.models.<modelId>.alias`에서 옵니다. 예제:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" },
      },
    },
  },
}
```

그런 다음 `/model sonnet` (또는 지원되는 경우 `/<alias>`)이 해당 모델 ID로 해결됩니다.

### OpenRouter나 Z.AI 같은 다른 프로바이더의 모델을 추가하려면

OpenRouter (토큰당 과금; 많은 모델):

```json5
{
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
      models: { "openrouter/anthropic/claude-sonnet-4-5": {} },
    },
  },
  env: { OPENROUTER_API_KEY: "sk-or-..." },
}
```

Z.AI (GLM 모델):

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
  env: { ZAI_API_KEY: "..." },
}
```

프로바이더/모델을 참조하지만 필수 프로바이더 키가 누락되면 런타임 인증 오류가 발생합니다 (예: `No API key found for provider "zai"`).

**새 에이전트 추가 후 프로바이더의 API 키를 찾을 수 없음**

이것은 보통 **새 에이전트**에 빈 인증 저장소가 있음을 의미합니다. 인증은 에이전트별이며 다음에 저장됩니다:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

수정 옵션:

- `openclaw agents add <id>`를 실행하고 마법사 중에 인증을 구성합니다.
- 또는 메인 에이전트의 `agentDir`에서 새 에이전트의 `agentDir`로 `auth-profiles.json`을 복사합니다.

에이전트 간에 `agentDir`을 재사용하지 **마세요**; 인증/세션 충돌이 발생합니다.

## 모델 장애 조치 및 "All models failed"

### 장애 조치는 어떻게 작동하나요

장애 조치는 두 단계로 발생합니다:

1. 같은 프로바이더 내에서 **인증 프로필 로테이션**.
2. `agents.defaults.model.fallbacks`의 다음 모델로 **모델 대체**.

실패하는 프로필에 쿨다운(지수 백오프)이 적용되므로, 프로바이더가 속도 제한되거나 일시적으로 실패해도 OpenClaw가 계속 응답할 수 있습니다.

### 이 오류는 무엇을 의미하나요

```
No credentials found for profile "anthropic:default"
```

시스템이 인증 프로필 ID `anthropic:default`를 사용하려 했지만, 예상 인증 저장소에서 자격 증명을 찾을 수 없음을 의미합니다.

### No credentials found for profile anthropic:default 수정 체크리스트

- **인증 프로필이 있는 위치 확인** (새 vs 레거시 경로)
  - 현재: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
  - 레거시: `~/.openclaw/agent/*` (`openclaw doctor`로 마이그레이션)
- **환경 변수가 Gateway에 로드되는지 확인**
  - 셸에서 `ANTHROPIC_API_KEY`를 설정했지만 systemd/launchd를 통해 Gateway를 실행하면 상속하지 않을 수 있습니다. `~/.openclaw/.env`에 넣거나 `env.shellEnv`를 활성화하세요.
- **올바른 에이전트를 편집하고 있는지 확인**
  - 멀티 에이전트 설정은 여러 `auth-profiles.json` 파일이 있을 수 있습니다.
- **모델/인증 상태 검사**
  - `openclaw models status`를 사용하여 구성된 모델과 프로바이더가 인증되었는지 확인하세요.

**No credentials found for profile anthropic 수정 체크리스트**

실행이 Anthropic 인증 프로필에 고정되어 있지만, Gateway가 인증 저장소에서 찾을 수 없음을 의미합니다.

- **setup-token 사용**
  - `claude setup-token`을 실행한 다음 `openclaw models auth setup-token --provider anthropic`으로 붙여넣으세요.
  - 다른 머신에서 토큰을 만들었다면 `openclaw models auth paste-token --provider anthropic`을 사용하세요.
- **API 키를 대신 사용하려면**
  - **게이트웨이 호스트**의 `~/.openclaw/.env`에 `ANTHROPIC_API_KEY`를 넣으세요.
  - 누락된 프로필을 강제하는 고정 순서를 해제하세요:

    ```bash
    openclaw models auth order clear --provider anthropic
    ```

- **게이트웨이 호스트에서 명령을 실행하고 있는지 확인**
  - 원격 모드에서 인증 프로필은 노트북이 아닌 게이트웨이 머신에 있습니다.

### 왜 Google Gemini도 시도하고 실패했나요

모델 구성에 Google Gemini가 대체로 포함되어 있거나 (또는 Gemini 약칭으로 전환한 경우) OpenClaw가 모델 대체 중에 시도합니다. Google 자격 증명을 구성하지 않았다면 `No API key found for provider "google"`이 표시됩니다.

수정: Google 인증을 제공하거나, 대체가 거기로 라우팅되지 않도록 `agents.defaults.model.fallbacks` / 별칭에서 Google 모델을 제거/회피하세요.

**LLM request rejected message thinking signature required google antigravity**

원인: 세션 기록에 **서명 없는 thinking 블록**이 포함되어 있습니다 (종종 중단된/부분 스트림에서). Google Antigravity는 thinking 블록에 서명을 요구합니다.

수정: OpenClaw는 이제 Google Antigravity Claude에 대해 서명되지 않은 thinking 블록을 제거합니다. 여전히 나타나면 **새 세션**을 시작하거나 해당 에이전트에 대해 `/thinking off`를 설정하세요.

## 인증 프로필: 정의와 관리 방법

관련: [/concepts/oauth](/concepts/oauth) (OAuth 흐름, 토큰 저장, 멀티 계정 패턴)

### 인증 프로필이란 무엇인가요

인증 프로필은 프로바이더에 연결된 명명된 자격 증명 레코드(OAuth 또는 API 키)입니다. 프로필은 다음에 있습니다:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

### 일반적인 프로필 ID는 무엇인가요

OpenClaw는 프로바이더 접두사가 있는 ID를 사용합니다:

- `anthropic:default` (이메일 ID가 없을 때 일반적)
- `anthropic:<email>` OAuth ID
- 직접 선택한 커스텀 ID (예: `anthropic:work`)

### 어떤 인증 프로필을 먼저 시도할지 제어할 수 있나요

네. 구성은 프로필에 대한 선택적 메타데이터와 프로바이더별 순서(`auth.order.<provider>`)를 지원합니다. 이것은 비밀을 저장하지 **않습니다**; ID를 프로바이더/모드에 매핑하고 로테이션 순서를 설정합니다.

OpenClaw는 프로필이 짧은 **쿨다운** (속도 제한/타임아웃/인증 실패) 또는 더 긴 **비활성화** 상태(결제/잔액 부족)에 있으면 일시적으로 건너뛸 수 있습니다. 이를 검사하려면 `openclaw models status --json`을 실행하고 `auth.unusableProfiles`를 확인하세요. 조정: `auth.cooldowns.billingBackoffHours*`.

CLI를 통해 **에이전트별** 순서 오버라이드도 설정할 수 있습니다 (해당 에이전트의 `auth-profiles.json`에 저장):

```bash
# 구성된 기본 에이전트에 적용 (--agent 생략)
openclaw models auth order get --provider anthropic

# 단일 프로필로 로테이션 잠금 (이것만 시도)
openclaw models auth order set --provider anthropic anthropic:default

# 명시적 순서 설정 (프로바이더 내 대체)
openclaw models auth order set --provider anthropic anthropic:work anthropic:default

# 오버라이드 해제 (구성 auth.order / 라운드 로빈으로 대체)
openclaw models auth order clear --provider anthropic
```

특정 에이전트를 대상으로:

```bash
openclaw models auth order set --provider anthropic --agent main anthropic:default
```

### OAuth vs API 키 차이점은

OpenClaw는 둘 다 지원합니다:

- **OAuth**는 종종 구독 접근을 활용합니다 (해당되는 경우).
- **API 키**는 토큰당 과금 결제를 사용합니다.

마법사는 Anthropic setup-token과 OpenAI Codex OAuth를 명시적으로 지원하며 API 키를 저장할 수 있습니다.

## Gateway: 포트, "already running", 원격 모드

### Gateway는 어떤 포트를 사용하나요

`gateway.port`는 WebSocket + HTTP (Control UI, 훅 등)에 대한 단일 다중화 포트를 제어합니다.

우선순위:

```
--port > OPENCLAW_GATEWAY_PORT > gateway.port > 기본값 18789
```

### openclaw gateway status가 Runtime running이지만 RPC probe failed인 이유는

"running"은 **슈퍼바이저의** 뷰(launchd/systemd/schtasks)이기 때문입니다. RPC 프로브는 CLI가 실제로 게이트웨이 WebSocket에 연결하고 `status`를 호출하는 것입니다.

`openclaw gateway status`를 사용하고 다음 줄을 신뢰하세요:

- `Probe target:` (프로브가 실제로 사용한 URL)
- `Listening:` (포트에 실제로 바인딩된 것)
- `Last gateway error:` (프로세스가 살아있지만 포트가 리슨하지 않을 때 일반적인 근본 원인)

### openclaw gateway status에서 Config (cli)와 Config (service)가 다른 이유는

서비스가 다른 구성 파일을 실행하는 동안 하나의 구성 파일을 편집하고 있습니다 (종종 `--profile` / `OPENCLAW_STATE_DIR` 불일치).

수정:

```bash
openclaw gateway install --force
```

서비스가 사용하기를 원하는 것과 같은 `--profile` / 환경에서 이를 실행하세요.

### "another gateway instance is already listening"은 무슨 의미인가요

OpenClaw는 시작 시 WebSocket 리스너를 즉시 바인딩하여 런타임 잠금을 시행합니다 (기본값 `ws://127.0.0.1:18789`). 바인드가 `EADDRINUSE`로 실패하면 다른 인스턴스가 이미 리슨 중임을 나타내는 `GatewayLockError`를 발생시킵니다.

수정: 다른 인스턴스를 중지하거나, 포트를 해제하거나, `openclaw gateway --port <port>`로 실행하세요.

### 원격 모드로 OpenClaw를 실행하려면 (클라이언트가 다른 곳의 Gateway에 연결)

`gateway.mode: "remote"`를 설정하고 원격 WebSocket URL을 가리키세요. 선택적으로 토큰/비밀번호 포함:

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

참고:

- `openclaw gateway`는 `gateway.mode`가 `local`일 때만 시작됩니다 (또는 오버라이드 플래그를 전달).
- macOS 앱은 구성 파일을 감시하고 이 값이 변경되면 모드를 라이브로 전환합니다.

### Control UI가 unauthorized라고 하거나 계속 재연결합니다 어떻게 하나요

게이트웨이가 인증 활성화(`gateway.auth.*`)로 실행되고 있지만, UI가 일치하는 토큰/비밀번호를 보내지 않습니다.

사실 (코드에서):

- Control UI는 토큰을 브라우저 localStorage 키 `openclaw.control.settings.v1`에 저장합니다.

수정:

- 가장 빠름: `openclaw dashboard` (대시보드 URL을 출력 + 복사하고 열려고 시도; headless이면 SSH 힌트를 표시).
- 아직 토큰이 없으면: `openclaw doctor --generate-gateway-token`.
- 원격이면 먼저 터널: `ssh -N -L 18789:127.0.0.1:18789 user@host` 그런 다음 `http://127.0.0.1:18789/`를 엽니다.
- 게이트웨이 호스트에서 `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`)을 설정합니다.
- Control UI 설정에서 같은 토큰을 붙여넣습니다.
- 여전히 문제가 있으면? `openclaw status --all`을 실행하고 [문제 해결](/gateway/troubleshooting)을 따르세요. 인증 세부 사항은 [대시보드](/web/dashboard)를 참조하세요.

### gateway.bind를 tailnet으로 설정했는데 바인드할 수 없음 아무것도 리슨하지 않음

`tailnet` 바인드는 네트워크 인터페이스에서 Tailscale IP를 선택합니다 (100.64.0.0/10). 머신이 Tailscale에 없거나 (또는 인터페이스가 다운되면) 바인딩할 것이 없습니다.

수정:

- 해당 호스트에서 Tailscale을 시작하거나 (100.x 주소가 생기도록), 또는
- `gateway.bind: "loopback"` / `"lan"`으로 전환하세요.

참고: `tailnet`은 명시적입니다. `auto`는 루프백을 선호합니다; tailnet 전용 바인드를 원할 때 `gateway.bind: "tailnet"`을 사용하세요.

### 같은 호스트에서 여러 Gateway를 실행할 수 있나요

보통은 아닙니다 - 하나의 Gateway가 여러 메시징 채널과 에이전트를 실행할 수 있습니다. 중복성(예: 구조 봇)이나 하드 격리가 필요할 때만 여러 Gateway를 사용하세요.

네, 하지만 다음을 격리해야 합니다:

- `OPENCLAW_CONFIG_PATH` (인스턴스별 구성)
- `OPENCLAW_STATE_DIR` (인스턴스별 상태)
- `agents.defaults.workspace` (워크스페이스 격리)
- `gateway.port` (고유 포트)

빠른 설정 (권장):

- 인스턴스별로 `openclaw --profile <name> ...`을 사용하세요 (`~/.openclaw-<name>`을 자동 생성).
- 각 프로필 구성에서 고유한 `gateway.port`를 설정하세요 (또는 수동 실행 시 `--port` 전달).
- 프로필별 서비스 설치: `openclaw --profile <name> gateway install`.

프로필은 서비스 이름에도 접미사를 붙입니다 (`bot.molt.<profile>`; 레거시 `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Gateway (<profile>)`).
전체 가이드: [여러 게이트웨이](/gateway/multiple-gateways).

### "invalid handshake" / code 1008은 무슨 의미인가요

Gateway는 **WebSocket 서버**이며, 첫 번째 메시지가 `connect` 프레임이어야 합니다. 다른 것을 받으면 **code 1008** (정책 위반)로 연결을 닫습니다.

일반적인 원인:

- WS 클라이언트 대신 브라우저에서 **HTTP** URL을 열었습니다 (`http://...`).
- 잘못된 포트나 경로를 사용했습니다.
- 프록시나 터널이 인증 헤더를 제거하거나 비Gateway 요청을 보냈습니다.

빠른 수정:

1. WS URL을 사용하세요: `ws://<host>:18789` (또는 HTTPS이면 `wss://...`).
2. 일반 브라우저 탭에서 WS 포트를 열지 마세요.
3. 인증이 켜져 있으면 `connect` 프레임에 토큰/비밀번호를 포함하세요.

CLI나 TUI를 사용하는 경우 URL은 다음과 같아야 합니다:

```
openclaw tui --url ws://<host>:18789 --token <token>
```

프로토콜 세부 사항: [Gateway 프로토콜](/gateway/protocol).

## 로깅 및 디버깅

### 로그는 어디에 있나요

파일 로그 (구조화):

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

`logging.file`을 통해 안정적인 경로를 설정할 수 있습니다. 파일 로그 레벨은 `logging.level`로 제어됩니다. 콘솔 상세도는 `--verbose`와 `logging.consoleLevel`로 제어됩니다.

가장 빠른 로그 tail:

```bash
openclaw logs --follow
```

서비스/슈퍼바이저 로그 (게이트웨이가 launchd/systemd를 통해 실행될 때):

- macOS: `$OPENCLAW_STATE_DIR/logs/gateway.log` 및 `gateway.err.log` (기본값: `~/.openclaw/logs/...`; 프로필은 `~/.openclaw-<profile>/logs/...` 사용)
- Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Windows: `schtasks /Query /TN "OpenClaw Gateway (<profile>)" /V /FO LIST`

자세한 내용은 [문제 해결](/gateway/troubleshooting#log-locations)을 참조하세요.

### Gateway 서비스를 시작/중지/재시작하려면

게이트웨이 헬퍼를 사용하세요:

```bash
openclaw gateway status
openclaw gateway restart
```

게이트웨이를 수동으로 실행하면 `openclaw gateway --force`로 포트를 회수할 수 있습니다. [Gateway](/gateway)를 참조하세요.

### Windows에서 터미널을 닫았습니다 OpenClaw를 재시작하려면

두 가지 **Windows 설치 모드**가 있습니다:

**1) WSL2 (권장):** Gateway가 Linux 내에서 실행됩니다.

PowerShell을 열고, WSL에 진입한 다음 재시작:

```powershell
wsl
openclaw gateway status
openclaw gateway restart
```

서비스를 설치하지 않았으면 포그라운드에서 시작:

```bash
openclaw gateway run
```

**2) 네이티브 Windows (권장하지 않음):** Gateway가 Windows에서 직접 실행됩니다.

PowerShell을 열고 실행:

```powershell
openclaw gateway status
openclaw gateway restart
```

수동으로 실행하면 (서비스 없음):

```powershell
openclaw gateway run
```

문서: [Windows (WSL2)](/platforms/windows), [Gateway 서비스 런북](/gateway).

### Gateway가 실행 중인데 응답이 도착하지 않습니다 무엇을 확인해야 하나요

빠른 상태 스윕으로 시작:

```bash
openclaw status
openclaw models status
openclaw channels status
openclaw logs --follow
```

일반적인 원인:

- **게이트웨이 호스트**에서 모델 인증이 로드되지 않음 (`models status` 확인).
- 채널 페어링/허용 목록이 응답을 차단 (채널 구성 + 로그 확인).
- WebChat/대시보드가 올바른 토큰 없이 열려 있음.

원격이면 터널/Tailscale 연결이 살아있고 Gateway WebSocket에 접근 가능한지 확인하세요.

문서: [채널](/channels), [문제 해결](/gateway/troubleshooting), [원격 접근](/gateway/remote).

### "Disconnected from gateway: no reason" 어떻게 하나요

이것은 보통 UI가 WebSocket 연결을 잃었음을 의미합니다. 확인:

1. Gateway가 실행 중인가요? `openclaw gateway status`
2. Gateway가 건강한가요? `openclaw status`
3. UI에 올바른 토큰이 있나요? `openclaw dashboard`
4. 원격이면 터널/Tailscale 링크가 살아있나요?

그런 다음 로그 tail:

```bash
openclaw logs --follow
```

문서: [대시보드](/web/dashboard), [원격 접근](/gateway/remote), [문제 해결](/gateway/troubleshooting).

### Telegram setMyCommands가 네트워크 오류와 함께 실패합니다 무엇을 확인해야 하나요

로그와 채널 상태로 시작:

```bash
openclaw channels status
openclaw channels logs --channel telegram
```

VPS에 있거나 프록시 뒤에 있으면 아웃바운드 HTTPS가 허용되고 DNS가 작동하는지 확인하세요.
Gateway가 원격이면 게이트웨이 호스트에서 로그를 보고 있는지 확인하세요.

문서: [Telegram](/channels/telegram), [채널 문제 해결](/channels/troubleshooting).

### TUI에 출력이 표시되지 않습니다 무엇을 확인해야 하나요

먼저 Gateway가 접근 가능하고 에이전트가 실행될 수 있는지 확인:

```bash
openclaw status
openclaw models status
openclaw logs --follow
```

TUI에서 `/status`를 사용하여 현재 상태를 확인하세요. 채팅 채널에서 응답을 기대하면 delivery가 활성화되어 있는지 확인하세요 (`/deliver on`).

문서: [TUI](/web/tui), [슬래시 명령](/tools/slash-commands).

### Gateway를 완전히 중지한 후 시작하려면

서비스를 설치한 경우:

```bash
openclaw gateway stop
openclaw gateway start
```

이렇게 하면 **감독 서비스**를 중지/시작합니다 (macOS에서 launchd, Linux에서 systemd).
Gateway가 백그라운드에서 데몬으로 실행될 때 사용하세요.

포그라운드에서 실행 중이면 Ctrl-C로 중지한 다음:

```bash
openclaw gateway run
```

문서: [Gateway 서비스 런북](/gateway).

### ELI5 openclaw gateway restart vs openclaw gateway

- `openclaw gateway restart`: **백그라운드 서비스**를 재시작합니다 (launchd/systemd).
- `openclaw gateway`: 이 터미널 세션에서 게이트웨이를 **포그라운드에서** 실행합니다.

서비스를 설치했으면 gateway 명령을 사용하세요. 일회성 포그라운드 실행을 원할 때 `openclaw gateway`를 사용하세요.

### 무언가 실패할 때 더 많은 세부 정보를 얻는 가장 빠른 방법은

`--verbose`로 Gateway를 시작하면 더 많은 콘솔 세부 정보가 표시됩니다. 그런 다음 채널 인증, 모델 라우팅, RPC 오류에 대해 로그 파일을 검사하세요.

## 미디어 및 첨부 파일

### 스킬이 이미지/PDF를 생성했는데 아무것도 전송되지 않았습니다

에이전트의 아웃바운드 첨부 파일에는 `MEDIA:<path-or-url>` 줄이 포함되어야 합니다 (자체 줄에). [OpenClaw 어시스턴트 설정](/start/openclaw) 및 [Agent send](/tools/agent-send)를 참조하세요.

CLI 전송:

```bash
openclaw message send --target +15555550123 --message "Here you go" --media /path/to/file.png
```

또한 확인:

- 대상 채널이 아웃바운드 미디어를 지원하고 허용 목록에 의해 차단되지 않는지.
- 파일이 프로바이더의 크기 제한 이내인지 (이미지는 최대 2048px로 리사이즈).

[이미지](/nodes/images)를 참조하세요.

## 보안 및 접근 제어

### 인바운드 DM에 OpenClaw를 노출하는 것이 안전한가요

인바운드 DM을 신뢰할 수 없는 입력으로 취급하세요. 기본값은 위험을 줄이도록 설계되었습니다:

- DM 가능 채널의 기본 동작은 **페어링**입니다:
  - 알 수 없는 보내는 사람은 페어링 코드를 받습니다; 봇은 해당 메시지를 처리하지 않습니다.
  - `openclaw pairing approve <channel> <code>`로 승인
  - 대기 중인 요청은 **채널당 3개**로 제한됩니다; 코드가 도착하지 않으면 `openclaw pairing list <channel>`을 확인하세요.
- DM을 공개적으로 여는 것은 명시적 opt-in이 필요합니다 (`dmPolicy: "open"` 및 허용 목록 `"*"`).

위험한 DM 정책을 확인하려면 `openclaw doctor`를 실행하세요.

### 프롬프트 인젝션은 공개 봇에만 해당되는 문제인가요

아닙니다. 프롬프트 인젝션은 봇에게 DM을 보낼 수 있는 사람이 아니라 **신뢰할 수 없는 콘텐츠**에 관한 것입니다. 어시스턴트가 외부 콘텐츠(웹 검색/fetch, 브라우저 페이지, 이메일, 문서, 첨부 파일, 붙여넣은 로그)를 읽으면, 해당 콘텐츠에 모델을 하이재킹하려는 지시가 포함될 수 있습니다. 이것은 **당신만 보내는 사람**일 때도 발생할 수 있습니다.

가장 큰 위험은 도구가 활성화될 때입니다: 모델이 컨텍스트를 유출하거나 당신을 대신하여 도구를 호출하도록 속을 수 있습니다. 폭발 반경을 줄이려면:

- 읽기 전용 또는 도구 비활성화된 "리더" 에이전트를 사용하여 신뢰할 수 없는 콘텐츠를 요약
- 도구 활성 에이전트에서 `web_search` / `web_fetch` / `browser`를 비활성화
- 샌드박싱과 엄격한 도구 허용 목록

자세한 내용: [보안](/gateway/security).

### 봇에게 별도의 이메일 GitHub 계정이나 전화번호가 있어야 하나요

네, 대부분의 설정에서. 별도의 계정과 전화번호로 봇을 격리하면 문제가 발생했을 때 폭발 반경이 줄어듭니다. 이렇게 하면 개인 계정에 영향을 주지 않고 자격 증명을 교체하거나 접근을 취소하기도 더 쉽습니다.

작게 시작하세요. 실제로 필요한 도구와 계정에만 접근 권한을 부여하고, 나중에 필요하면 확장하세요.

문서: [보안](/gateway/security), [페어링](/channels/pairing).

### 텍스트 메시지에 대한 자율권을 줄 수 있고 안전한가요

개인 메시지에 대한 완전한 자율권은 **권장하지 않습니다**. 가장 안전한 패턴:

- DM을 **페어링 모드**나 엄격한 허용 목록으로 유지.
- 대리로 메시지를 보내려면 **별도의 번호나 계정**을 사용.
- 초안을 작성하게 하고 **보내기 전에 승인**.

실험하려면 전용 계정에서 하고 격리를 유지하세요. [보안](/gateway/security)을 참조하세요.

### 개인 어시스턴트 작업에 저렴한 모델을 사용할 수 있나요

네, 에이전트가 채팅 전용이고 입력이 신뢰할 수 있는 **경우**. 더 작은 티어는 지시 하이재킹에 더 취약하므로, 도구 활성 에이전트나 신뢰할 수 없는 콘텐츠를 읽을 때는 피하세요. 여전히 더 작은 모델을 사용해야 하면 도구를 잠그고 샌드박스 내에서 실행하세요. [보안](/gateway/security)을 참조하세요.

### Telegram에서 /start를 실행했는데 페어링 코드를 받지 못했습니다

페어링 코드는 알 수 없는 보내는 사람이 봇에게 메시지를 보내고 `dmPolicy: "pairing"`이 활성화된 경우**에만** 전송됩니다. `/start` 자체로는 코드를 생성하지 않습니다.

대기 중인 요청 확인:

```bash
openclaw pairing list telegram
```

즉시 접근을 원하면 보내는 사람 id를 허용 목록에 추가하거나 해당 계정에 `dmPolicy: "open"`을 설정하세요.

### WhatsApp: 내 연락처에 메시지를 보내나요 페어링은 어떻게 작동하나요

아닙니다. 기본 WhatsApp DM 정책은 **페어링**입니다. 알 수 없는 보내는 사람은 페어링 코드만 받고 메시지는 **처리되지 않습니다**. OpenClaw는 수신하는 채팅이나 명시적으로 트리거하는 전송에만 응답합니다.

페어링 승인:

```bash
openclaw pairing approve whatsapp <code>
```

대기 중인 요청 나열:

```bash
openclaw pairing list whatsapp
```

마법사 전화번호 프롬프트: DM이 허용되도록 **허용 목록/소유자**를 설정하는 데 사용됩니다. 자동 전송에는 사용되지 않습니다. 개인 WhatsApp 번호로 실행하면 해당 번호를 사용하고 `channels.whatsapp.selfChatMode`를 활성화하세요.

## 채팅 명령, 작업 중단, "멈추지 않습니다"

### 내부 시스템 메시지가 채팅에 표시되지 않게 하려면

대부분의 내부 또는 도구 메시지는 해당 세션에서 **verbose** 또는 **reasoning**이 활성화된 경우에만 나타납니다.

보이는 채팅에서 수정:

```
/verbose off
/reasoning off
```

여전히 시끄러우면 Control UI에서 세션 설정을 확인하고 verbose를 **inherit**로 설정하세요. 또한 구성에서 `verboseDefault`가 `on`으로 설정된 봇 프로필을 사용하고 있지 않은지 확인하세요.

문서: [생각 및 verbose](/tools/thinking), [보안](/gateway/security#reasoning--verbose-output-in-groups).

### 실행 중인 작업을 중지/취소하려면

다음 중 하나를 **독립 메시지**로 보내세요 (슬래시 없음):

```
stop
abort
esc
wait
exit
interrupt
```

이것들은 중단 트리거입니다 (슬래시 명령이 아님).

백그라운드 프로세스(exec 도구에서)의 경우 에이전트에게 실행을 요청할 수 있습니다:

```
process action:kill sessionId:XXX
```

슬래시 명령 개요: [슬래시 명령](/tools/slash-commands)을 참조하세요.

대부분의 명령은 `/`로 시작하는 **독립** 메시지로 보내야 하지만, 일부 단축키(예: `/status`)는 허용된 보내는 사람에게 인라인으로도 작동합니다.

### Telegram에서 Discord 메시지를 보내려면 ("Cross-context messaging denied")

OpenClaw는 기본적으로 **크로스 프로바이더** 메시징을 차단합니다. 도구 호출이 Telegram에 바인딩되어 있으면 명시적으로 허용하지 않는 한 Discord로 보내지 않습니다.

에이전트에 대한 크로스 프로바이더 메시징 활성화:

```json5
{
  agents: {
    defaults: {
      tools: {
        message: {
          crossContext: {
            allowAcrossProviders: true,
            marker: { enabled: true, prefix: "[from {channel}] " },
          },
        },
      },
    },
  },
}
```

구성 편집 후 게이트웨이를 재시작하세요. 단일 에이전트에 대해서만 원하면 `agents.list[].tools.message` 아래에 설정하세요.

### 봇이 빠른 연속 메시지를 무시하는 것 같은 이유는

큐 모드는 새 메시지가 진행 중인 실행과 어떻게 상호작용하는지 제어합니다. `/queue`를 사용하여 모드를 변경하세요:

- `steer` - 새 메시지가 현재 작업을 리디렉션
- `followup` - 메시지를 하나씩 실행
- `collect` - 메시지를 배치하고 한 번에 응답 (기본값)
- `steer-backlog` - 지금 조종하고 백로그 처리
- `interrupt` - 현재 실행을 중단하고 새로 시작

followup 모드에 `debounce:2s cap:25 drop:summarize` 같은 옵션을 추가할 수 있습니다.

## 스크린샷/채팅 로그의 정확한 질문에 대한 답변

**Q: "API 키가 있는 Anthropic의 기본 모델은 무엇인가요?"**

**A:** OpenClaw에서 자격 증명과 모델 선택은 별개입니다. `ANTHROPIC_API_KEY`를 설정하거나(또는 인증 프로필에 Anthropic API 키를 저장하면) 인증이 활성화되지만, 실제 기본 모델은 `agents.defaults.model.primary`에서 구성한 것입니다 (예: `anthropic/claude-sonnet-4-5` 또는 `anthropic/claude-opus-4-6`). `No credentials found for profile "anthropic:default"`가 보이면 Gateway가 실행 중인 에이전트의 예상 `auth-profiles.json`에서 Anthropic 자격 증명을 찾을 수 없었다는 의미입니다.

---

여전히 문제가 있나요? [Discord](https://discord.com/invite/clawd)에서 질문하거나 [GitHub 토론](https://github.com/openclaw/openclaw/discussions)을 열어주세요.
