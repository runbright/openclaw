---
summary: "Together AI 설정 (인증 + 모델 선택)"
read_when:
  - OpenClaw에서 Together AI를 사용하고 싶을 때
  - API 키 환경 변수 또는 CLI 인증 선택이 필요할 때
---

# Together AI

[Together AI](https://together.ai)는 통합 API를 통해 Llama, DeepSeek, Kimi 등 주요 오픈소스 모델에 대한 접근을 제공합니다.

- 프로바이더: `together`
- 인증: `TOGETHER_API_KEY`
- API: OpenAI 호환

## 빠른 시작

1. API 키를 설정합니다 (권장: 게이트웨이용으로 저장):

```bash
openclaw onboard --auth-choice together-api-key
```

2. 기본 모델을 설정합니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## 비대화형 예시

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

이 명령은 `together/moonshotai/Kimi-K2.5`를 기본 모델로 설정합니다.

## 환경 참고 사항

게이트웨이가 데몬(launchd/systemd)으로 실행되는 경우, `TOGETHER_API_KEY`가
해당 프로세스에서 사용 가능한지 확인하세요 (예: `~/.openclaw/.env` 또는
`env.shellEnv` 사용).

## 사용 가능한 모델

Together AI는 다양한 인기 오픈소스 모델에 대한 접근을 제공합니다:

- **GLM 4.7 Fp8** - 200K 컨텍스트 윈도우를 가진 기본 모델
- **Llama 3.3 70B Instruct Turbo** - 빠르고 효율적인 지시 수행
- **Llama 4 Scout** - 이미지 이해 기능을 갖춘 비전 모델
- **Llama 4 Maverick** - 고급 비전 및 추론
- **DeepSeek V3.1** - 강력한 코딩 및 추론 모델
- **DeepSeek R1** - 고급 추론 모델
- **Kimi K2 Instruct** - 262K 컨텍스트 윈도우를 가진 고성능 모델

모든 모델은 표준 채팅 완성을 지원하며 OpenAI API와 호환됩니다.
