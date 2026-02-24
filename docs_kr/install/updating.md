---
summary: "OpenClaw를 안전하게 최신 버전으로 업데이트하거나 이전 버전으로 되돌리는 방법"
read_when:
  - OpenClaw를 최신 기능을 사용하기 위해 업데이트하고 싶을 때
  - 업데이트 후 문제가 발생하여 이전 상태로 복구해야 할 때
title: "업데이트"
---

# 업데이트 (Updating)

OpenClaw는 매우 빠르게 발전하고 있습니다. 최신 기능과 보안 패치를 적용하기 위해 주기적으로 업데이트하는 것이 좋습니다.

## 추천 방법: 설치 스크립트 재실행

가장 쉽고 안전한 방법은 처음 설치할 때 사용했던 스크립트를 다시 실행하는 것입니다. 기존 설정을 유지하면서 프로그램만 최신 버전으로 교체합니다.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

*   **초기 설정 건너뛰기**: 이미 설정을 완료했다면 뒤에 `--no-onboard` 플래그를 붙여서 업데이트만 진행할 수 있습니다.

## 명령어로 업데이트하기 (`openclaw update`)

터미널에서 직접 명령어를 입력해 업데이트할 수도 있습니다.

```bash
openclaw update
```

이 명령어는 현재 설치된 방식(npm 또는 소스 코드)을 감지하여 적절한 업데이트 절차를 수행하고 게이트웨이를 자동으로 재시작합니다.

### 업데이트 채널 변경
- **안정 버전 (Stable)**: `openclaw update --channel stable`
- **베타 버전 (Beta)**: `openclaw update --channel beta`
- **개발 버전 (Dev)**: `openclaw update --channel dev`

## 업데이트 후 필수 단계: `openclaw doctor`

버전이 올라가면 설정 파일의 구조가 바뀌거나 새로운 설정이 필요할 수 있습니다. 업데이트 후에는 반드시 다음 명령어를 실행하여 시스템을 점검하세요.

```bash
openclaw doctor
```

이 명령어는 잘못된 설정을 수정하고 필요한 데이터 마이그레이션을 자동으로 진행합니다.

## 문제가 생겼을 때 (되돌리기)

업데이트 후 시스템이 불안정하다면 특정 버전으로 되돌릴 수 있습니다.

**npm 설치 시:**
```bash
npm install -g openclaw@<버전번호>
```

**소스 코드 사용 시:**
문제가 없던 시점의 날짜로 코드를 되돌립니다.
```bash
git checkout "$(git rev-list -n 1 --before=\"YYYY-MM-DD\" origin/main)"
pnpm install && pnpm build
```

---
상세한 문제 해결 방법은 [문제 해결 가이드](/help/troubleshooting)를 참고하세요.
---
