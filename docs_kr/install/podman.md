---
summary: "루트리스 Podman 컨테이너에서 OpenClaw 실행"
read_when:
  - Docker 대신 Podman으로 컨테이너화된 게이트웨이를 원할 때
title: "Podman"
---

# Podman

**루트리스** Podman 컨테이너에서 OpenClaw 게이트웨이를 실행합니다. Docker와 동일한 이미지를 사용합니다 (저장소의 [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)에서 빌드).

## 요구 사항

- Podman (루트리스)
- 일회성 설정을 위한 Sudo (사용자 생성, 이미지 빌드)

## 빠른 시작

**1. 일회성 설정** (저장소 루트에서; 사용자 생성, 이미지 빌드, 실행 스크립트 설치):

```bash
./setup-podman.sh
```

게이트웨이가 마법사 없이 시작할 수 있도록 최소한의 `~openclaw/.openclaw/openclaw.json` (`gateway.mode="local"` 설정)도 생성합니다.

기본적으로 컨테이너는 systemd 서비스로 설치되지 **않으며**, 수동으로 시작합니다 (아래 참조). 자동 시작 및 재시작이 되는 프로덕션 스타일 설정을 위해 systemd Quadlet 사용자 서비스로 설치하세요:

```bash
./setup-podman.sh --quadlet
```

(`OPENCLAW_PODMAN_QUADLET=1`을 설정해도 됩니다; 컨테이너와 실행 스크립트만 설치하려면 `--container`를 사용하세요.)

**2. 게이트웨이 시작** (수동, 빠른 스모크 테스트용):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. 온보딩 마법사** (예: 채널이나 프로바이더 추가):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

그런 다음 `http://127.0.0.1:18789/`를 열고 `~openclaw/.openclaw/.env`의 토큰 (또는 설정에서 출력된 값)을 사용하세요.

## Systemd (Quadlet, 선택 사항)

`./setup-podman.sh --quadlet` (또는 `OPENCLAW_PODMAN_QUADLET=1`)을 실행한 경우, [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) 유닛이 설치되어 게이트웨이가 openclaw 사용자의 systemd 사용자 서비스로 실행됩니다. 서비스는 설정 마지막에 활성화되고 시작됩니다.

- **시작:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **중지:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **상태:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **로그:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

Quadlet 파일은 `~openclaw/.config/containers/systemd/openclaw.container`에 있습니다. 포트나 환경 변수를 변경하려면 해당 파일 (또는 참조하는 `.env`)을 편집한 후 `sudo systemctl --machine openclaw@ --user daemon-reload`하고 서비스를 재시작하세요. 부팅 시 openclaw에 대해 lingering이 활성화되어 있으면 서비스가 자동으로 시작됩니다 (loginctl이 사용 가능한 경우 설정에서 이를 수행합니다).

초기 설정에서 quadlet을 사용하지 않은 후에 추가하려면 다시 실행하세요: `./setup-podman.sh --quadlet`.

## openclaw 사용자 (비로그인)

`setup-podman.sh`는 전용 시스템 사용자 `openclaw`을 생성합니다:

- **셸:** `nologin` — 대화형 로그인 없음; 공격 표면을 줄입니다.
- **홈:** 예: `/home/openclaw` — `~/.openclaw` (설정, 워크스페이스)과 실행 스크립트 `run-openclaw-podman.sh`를 보관합니다.
- **루트리스 Podman:** 사용자에게 **subuid**와 **subgid** 범위가 있어야 합니다. 많은 배포판에서 사용자 생성 시 자동으로 할당합니다. 설정에서 경고가 표시되면 `/etc/subuid`와 `/etc/subgid`에 줄을 추가하세요:

  ```text
  openclaw:100000:65536
  ```

  그런 다음 해당 사용자로 게이트웨이를 시작하세요 (예: cron이나 systemd에서):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **설정:** `openclaw`과 root만 `/home/openclaw/.openclaw`에 접근할 수 있습니다. 설정을 편집하려면: 게이트웨이가 실행 중이면 Control UI를 사용하거나, `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`을 사용하세요.

## 환경 및 설정

