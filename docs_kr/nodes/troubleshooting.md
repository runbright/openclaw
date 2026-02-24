---
summary: "노드 작업 중 발생하는 페어링 문제, 권한 부족 및 도구 실행 실패 해결 가이드"
read_when:
  - 노드가 연결은 되어 있으나 카메라/화면/명령어 실행 도구가 작동하지 않을 때
  - 노드 승인과 실행 권한의 차이를 이해하고 싶을 때
title: "노드 문제 해결"
---

# 노드 문제 해결 (Node Troubleshooting)

노드가 상태창에는 보이지만 실제 도구(카메라, 화면 캡처 등)가 작동하지 않을 때 확인해야 할 사항들입니다.

## 원인 진단 명령어

터미널에서 다음 명령어들을 순서대로 실행해 보세요:

```bash
openclaw nodes status         # 노드 연결 상태 확인
openclaw nodes describe --node <ID>  # 노드 상세 정보 및 기능 확인
openclaw approvals get --node <ID>   # 명령어 실행 승인 상태 확인
openclaw logs --follow        # 실시간 로그 확인
```

## 주요 체크포인트

### 1. 앱이 화면에 떠 있나요? (Foreground)
iOS와 Android 노드의 경우, **OpenClaw 앱이 화면에 띄워져 있는 상태**에서만 카메라와 화면 캡처 기능을 사용할 수 있습니다.
- **오류 코드**: `NODE_BACKGROUND_UNAVAILABLE`
- **해결 방법**: 휴대폰에서 앱을 열고 화면을 켠 상태로 유지하세요.

### 2. 권한을 허용했나요? (Permissions)
기능별로 OS 레벨의 권한 허용이 필요합니다.
- **카메라**: 사진/영상 촬영
- **위치**: GPS 정보 가져오기
- **화면 기록**: 스크린샷 및 화면 녹화
- **오류 코드**: `*_PERMISSION_REQUIRED`

### 3. 기기 승인 vs 명령어 승인
두 가지 승인 단계가 다릅니다:
1. **기기 승인 (Device Pairing)**: 이 기기가 내 게이트웨이에 접속해도 되는가?
2. **명령어 승인 (Exec Approvals)**: 이 기기가 특정 터미널 명령어를 실행해도 되는가?

기기는 연결되었는데 명령어가 실행되지 않는다면 `openclaw approvals` 설정을 확인해야 합니다.

## 자주 발생하는 오류 코드

- `SYSTEM_RUN_DENIED: approval required`: 명령어 실행을 위해 사용자의 명시적인 승인이 필요합니다.
- `SYSTEM_RUN_DENIED: allowlist miss`: 실행하려는 명령어가 승인 목록(Allowlist)에 없습니다.
- `LOCATION_DISABLED`: 노드 설정에서 위치 정보 공유가 꺼져 있습니다.

---
추가적인 정보는 [노드 메인 가이드](/nodes/index)를 참고하세요.
---
