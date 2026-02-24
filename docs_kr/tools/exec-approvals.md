---
summary: "Exec 승인, 허용 목록 및 샌드박스 탈출 프롬프트"
read_when:
  - exec 승인 또는 허용 목록을 설정할 때
  - macOS 앱에서 exec 승인 UX를 구현할 때
  - 샌드박스 탈출 프롬프트와 그 함의를 검토할 때
title: "Exec 승인"
---

# Exec 승인

Exec 승인은 샌드박스된 에이전트가 실제 호스트 (`gateway` 또는 `node`)에서 명령을 실행할 수 있도록 하는 **컴패니언 앱 / 노드 호스트 가드레일**입니다. 안전 인터록과 같다고 생각하세요: 정책 + 허용 목록 + (선택적) 사용자 승인이 모두 동의할 때만 명령이 허용됩니다.
Exec 승인은 도구 정책과 상승 게이팅에 **추가**됩니다 (상승 모드가 `full`로 설정된 경우 승인을 건너뜁니다).
유효 정책은 `tools.exec.*`와 승인 기본값 중 **더 엄격한** 것입니다; 승인 필드가 생략되면 `tools.exec` 값이 사용됩니다.

컴패니언 앱 UI를 **사용할 수 없는** 경우, 프롬프트가 필요한 모든 요청은 **ask 폴백** (기본값: 거부)으로 해결됩니다.

## 적용 대상

Exec 승인은 실행 호스트에서 로컬로 적용됩니다:

- **게이트웨이 호스트** → 게이트웨이 머신의 `openclaw` 프로세스
- **노드 호스트** → 노드 러너 (macOS 컴패니언 앱 또는 헤드리스 노드 호스트)

신뢰 모델 참고:

- 게이트웨이 인증된 호출자는 해당 게이트웨이의 신뢰된 운영자입니다.
- 페어링된 노드는 그 신뢰된 운영자 능력을 노드 호스트로 확장합니다.
- Exec 승인은 실수로 인한 실행 위험을 줄이지만, 사용자별 인증 경계는 아닙니다.

macOS 분할:

- **노드 호스트 서비스**는 로컬 IPC를 통해 `system.run`을 **macOS 앱**으로 전달합니다.
- **macOS 앱**은 승인을 적용하고 UI 컨텍스트에서 명령을 실행합니다.

## 설정 및 저장소

승인은 실행 호스트의 로컬 JSON 파일에 저장됩니다:

`~/.openclaw/exec-approvals.json`

예시 스키마:

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## 정책 조절

### 보안 (`exec.security`)

- **deny**: 모든 호스트 exec 요청을 차단합니다.
- **allowlist**: 허용 목록에 있는 명령만 허용합니다.
- **full**: 모든 것을 허용합니다 (상승 모드와 동등).

### Ask (`exec.ask`)

- **off**: 절대 프롬프트하지 않습니다.
- **on-miss**: 허용 목록이 일치하지 않을 때만 프롬프트합니다.
- **always**: 모든 명령에 대해 프롬프트합니다.

### Ask 폴백 (`askFallback`)

프롬프트가 필요하지만 UI에 접근할 수 없는 경우, 폴백이 결정합니다:

- **deny**: 차단.
- **allowlist**: 허용 목록이 일치하는 경우에만 허용.
- **full**: 허용.

## 허용 목록 (에이전트별)

허용 목록은 **에이전트별**입니다. 여러 에이전트가 있는 경우 macOS 앱에서 편집할 에이전트를 전환하세요. 패턴은 **대소문자 구분 없는 glob 일치**입니다.
패턴은 **바이너리 경로**로 해결되어야 합니다 (기본 이름만 있는 항목은 무시됩니다).
레거시 `agents.default` 항목은 로드 시 `agents.main`으로 마이그레이션됩니다.

예시:

- `~/Projects/**/bin/peekaboo`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

각 허용 목록 항목은 다음을 추적합니다:

- **id** UI 식별에 사용되는 안정적인 UUID (선택 사항)
- **마지막 사용** 타임스탬프
- **마지막 사용된 명령**
- **마지막 해결된 경로**

## 스킬 CLI 자동 허용

**스킬 CLI 자동 허용**이 활성화되면, 알려진 스킬이 참조하는 실행 파일은 노드 (macOS 노드 또는 헤드리스 노드 호스트)에서 허용 목록에 있는 것으로 처리됩니다. 이는 게이트웨이 RPC를 통해 `skills.bins`를 사용하여 스킬 바이너리 목록을 가져옵니다. 엄격한 수동 허용 목록을 원하면 비활성화하세요.

