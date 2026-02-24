---
summary: "스킬 구성 스키마 및 예제"
read_when:
  - 스킬 구성을 추가하거나 수정하는 경우
  - 번들 허용 목록 또는 설치 동작을 조정하는 경우
title: "스킬 구성"
---

# 스킬 구성

모든 스킬 관련 구성은 `~/.openclaw/openclaw.json`의 `skills` 아래에 위치합니다.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway 런타임은 여전히 Node; bun 권장하지 않음)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## 필드

- `allowBundled`: **번들된** 스킬에 대한 선택적 허용 목록. 설정되면 목록에 있는 번들 스킬만 적격합니다(관리/워크스페이스 스킬은 영향 없음).
- `load.extraDirs`: 스캔할 추가 스킬 디렉토리(가장 낮은 우선순위).
- `load.watch`: 스킬 폴더를 감시하고 스킬 스냅샷을 새로 고침합니다(기본값: true).
- `load.watchDebounceMs`: 스킬 감시자 이벤트의 디바운스 시간(밀리초)(기본값: 250).
- `install.preferBrew`: 사용 가능한 경우 brew 설치를 선호합니다(기본값: true).
- `install.nodeManager`: node 설치 기본 설정(`npm` | `pnpm` | `yarn` | `bun`, 기본값: npm).
  이는 **스킬 설치**에만 영향을 미칩니다; Gateway 런타임은 여전히 Node여야 합니다(WhatsApp/Telegram에는 Bun 권장하지 않음).
- `entries.<skillKey>`: 스킬별 재정의.

스킬별 필드:

- `enabled`: 번들/설치된 스킬이라도 비활성화하려면 `false`로 설정합니다.
- `env`: 에이전트 실행에 주입되는 환경 변수(이미 설정되지 않은 경우에만).
- `apiKey`: 주요 환경 변수를 선언하는 스킬을 위한 선택적 편의 기능.

## 참고 사항

- `entries` 아래의 키는 기본적으로 스킬 이름에 매핑됩니다. 스킬이 `metadata.openclaw.skillKey`를 정의하면 해당 키를 대신 사용하세요.
- 감시자가 활성화된 경우 스킬에 대한 변경 사항은 다음 에이전트 턴에서 반영됩니다.

### 샌드박스된 스킬 + 환경 변수

세션이 **샌드박스**된 경우 스킬 프로세스는 Docker 내부에서 실행됩니다. 샌드박스는 호스트 `process.env`를 **상속하지 않습니다**.

다음 중 하나를 사용하세요:

- `agents.defaults.sandbox.docker.env`(또는 에이전트별 `agents.list[].sandbox.docker.env`)
- 사용자 정의 샌드박스 이미지에 환경 변수를 구워 넣기

전역 `env`와 `skills.entries.<skill>.env/apiKey`는 **호스트** 실행에만 적용됩니다.
