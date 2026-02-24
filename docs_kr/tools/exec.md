---
summary: "Exec 도구 사용법, stdin 모드 및 TTY 지원"
read_when:
  - exec 도구를 사용하거나 수정할 때
  - stdin 또는 TTY 동작을 디버깅할 때
title: "Exec 도구"
---

# Exec 도구

워크스페이스에서 셸 명령을 실행합니다. `process`를 통해 포그라운드 + 백그라운드 실행을 지원합니다.
`process`가 비허용된 경우, `exec`는 동기적으로 실행되며 `yieldMs`/`background`를 무시합니다.
백그라운드 세션은 에이전트별로 범위가 지정됩니다; `process`는 같은 에이전트의 세션만 볼 수 있습니다.

## 매개변수

- `command` (필수)
- `workdir` (기본값은 cwd)
- `env` (키/값 재정의)
- `yieldMs` (기본값 10000): 지연 후 자동 백그라운드
- `background` (bool): 즉시 백그라운드
- `timeout` (초, 기본값 1800): 만료 시 종료
- `pty` (bool): 가능한 경우 의사 터미널에서 실행 (TTY 전용 CLI, 코딩 에이전트, 터미널 UI)
- `host` (`sandbox | gateway | node`): 실행 위치
- `security` (`deny | allowlist | full`): `gateway`/`node`에 대한 적용 모드
- `ask` (`off | on-miss | always`): `gateway`/`node`에 대한 승인 프롬프트
- `node` (string): `host=node`에 대한 노드 id/이름
- `elevated` (bool): 상승 모드 요청 (게이트웨이 호스트); `security=full`은 elevated가 `full`로 해결될 때만 강제

참고:

- `host` 기본값은 `sandbox`입니다.
- `elevated`는 샌드박싱이 꺼져 있을 때 무시됩니다 (exec가 이미 호스트에서 실행).
- `gateway`/`node` 승인은 `~/.openclaw/exec-approvals.json`으로 제어됩니다.
- `node`는 페어링된 노드 (컴패니언 앱 또는 헤드리스 노드 호스트)가 필요합니다.
- 여러 노드가 사용 가능하면, `exec.node` 또는 `tools.exec.node`를 설정하여 하나를 선택하세요.
- 비 Windows 호스트에서, exec는 `SHELL`이 설정되었을 때 이를 사용합니다; `SHELL`이 `fish`인 경우, fish 비호환 스크립트를 피하기 위해 `PATH`에서 `bash` (또는 `sh`)를 선호한 다음, 둘 다 없으면 `SHELL`로 폴백합니다.
- 호스트 실행 (`gateway`/`node`)은 바이너리 하이재킹이나 주입된 코드를 방지하기 위해 `env.PATH`와 로더 재정의 (`LD_*`/`DYLD_*`)를 거부합니다.
- 중요: 샌드박싱은 **기본적으로 꺼져** 있습니다. 샌드박싱이 꺼져 있고 `host=sandbox`가 명시적으로 설정/요청된 경우, exec는 게이트웨이 호스트에서 조용히 실행하는 대신 실패 종료합니다. 샌드박싱을 활성화하거나 승인과 함께 `host=gateway`를 사용하세요.
- 스크립트 사전 검사 (일반적인 Python/Node 셸 구문 실수)는 유효 `workdir` 경계 내의 파일만 검사합니다. 스크립트 경로가 `workdir` 외부로 해결되면, 해당 파일의 사전 검사는 건너뜁니다.

## 설정

