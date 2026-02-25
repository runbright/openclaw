# OpenClaw 노드 (Android) (내부 테스트용)

이 앱은 최신형 Android 노드 앱으로, **게이트웨이 WebSocket**에 연결되어 **캔버스, 채팅, 카메라** 기능을 제공합니다.

## 주요 특징
- **포그라운드 서비스**: 연결을 유지하기 위해 지속적인 알림과 함께 포그라운드 서비스를 실행합니다.
- **공유 세션**: 채팅은 항상 공유 세션 키인 `main`을 사용하며, iOS/macOS/웹/Android 간에 대화 내역이 공유됩니다.
- **기술 스택**: 현대적인 Android 환경(`minSdk 31`, Kotlin + Jetpack Compose)을 지원합니다.

## 빌드 및 실행
```bash
cd apps/android
./gradlew :app:assembleDebug   # 빌드
./gradlew :app:installDebug    # 설치 및 디버그 실행
```

## 연결 및 페어링
1. 메인 컴퓨터에서 **게이트웨이**를 실행합니다.
2. Android 앱의 **Settings(설정)**에서 자동 탐지된 게이트웨이를 선택하거나, **Advanced(고급)** 메뉴에서 수동으로 호스트와 포트를 입력합니다.
3. 메인 컴퓨터에서 페어링을 승인합니다:
   ```bash
   openclaw nodes pending     # 보류 중인 요청 확인
   openclaw nodes approve <요청ID>  # 승인
   ```

## 권한 안내
- **탐지**: 기기 탐지를 위해 `NEARBY_WIFI_DEVICES` 또는 `ACCESS_FINE_LOCATION` 권한이 필요합니다.
- **포그라운드 서비스**: 안드로이드 13 이상에서 알림 권한이 필요합니다.
- **카메라/오디오**: 원격 촬영 및 동영상 기능을 위해 `CAMERA` 및 `RECORD_AUDIO` 권한이 필요합니다.

---
상세 내용은 `docs/platforms/android.md`(영문)를 참조하세요.
---