중요한 신뢰 참고 사항:

- 이것은 수동 경로 허용 목록 항목과 별개인 **암묵적 편의 허용 목록**입니다.
- 게이트웨이와 노드가 같은 신뢰 경계에 있는 신뢰된 운영자 환경을 위한 것입니다.
- 엄격한 명시적 신뢰가 필요하면 `autoAllowSkills: false`를 유지하고 수동 경로 허용 목록 항목만 사용하세요.

## 안전 바이너리 (stdin 전용)

`tools.exec.safeBins`는 명시적 허용 목록 항목 없이 allowlist 모드에서 실행할 수 있는 **stdin 전용** 바이너리 (예: `jq`)의 작은 목록을 정의합니다. 안전 바이너리는 위치 파일 인수와 경로 유사 토큰을 거부하므로, 들어오는 스트림에서만 작동할 수 있습니다.
이것을 일반적인 신뢰 목록이 아닌 스트림 필터를 위한 좁은 빠른 경로로 취급하세요.
인터프리터 또는 런타임 바이너리 (예: `python3`, `node`, `ruby`, `bash`, `sh`, `zsh`)를 `safeBins`에 추가하지 **마세요**.
명령이 코드를 평가하거나, 하위 명령을 실행하거나, 설계상 파일을 읽을 수 있는 경우 명시적 허용 목록 항목을 사용하고 승인 프롬프트를 활성화된 상태로 유지하세요.
커스텀 안전 바이너리는 `tools.exec.safeBinProfiles.<bin>`에 명시적 프로파일을 정의해야 합니다.
검증은 argv 형태에서만 결정론적입니다 (호스트 파일시스템 존재 확인 없음), 이는 허용/거부 차이로 인한 파일 존재 오라클 동작을 방지합니다.
파일 지향 옵션은 기본 안전 바이너리에 대해 거부됩니다 (예: `sort -o`, `sort --output`, `sort --files0-from`, `sort --compress-program`, `sort --random-source`, `sort --temporary-directory`/`-T`, `wc --files0-from`, `jq -f/--from-file`, `grep -f/--file`).
안전 바이너리는 또한 stdin 전용 동작을 깨는 옵션에 대해 명시적 바이너리별 플래그 정책을 적용합니다 (예: `sort -o/--output/--compress-program` 및 grep 재귀 플래그).
긴 옵션은 안전 바이너리 모드에서 fail-closed로 검증됩니다: 알 수 없는 플래그와 모호한 약어는 거부됩니다.
안전 바이너리 프로파일별 거부된 플래그:

<!-- SAFE_BIN_DENIED_FLAGS:START -->

- `grep`: `--dereference-recursive`, `--directories`, `--exclude-from`, `--file`, `--recursive`, `-R`, `-d`, `-f`, `-r`
- `jq`: `--argfile`, `--from-file`, `--library-path`, `--rawfile`, `--slurpfile`, `-L`, `-f`
- `sort`: `--compress-program`, `--files0-from`, `--output`, `--random-source`, `--temporary-directory`, `-T`, `-o`
- `wc`: `--files0-from`
<!-- SAFE_BIN_DENIED_FLAGS:END -->

안전 바이너리는 또한 argv 토큰을 실행 시 stdin 전용 세그먼트에 대해 **리터럴 텍스트**로 처리하도록 강제합니다 (글로빙 없음, `$VARS` 확장 없음), 따라서 `*`이나 `$HOME/...`과 같은 패턴으로 파일 읽기를 밀수입할 수 없습니다.
안전 바이너리는 또한 신뢰된 바이너리 디렉토리 (시스템 기본값 + 선택적 `tools.exec.safeBinTrustedDirs`)에서 해결되어야 합니다. `PATH` 항목은 자동으로 신뢰되지 않습니다.
셸 체이닝과 리디렉션은 allowlist 모드에서 자동 허용되지 않습니다.

