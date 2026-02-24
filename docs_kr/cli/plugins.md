---
summary: "`openclaw plugins` (목록, 설치, 제거, 활성화/비활성화, 진단) CLI 레퍼런스"
read_when:
  - 인프로세스 Gateway 플러그인을 설치하거나 관리하고 싶을 때
  - 플러그인 로드 실패를 디버깅하고 싶을 때
title: "plugins"
---

# `openclaw plugins`

Gateway 플러그인/확장 기능을 관리합니다 (인프로세스로 로드).

관련 항목:

- 플러그인 시스템: [플러그인](/tools/plugin)
- 플러그인 매니페스트 + 스키마: [플러그인 매니페스트](/plugins/manifest)
- 보안 강화: [보안](/gateway/security)

## 명령어

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins uninstall <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

번들 플러그인은 OpenClaw와 함께 제공되지만 비활성화된 상태로 시작합니다.
`plugins enable`을 사용하여 활성화하세요.

모든 플러그인은 인라인 JSON Schema(`configSchema`, 비어 있더라도)가 포함된
`openclaw.plugin.json` 파일을 포함해야 합니다. 매니페스트나 스키마가 없거나 유효하지 않으면
플러그인 로딩이 방지되고 구성 유효성 검사에 실패합니다.

### 설치

```bash
openclaw plugins install <path-or-spec>
openclaw plugins install <npm-spec> --pin
```

보안 참고: 플러그인 설치를 코드 실행처럼 취급하세요. 고정된 버전을 권장합니다.

npm 스펙은 **레지스트리 전용**입니다 (패키지 이름 + 선택적 버전/태그). Git/URL/파일
스펙은 거부됩니다. 의존성 설치는 안전을 위해 `--ignore-scripts`로 실행됩니다.

지원되는 아카이브: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

로컬 디렉토리를 복사하지 않으려면 `--link`를 사용하세요 (`plugins.load.paths`에 추가):

```bash
openclaw plugins install -l ./my-plugin
```

npm 설치 시 `--pin`을 사용하면 해석된 정확한 스펙(`name@version`)을
`plugins.installs`에 저장하면서 기본 동작은 고정되지 않은 상태를 유지합니다.

### 제거

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall`은 `plugins.entries`, `plugins.installs`, 플러그인 허용 목록 및
해당하는 경우 링크된 `plugins.load.paths` 항목에서 플러그인 레코드를 제거합니다.
활성 메모리 플러그인의 경우, 메모리 슬롯이 `memory-core`로 초기화됩니다.

기본적으로 uninstall은 활성 상태 디렉토리 확장 루트 아래의 플러그인 설치 디렉토리도
제거합니다 (`$OPENCLAW_STATE_DIR/extensions/<id>`).
파일을 디스크에 유지하려면 `--keep-files`를 사용하세요.

`--keep-config`는 `--keep-files`의 더 이상 사용되지 않는 별칭으로 지원됩니다.

### 업데이트

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

업데이트는 npm에서 설치된 플러그인(`plugins.installs`에서 추적)에만 적용됩니다.

저장된 무결성 해시가 존재하고 가져온 아티팩트 해시가 변경된 경우,
OpenClaw는 경고를 출력하고 진행하기 전에 확인을 요청합니다.
CI/비인터랙티브 실행에서 프롬프트를 우회하려면 전역 `--yes`를 사용하세요.
