---
summary: "`openclaw hooks` CLI 참조 (에이전트 후크)"
read_when:
  - 에이전트 후크를 관리하려는 경우
  - 후크를 설치하거나 업데이트하려는 경우
title: "hooks"
---

# `openclaw hooks`

에이전트 후크를 관리합니다 (`/new`, `/reset` 같은 명령과 Gateway 시작 시 트리거되는 이벤트 기반 자동화).

관련 항목:

- 후크: [Hooks](/automation/hooks)
- 플러그인 후크: [Plugins](/tools/plugin#plugin-hooks)

## 모든 후크 나열

```bash
openclaw hooks list
```

워크스페이스, 관리, 번들 디렉토리에서 발견된 모든 후크를 나열합니다.

**옵션:**

- `--eligible`: 적합한 후크만 표시 (요구사항 충족)
- `--json`: JSON으로 출력
- `-v, --verbose`: 누락된 요구사항을 포함한 상세 정보 표시

**출력 예시:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📎 bootstrap-extra-files ✓ - Inject extra workspace bootstrap files during agent bootstrap
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
```

**예시 (상세):**

```bash
openclaw hooks list --verbose
```

부적합한 후크의 누락된 요구사항을 표시합니다.

**예시 (JSON):**

```bash
openclaw hooks list --json
```

프로그래밍 방식 사용을 위해 구조화된 JSON을 반환합니다.

## 후크 정보 조회

```bash
openclaw hooks info <name>
```

특정 후크에 대한 상세 정보를 표시합니다.

**인수:**

- `<name>`: 후크 이름 (예: `session-memory`)

**옵션:**

- `--json`: JSON으로 출력

**예시:**

```bash
openclaw hooks info session-memory
```

**출력:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/automation/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## 후크 적합성 확인

```bash
openclaw hooks check
```

후크 적합성 상태 요약을 표시합니다 (준비 완료 vs 미준비).

**옵션:**

- `--json`: JSON으로 출력

**출력 예시:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## 후크 활성화

```bash
openclaw hooks enable <name>
```

설정(`~/.openclaw/config.json`)에 추가하여 특정 후크를 활성화합니다.

**참고:** 플러그인이 관리하는 후크는 `openclaw hooks list`에서 `plugin:<id>`로 표시되며
여기서 활성화/비활성화할 수 없습니다. 대신 플러그인을 활성화/비활성화합니다.

**인수:**

- `<name>`: 후크 이름 (예: `session-memory`)

**예시:**

```bash
openclaw hooks enable session-memory
```

**출력:**

```
✓ Enabled hook: 💾 session-memory
```

**수행 작업:**

- 후크가 존재하고 적합한지 확인합니다
- 설정에서 `hooks.internal.entries.<name>.enabled = true`를 업데이트합니다
- 설정을 디스크에 저장합니다

**활성화 후:**

- 후크가 다시 로드되도록 Gateway를 다시 시작합니다 (macOS에서는 메뉴 바 앱 재시작, 또는 개발 중인 Gateway 프로세스 재시작).

## 후크 비활성화

```bash
openclaw hooks disable <name>
```

설정을 업데이트하여 특정 후크를 비활성화합니다.

**인수:**

- `<name>`: 후크 이름 (예: `command-logger`)

**예시:**

```bash
openclaw hooks disable command-logger
```

**출력:**

```
⏸ Disabled hook: 📝 command-logger
```

**비활성화 후:**

- 후크가 다시 로드되도록 Gateway를 다시 시작합니다

## 후크 설치

```bash
openclaw hooks install <path-or-spec>
openclaw hooks install <npm-spec> --pin
```

로컬 폴더/아카이브 또는 npm에서 후크 팩을 설치합니다.

npm 스펙은 **레지스트리 전용**입니다 (패키지 이름 + 선택적 버전/태그). Git/URL/파일
스펙은 거부됩니다. 의존성 설치는 안전을 위해 `--ignore-scripts`로 실행됩니다.

**수행 작업:**

- 후크 팩을 `~/.openclaw/hooks/<id>`로 복사합니다
- `hooks.internal.entries.*`에서 설치된 후크를 활성화합니다
- `hooks.internal.installs`에 설치를 기록합니다

**옵션:**

- `-l, --link`: 복사 대신 로컬 디렉토리를 링크합니다 (`hooks.internal.load.extraDirs`에 추가)
- `--pin`: npm 설치를 `hooks.internal.installs`에 정확한 `name@version`으로 기록합니다

**지원되는 아카이브:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

**예시:**

```bash
# 로컬 디렉토리
openclaw hooks install ./my-hook-pack

# 로컬 아카이브
openclaw hooks install ./my-hook-pack.zip

# NPM 패키지
openclaw hooks install @openclaw/my-hook-pack

# 복사 없이 로컬 디렉토리 링크
openclaw hooks install -l ./my-hook-pack
```

## 후크 업데이트

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

설치된 후크 팩을 업데이트합니다 (npm 설치만 해당).

**옵션:**

- `--all`: 추적된 모든 후크 팩을 업데이트합니다
- `--dry-run`: 기록하지 않고 변경될 내용을 표시합니다

저장된 무결성 해시가 존재하고 가져온 아티팩트 해시가 변경되면,
OpenClaw이 경고를 출력하고 진행하기 전에 확인을 요청합니다. CI/비대화형 실행에서
프롬프트를 우회하려면 전역 `--yes`를 사용합니다.

## 번들 후크

### session-memory

`/new`를 실행할 때 세션 컨텍스트를 메모리에 저장합니다.

**활성화:**

```bash
openclaw hooks enable session-memory
```

**출력:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**참조:** [session-memory 문서](/automation/hooks#session-memory)

### bootstrap-extra-files

`agent:bootstrap` 중에 추가 부트스트랩 파일(예: 모노레포 로컬 `AGENTS.md` / `TOOLS.md`)을 주입합니다.

**활성화:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**참조:** [bootstrap-extra-files 문서](/automation/hooks#bootstrap-extra-files)

### command-logger

모든 명령 이벤트를 중앙 감사 파일에 기록합니다.

**활성화:**

```bash
openclaw hooks enable command-logger
```

**출력:** `~/.openclaw/logs/commands.log`

**로그 보기:**

```bash
# 최근 명령
tail -n 20 ~/.openclaw/logs/commands.log

# 정렬된 출력
cat ~/.openclaw/logs/commands.log | jq .

# 액션별 필터
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**참조:** [command-logger 문서](/automation/hooks#command-logger)

### boot-md

Gateway가 시작될 때(채널 시작 후) `BOOT.md`를 실행합니다.

**이벤트**: `gateway:startup`

**활성화**:

```bash
openclaw hooks enable boot-md
```

**참조:** [boot-md 문서](/automation/hooks#boot-md)