셸 체이닝 (`&&`, `||`, `;`)은 모든 최상위 세그먼트가 허용 목록 (안전 바이너리 또는 스킬 자동 허용 포함)을 만족하는 경우 허용됩니다. 리디렉션은 allowlist 모드에서 지원되지 않습니다.
명령 치환 (`$()` / 백틱)은 이중 따옴표 내부를 포함하여 allowlist 파싱 중에 거부됩니다; 리터럴 `$()`텍스트가 필요하면 작은 따옴표를 사용하세요.
macOS 컴패니언 앱 승인에서는 셸 제어 또는 확장 구문 (`&&`, `||`, `;`, `|`, `` ` ``, `$`, `<`, `>`, `(`, `)`)을 포함하는 원시 셸 텍스트는 셸 바이너리 자체가 허용 목록에 없으면 허용 목록 미스로 처리됩니다.
셸 래퍼 (`bash|sh|zsh ... -c/-lc`)의 경우, 요청 범위 env 재정의는 작은 명시적 허용 목록 (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`)으로 축소됩니다.
allowlist 모드에서 항상 허용 결정의 경우, 알려진 디스패치 래퍼 (`env`, `nice`, `nohup`, `stdbuf`, `timeout`)는 래퍼 경로 대신 내부 실행 파일 경로를 유지합니다. 셸 멀티플렉서 (`busybox`, `toybox`)도 셸 애플릿 (`sh`, `ash` 등)에 대해 언래핑되어 멀티플렉서 바이너리 대신 내부 실행 파일이 유지됩니다. 래퍼나 멀티플렉서를 안전하게 언래핑할 수 없으면 허용 목록 항목이 자동으로 유지되지 않습니다.

기본 안전 바이너리: `jq`, `cut`, `uniq`, `head`, `tail`, `tr`, `wc`.

`grep`과 `sort`는 기본 목록에 없습니다. 옵트인하는 경우, 비 stdin 워크플로우를 위한 명시적 허용 목록 항목을 유지하세요.
안전 바이너리 모드에서 `grep`의 경우, `-e`/`--regexp`로 패턴을 제공하세요; 위치 패턴 형식은 거부되어 파일 피연산자가 모호한 위치 인수로 밀수입될 수 없습니다.

### 안전 바이너리 vs 허용 목록

| 주제             | `tools.exec.safeBins`                                  | 허용 목록 (`exec-approvals.json`)                            |
| ---------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| 목표             | 좁은 stdin 필터 자동 허용                              | 특정 실행 파일을 명시적으로 신뢰                             |
| 일치 유형        | 실행 파일 이름 + 안전 바이너리 argv 정책               | 해결된 실행 파일 경로 glob 패턴                              |
| 인수 범위        | 안전 바이너리 프로파일과 리터럴 토큰 규칙으로 제한     | 경로 일치만; 인수는 사용자 책임                              |
| 일반적인 예시    | `jq`, `head`, `tail`, `wc`                             | `python3`, `node`, `ffmpeg`, 커스텀 CLI                      |
| 최적 사용        | 파이프라인의 저위험 텍스트 변환                         | 광범위한 동작이나 부작용이 있는 모든 도구                    |

설정 위치:

- `safeBins`는 설정에서 가져옵니다 (`tools.exec.safeBins` 또는 에이전트별 `agents.list[].tools.exec.safeBins`).
- `safeBinTrustedDirs`는 설정에서 가져옵니다 (`tools.exec.safeBinTrustedDirs` 또는 에이전트별 `agents.list[].tools.exec.safeBinTrustedDirs`).
- `safeBinProfiles`는 설정에서 가져옵니다 (`tools.exec.safeBinProfiles` 또는 에이전트별 `agents.list[].tools.exec.safeBinProfiles`). 에이전트별 프로파일 키가 전역 키를 재정의합니다.
- 허용 목록 항목은 `agents.<id>.allowlist` 아래의 호스트 로컬 `~/.openclaw/exec-approvals.json`에 있습니다 (또는 Control UI / `openclaw approvals allowlist ...`를 통해).
- `openclaw security audit`는 인터프리터/런타임 바이너리가 명시적 프로파일 없이 `safeBins`에 나타날 때 `tools.exec.safe_bins_interpreter_unprofiled`로 경고합니다.
- `openclaw doctor --fix`는 누락된 커스텀 `safeBinProfiles.<bin>` 항목을 `{}`로 스캐폴딩할 수 있습니다 (이후 검토하고 강화하세요). 인터프리터/런타임 바이너리는 자동 스캐폴딩되지 않습니다.

커스텀 프로파일 예시:

```json5
{
  tools: {
    exec: {
      safeBins: ["jq", "myfilter"],
      safeBinProfiles: {
        myfilter: {
          minPositional: 0,
          maxPositional: 0,
          allowedValueFlags: ["-n", "--limit"],
          deniedFlags: ["-f", "--file", "-c", "--command"],
        },
      },
    },
  },
}
```

