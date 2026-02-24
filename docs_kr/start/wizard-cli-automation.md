---
summary: "OpenClaw CLI를 이용한 스크립트 기반 온보딩 및 에이전트 설정 가이드"
read_when:
  - 스크립트나 CI 환경에서 온보딩을 자동화할 때
  - 특정 프로바이더에 대한 비대화형(non-interactive) 설정 예시가 필요할 때
title: "CLI 자동화"
sidebarTitle: "CLI 자동화"
---

# CLI 자동화

`openclaw onboard` 명령 실행 시 `--non-interactive` 플래그를 사용하여 온보딩 과정을 자동화할 수 있습니다.

<Note>
`--json` 플래그가 자동화를 의미하지는 않습니다. 자동화 스크립트에서는 반드시 `--non-interactive` (그리고 가급적 `--workspace`) 플래그를 함께 사용하세요.
</Note>

## 기본 자동화 예시

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

기계 판독 가능한 요약 정보를 보려면 `--json` 플래그를 추가하세요.

## 프로바이더별 예시

<AccordionGroup>
  <Accordion title="Gemini 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Mistral 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice mistral-api-key \
      --mistral-api-key "$MISTRAL_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="커스텀 프로바이더 (Custom Provider) 예시">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice custom-api-key \
      --custom-base-url "https://llm.example.com/v1" \
      --custom-model-id "foo-large" \
      --custom-api-key "$CUSTOM_API_KEY" \
      --custom-provider-id "my-custom" \
      --custom-compatibility anthropic \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```

    `--custom-api-key`는 선택 사항입니다. 생략하면 온보딩 시 `CUSTOM_API_KEY` 환경 변수를 확인합니다.

  </Accordion>
</AccordionGroup>

## 추가 에이전트 생성

자체 워크스페이스, 세션, 인증 프로필을 가진 별개의 에이전트를 추가하려면 `openclaw agents add <이름>` 명령을 사용하세요. `--workspace` 플래그 없이 실행하면 위저드가 시작됩니다.

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

설정되는 항목들:

- `agents.list[].name` (에이전트 이름)
- `agents.list[].workspace` (워크스페이스 경로)
- `agents.list[].agentDir` (에이전트 데이터 디렉토리)

참고 사항:

- 기본 워크스페이스는 `~/.openclaw/workspace-<agentId>` 경로를 따릅니다.
- 수신 메시지 라우팅을 위해 `bindings`를 추가하세요 (위저드에서 처리 가능).
- 자동화 플래그: `--model`, `--agent-dir`, `--bind`, `--non-interactive`.

## 관련 문서

- 온보딩 허브: [온보딩 위저드 (CLI)](/start/wizard)
- 전체 참조 가이드: [CLI 온보딩 참조 가이드](/start/wizard-cli-reference)
- 명령 참조: [`openclaw onboard`](/cli/onboard)
