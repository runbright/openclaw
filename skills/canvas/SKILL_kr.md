# 캔버스(Canvas) 스킬

연결된 OpenClaw 노드(Mac 앱, iOS, Android)에 HTML 콘텐츠를 표시합니다.

## 개요
캔버스 도구는 연결된 노드의 화면에 웹 콘텐츠를 보여줍니다. 다음과 같은 경우에 유용합니다:
- 게임, 데이터 시각화, 대시보드 표시
- 에이전트가 생성한 HTML 콘텐츠 보여주기
- 대화형 데모 실행

## 주요 기능 (액션)
- `present`: 지정된 URL의 캔버스 표시
- `hide`: 캔버스 숨기기
- `navigate`: 새로운 URL로 이동
- `eval`: 캔버스 내에서 JavaScript 실행
- `snapshot`: 캔버스 화면 캡처

## 워크플로우 예시

### 1. HTML 콘텐츠 만들기
캔버스 루트 디렉토리(기본값 `~/clawd/canvas/`)에 파일을 저장합니다:
```bash
cat > ~/clawd/canvas/hello.html << 'HTML'
<!DOCTYPE html>
<html>
<body><h1>Hello OpenClaw Canvas!</h1></body>
</html>
HTML
```

### 2. 콘텐츠 표시하기
명령어를 통해 특정 기기(노드)에 화면을 띄웁니다:
```bash
# 특정 노드에 hello.html 표시
canvas action:present node:<기기ID> target:http://<호스트주소>:18793/__openclaw__/canvas/hello.html
```

## 디버깅 팁
- **화면이 하얗게 나오나요?**: 서버의 바인딩 주소(`127.0.0.1` vs `LAN IP`)와 노드가 접근하려는 주소가 일치하는지 확인하세요. 가급적 외부 노드(iOS/Android)에서는 `localhost` 대신 장치의 실제 IP나 Tailscale 주소를 사용해야 합니다.
- **실시간 반영 (Live Reload)**: 설정에서 `liveReload: true`인 경우, 파일을 수정하고 저장하면 연결된 화면이 즉시 업데이트됩니다.

---
상세한 네트워크 구성 및 고급 설정은 영문 문서를 참조하세요.
---