- `tools.exec.notifyOnExit` (기본값: true): true이면, 백그라운드된 exec 세션이 종료 시 시스템 이벤트를 큐에 넣고 하트비트를 요청합니다.
- `tools.exec.approvalRunningNoticeMs` (기본값: 10000): 승인 게이팅된 exec가 이 시간보다 오래 실행되면 단일 "running" 알림을 발생시킵니다 (0은 비활성화).
- `tools.exec.host` (기본값: `sandbox`)
- `tools.exec.security` (기본값: sandbox는 `deny`, 설정되지 않은 경우 gateway + node는 `allowlist`)
- `tools.exec.ask` (기본값: `on-miss`)
- `tools.exec.node` (기본값: 미설정)
- `tools.exec.pathPrepend`: exec 실행을 위해 `PATH`에 앞에 추가할 디렉토리 목록 (gateway + sandbox 전용).
- `tools.exec.safeBins`: 명시적 허용 목록 항목 없이 실행할 수 있는 stdin 전용 안전 바이너리. 동작 세부 사항은 [안전 바이너리](/tools/exec-approvals#safe-bins-stdin-only)를 참조하세요.
- `tools.exec.safeBinTrustedDirs`: `safeBins` 경로 검사를 위해 신뢰되는 추가 명시적 디렉토리. `PATH` 항목은 자동으로 신뢰되지 않습니다.
- `tools.exec.safeBinProfiles`: 안전 바이너리별 선택적 커스텀 argv 정책 (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`).

예시:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH 처리

- `host=gateway`: 로그인 셸 `PATH`를 exec 환경에 병합합니다. 호스트 실행에 대해 `env.PATH` 재정의는 거부됩니다. 데몬 자체는 최소 `PATH`로 실행됩니다:
  - macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  - Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
- `host=sandbox`: 컨테이너 내에서 `sh -lc` (로그인 셸)를 실행하므로, `/etc/profile`이 `PATH`를 재설정할 수 있습니다. OpenClaw는 내부 env 변수를 통해 프로파일 소싱 후 `env.PATH`를 앞에 추가합니다 (셸 보간 없음); `tools.exec.pathPrepend`도 여기에 적용됩니다.
- `host=node`: 전달하는 비차단 env 재정의만 노드로 전송됩니다. 호스트 실행에 대해 `env.PATH` 재정의는 거부되며 노드 호스트에서 무시됩니다. 노드에 추가 PATH 항목이 필요하면, 노드 호스트 서비스 환경 (systemd/launchd)을 설정하거나 표준 위치에 도구를 설치하세요.

에이전트별 노드 바인딩 (설정의 에이전트 목록 인덱스 사용):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Control UI: Nodes 탭에 동일한 설정을 위한 작은 "Exec node binding" 패널이 포함되어 있습니다.

## 세션 재정의 (`/exec`)

`/exec`를 사용하여 `host`, `security`, `ask`, `node`에 대한 **세션별** 기본값을 설정하세요.
인수 없이 `/exec`를 전송하면 현재 값이 표시됩니다.

예시:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## 권한 부여 모델

`/exec`는 **승인된 발신자** (채널 허용 목록/페어링 + `commands.useAccessGroups`)에게만 적용됩니다.
**세션 상태만** 업데이트하며 설정을 쓰지 않습니다. exec를 완전히 비활성화하려면 도구 정책을 통해 거부하세요 (`tools.deny: ["exec"]` 또는 에이전트별). `security=full`과 `ask=off`를 명시적으로 설정하지 않으면 호스트 승인이 여전히 적용됩니다.

## Exec 승인 (컴패니언 앱 / 노드 호스트)

샌드박스된 에이전트는 `exec`가 게이트웨이 또는 노드 호스트에서 실행되기 전에 요청별 승인을 요구할 수 있습니다.
정책, 허용 목록 및 UI 흐름은 [Exec 승인](/tools/exec-approvals)을 참조하세요.

승인이 필요한 경우, exec 도구는 `status: "approval-pending"`과 승인 id와 함께 즉시 반환됩니다. 승인 (또는 거부 / 타임아웃) 후 게이트웨이는 시스템 이벤트 (`Exec finished` / `Exec denied`)를 발생시킵니다. 명령이 `tools.exec.approvalRunningNoticeMs` 이후에도 실행 중이면, 단일 `Exec running` 알림이 발생합니다.

## 허용 목록 + 안전 바이너리

수동 허용 목록 적용은 **해결된 바이너리 경로만** 일치합니다 (기본 이름 일치 없음). `security=allowlist`일 때, 셸 명령은 모든 파이프라인 세그먼트가 허용 목록에 있거나 안전 바이너리인 경우에만 자동 허용됩니다. 체이닝 (`;`, `&&`, `||`)과 리디렉션은 모든 최상위 세그먼트가 허용 목록 (안전 바이너리 포함)을 만족하지 않으면 allowlist 모드에서 거부됩니다. 리디렉션은 지원되지 않습니다.

`autoAllowSkills`는 exec 승인의 별도 편의 경로입니다. 수동 경로 허용 목록 항목과 같지 않습니다. 엄격한 명시적 신뢰를 위해 `autoAllowSkills`를 비활성화된 상태로 유지하세요.

두 가지 제어를 다른 작업에 사용하세요:

- `tools.exec.safeBins`: 작은, stdin 전용 스트림 필터.
- `tools.exec.safeBinTrustedDirs`: 안전 바이너리 실행 파일 경로를 위한 명시적 추가 신뢰 디렉토리.
- `tools.exec.safeBinProfiles`: 커스텀 안전 바이너리를 위한 명시적 argv 정책.
- 허용 목록: 실행 파일 경로에 대한 명시적 신뢰.

`safeBins`를 일반 허용 목록으로 취급하지 마세요, 인터프리터/런타임 바이너리 (예: `python3`, `node`, `ruby`, `bash`)를 추가하지 마세요. 이들이 필요하면 명시적 허용 목록 항목을 사용하고 승인 프롬프트를 활성화된 상태로 유지하세요.
`openclaw security audit`는 인터프리터/런타임 `safeBins` 항목에 명시적 프로파일이 누락되었을 때 경고하며, `openclaw doctor --fix`는 누락된 커스텀 `safeBinProfiles` 항목을 스캐폴딩할 수 있습니다.

전체 정책 세부 사항과 예시는 [Exec 승인](/tools/exec-approvals#safe-bins-stdin-only) 및 [안전 바이너리 vs 허용 목록](/tools/exec-approvals#safe-bins-versus-allowlist)을 참조하세요.

## 예시

포그라운드:

```json
{ "tool": "exec", "command": "ls -la" }
```

백그라운드 + 폴링:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

키 전송 (tmux 스타일):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

제출 (CR만 전송):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

붙여넣기 (기본적으로 괄호 처리):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply_patch (실험적)

`apply_patch`는 구조화된 다중 파일 편집을 위한 `exec`의 하위 도구입니다.
명시적으로 활성화하세요:

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

참고:

- OpenAI/OpenAI Codex 모델에서만 사용 가능합니다.
- 도구 정책이 여전히 적용됩니다; `allow: ["exec"]`는 암묵적으로 `apply_patch`를 허용합니다.
- 설정은 `tools.exec.applyPatch` 아래에 있습니다.
- `tools.exec.applyPatch.workspaceOnly`는 기본값이 `true`입니다 (워크스페이스 내 제한). `apply_patch`가 워크스페이스 디렉토리 외부에 쓰기/삭제하도록 의도적으로 허용하려면 `false`로 설정하세요.