## Control UI 편집

**Control UI -> Nodes -> Exec approvals** 카드를 사용하여 기본값, 에이전트별 재정의 및 허용 목록을 편집하세요. 범위 (기본값 또는 에이전트)를 선택하고, 정책을 조정하고, 허용 목록 패턴을 추가/제거한 다음 **저장**하세요. UI는 패턴별 **마지막 사용** 메타데이터를 보여주므로 목록을 정리할 수 있습니다.

대상 선택기는 **게이트웨이** (로컬 승인) 또는 **노드**를 선택합니다. 노드는 `system.execApprovals.get/set`을 알려야 합니다 (macOS 앱 또는 헤드리스 노드 호스트). 노드가 아직 exec 승인을 알리지 않으면, 로컬 `~/.openclaw/exec-approvals.json`을 직접 편집하세요.

CLI: `openclaw approvals`는 게이트웨이 또는 노드 편집을 지원합니다 ([승인 CLI](/cli/approvals) 참조).

## 승인 흐름

프롬프트가 필요한 경우, 게이트웨이는 `exec.approval.requested`를 운영자 클라이언트에 브로드캐스트합니다. Control UI와 macOS 앱은 `exec.approval.resolve`를 통해 해결한 다음, 게이트웨이가 승인된 요청을 노드 호스트로 전달합니다.

승인이 필요한 경우, exec 도구는 승인 id와 함께 즉시 반환됩니다. 나중에 시스템 이벤트 (`Exec finished` / `Exec denied`)를 상관시키기 위해 해당 id를 사용하세요. 타임아웃 전에 결정이 도착하지 않으면, 요청은 승인 타임아웃으로 처리되고 거부 사유로 표시됩니다.

확인 대화상자에는 다음이 포함됩니다:

- 명령 + 인수
- cwd
- 에이전트 id
- 해결된 실행 파일 경로
- 호스트 + 정책 메타데이터

액션:

- **한 번 허용** -> 지금 실행
- **항상 허용** -> 허용 목록에 추가 + 실행
- **거부** -> 차단

## 채팅 채널로의 승인 전달

exec 승인 프롬프트를 모든 채팅 채널 (플러그인 채널 포함)로 전달하고 `/approve`로 승인할 수 있습니다. 이는 일반 아웃바운드 전달 파이프라인을 사용합니다.

설정:

```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // 부분 문자열 또는 정규식
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" },
      ],
    },
  },
}
```

채팅에서 응답:

```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

### macOS IPC 흐름

```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

보안 참고:

- Unix 소켓 모드 `0600`, 토큰은 `exec-approvals.json`에 저장.
- 동일 UID 피어 검사.
- 챌린지/응답 (nonce + HMAC 토큰 + 요청 해시) + 짧은 TTL.

## 시스템 이벤트

Exec 생명주기는 시스템 메시지로 표시됩니다:

- `Exec running` (명령이 실행 중 알림 임계값을 초과한 경우에만)
- `Exec finished`
- `Exec denied`

노드가 이벤트를 보고한 후 에이전트의 세션에 게시됩니다.
게이트웨이 호스트 exec 승인은 명령이 완료될 때 동일한 생명주기 이벤트를 발생시킵니다 (선택적으로 임계값보다 오래 실행될 때도).
승인 게이팅된 exec는 쉬운 상관을 위해 이 메시지에서 승인 id를 `runId`로 재사용합니다.

## 함의

- **full**은 강력합니다; 가능하면 허용 목록을 선호하세요.
- **ask**는 빠른 승인을 허용하면서도 사용자를 루프에 유지합니다.
- 에이전트별 허용 목록은 한 에이전트의 승인이 다른 에이전트로 누출되는 것을 방지합니다.
- 승인은 **승인된 발신자**의 호스트 exec 요청에만 적용됩니다. 승인되지 않은 발신자는 `/exec`를 실행할 수 없습니다.
- `/exec security=full`은 승인된 운영자를 위한 세션 수준 편의 기능이며 설계상 승인을 건너뜁니다.
  호스트 exec를 완전히 차단하려면, 승인 보안을 `deny`로 설정하거나 도구 정책을 통해 `exec` 도구를 거부하세요.

관련:

- [Exec 도구](/tools/exec)
- [상승 모드](/tools/elevated)
- [스킬](/tools/skills)