- **토큰:** `~openclaw/.openclaw/.env`에 `OPENCLAW_GATEWAY_TOKEN`으로 저장됩니다. `setup-podman.sh`와 `run-openclaw-podman.sh`가 없으면 생성합니다 (`openssl`, `python3`, 또는 `od` 사용).
- **선택 사항:** 해당 `.env`에 프로바이더 키 (예: `GROQ_API_KEY`, `OLLAMA_API_KEY`)와 기타 OpenClaw 환경 변수를 설정할 수 있습니다.
- **호스트 포트:** 기본적으로 스크립트가 `18789` (게이트웨이)와 `18790` (브릿지)를 매핑합니다. 실행 시 `OPENCLAW_PODMAN_GATEWAY_HOST_PORT`와 `OPENCLAW_PODMAN_BRIDGE_HOST_PORT`로 **호스트** 포트 매핑을 재정의하세요.
- **경로:** 호스트 설정과 워크스페이스는 기본적으로 `~openclaw/.openclaw`과 `~openclaw/.openclaw/workspace`입니다. 실행 스크립트에서 사용하는 호스트 경로를 `OPENCLAW_CONFIG_DIR`과 `OPENCLAW_WORKSPACE_DIR`로 재정의하세요.

## 유용한 명령어

- **로그:** Quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. 스크립트: `sudo -u openclaw podman logs -f openclaw`
- **중지:** Quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. 스크립트: `sudo -u openclaw podman stop openclaw`
- **다시 시작:** Quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. 스크립트: 실행 스크립트를 다시 실행하거나 `podman start openclaw`
- **컨테이너 제거:** `sudo -u openclaw podman rm -f openclaw` — 호스트의 설정과 워크스페이스는 유지됩니다

## 문제 해결

- **설정 또는 인증 프로필에 권한 거부 (EACCES):** 컨테이너는 기본적으로 `--userns=keep-id`를 사용하고 스크립트를 실행하는 호스트 사용자와 동일한 uid/gid로 실행됩니다. 호스트의 `OPENCLAW_CONFIG_DIR`과 `OPENCLAW_WORKSPACE_DIR`이 해당 사용자 소유인지 확인하세요.
- **게이트웨이 시작 차단 (`gateway.mode=local` 누락):** `~openclaw/.openclaw/openclaw.json`이 존재하고 `gateway.mode="local"`을 설정하는지 확인하세요. `setup-podman.sh`는 파일이 없으면 이를 생성합니다.
- **openclaw 사용자에 대한 루트리스 Podman 실패:** `/etc/subuid`와 `/etc/subgid`에 `openclaw` 줄이 포함되어 있는지 확인하세요 (예: `openclaw:100000:65536`). 없으면 추가하고 재시작하세요.
- **컨테이너 이름 사용 중:** 실행 스크립트는 `podman run --replace`를 사용하므로 다시 시작할 때 기존 컨테이너가 대체됩니다. 수동 정리: `podman rm -f openclaw`.
- **openclaw으로 실행 시 스크립트를 찾을 수 없음:** `setup-podman.sh`가 실행되어 `run-openclaw-podman.sh`가 openclaw 홈에 복사되었는지 확인하세요 (예: `/home/openclaw/run-openclaw-podman.sh`).
- **Quadlet 서비스를 찾을 수 없거나 시작 실패:** `.container` 파일을 편집한 후 `sudo systemctl --machine openclaw@ --user daemon-reload`를 실행하세요. Quadlet에는 cgroups v2가 필요합니다: `podman info --format '{{.Host.CgroupsVersion}}'`가 `2`를 표시해야 합니다.

## 선택 사항: 자신의 사용자로 실행

게이트웨이를 일반 사용자로 실행하려면 (전용 openclaw 사용자 없이): 이미지를 빌드하고, `OPENCLAW_GATEWAY_TOKEN`이 포함된 `~/.openclaw/.env`를 생성하고, `--userns=keep-id`와 `~/.openclaw` 마운트로 컨테이너를 실행하세요. 실행 스크립트는 openclaw 사용자 흐름을 위해 설계되었습니다; 단일 사용자 설정의 경우 스크립트의 `podman run` 명령을 수동으로 실행하여 설정과 워크스페이스를 홈으로 지정할 수 있습니다. 대부분의 사용자에게 권장: `setup-podman.sh`를 사용하고 openclaw 사용자로 실행하여 설정과 프로세스를 격리하세요.
