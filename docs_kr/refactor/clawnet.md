---
summary: "Clawnet 리팩토링: 네트워크 프로토콜, 역할, 인증, 승인, 신원 통합"
read_when:
  - Planning a unified network protocol for nodes + operator clients
  - Reworking approvals, pairing, TLS, and presence across devices
title: "Clawnet 리팩토링"
---

# Clawnet 리팩토링 (프로토콜 + 인증 통합)

## 인사

안녕하세요 Peter - 좋은 방향입니다; 이것이 더 간단한 UX + 더 강력한 보안을 가능하게 합니다.

## 목적

단일하고 엄격한 문서:

- 현재 상태: 프로토콜, 흐름, 신뢰 경계.
- 문제점: 승인, 다중 홉 라우팅, UI 중복.
- 제안된 새 상태: 하나의 프로토콜, 범위 지정 역할, 통합 인증/페어링, TLS 고정.
- 신원 모델: 안정적 ID + 귀여운 slug.
- 마이그레이션 계획, 위험, 미해결 질문.

## 목표 (논의에서)

- 모든 클라이언트를 위한 하나의 프로토콜 (mac app, CLI, iOS, Android, headless node).
- 모든 네트워크 참가자가 인증 + 페어링됨.
- 역할 명확성: node vs operator.
- 사용자가 있는 곳으로 라우팅되는 중앙 승인.
- 모든 원격 트래픽에 대한 TLS 암호화 + 선택적 고정.
- 최소한의 코드 중복.
- 단일 머신이 한 번만 나타나야 함 (UI/node 중복 항목 없음).

## 비목표 (명시적)

- 기능 분리 제거 (여전히 최소 권한 필요).
- 범위 검사 없이 전체 gateway 제어 플레인 노출.
- 인증을 인간 레이블에 의존하게 만들기 (slug는 비보안으로 유지).

---

# 현재 상태 (있는 그대로)

## 두 개의 프로토콜

### 1) Gateway WebSocket (제어 플레인)

- 전체 API 표면: config, channels, models, sessions, agent runs, logs, nodes 등.
- 기본 바인딩: loopback. SSH/Tailscale를 통한 원격 접근.
- 인증: `connect`를 통한 token/password.
- TLS 고정 없음 (loopback/터널에 의존).
- 코드:
  - `src/gateway/server/ws-connection/message-handler.ts`
  - `src/gateway/client.ts`
  - `docs/gateway/protocol.md`

### 2) Bridge (node 전송)

- 좁은 허용 목록 표면, node 신원 + 페어링.
- TCP를 통한 JSONL; 선택적 TLS + cert 핑거프린트 고정.
- TLS는 discovery TXT에서 핑거프린트를 알림.
- 코드:
  - `src/infra/bridge/server/connection.ts`
  - `src/gateway/server-bridge.ts`
  - `src/node-host/bridge-client.ts`
  - `docs/gateway/bridge-protocol.md`

## 현재 제어 플레인 클라이언트

- CLI -> `callGateway`를 통한 Gateway WS (`src/gateway/call.ts`).
- macOS app UI -> Gateway WS (`GatewayConnection`).
- Web Control UI -> Gateway WS.
- ACP -> Gateway WS.
- 브라우저 제어는 자체 HTTP 제어 서버를 사용.

## 현재 Node

- node 모드의 macOS app이 Gateway bridge에 연결 (`MacNodeBridgeSession`).
- iOS/Android 앱이 Gateway bridge에 연결.
- 페어링 + node별 토큰이 gateway에 저장됨.

## 현재 승인 흐름 (exec)

- 에이전트가 Gateway를 통해 `system.run`을 사용.
- Gateway가 bridge를 통해 node를 호출.
- Node 런타임이 승인을 결정.
- UI 프롬프트가 mac app에 표시됨 (node == mac app일 때).
- Node가 `invoke-res`를 Gateway에 반환.
- 다중 홉, UI가 node 호스트에 묶임.

## 현재 Presence + 신원

- WS 클라이언트의 Gateway presence 항목.
- bridge의 Node presence 항목.
- mac app이 같은 머신에 대해 두 항목을 표시할 수 있음 (UI + node).
- Node 신원은 페어링 저장소에 저장; UI 신원은 별도.

---

# 문제점 / 고통점

- 유지관리할 두 개의 프로토콜 스택 (WS + Bridge).
- 원격 node의 승인: 프롬프트가 사용자가 있는 곳이 아닌 node 호스트에 나타남.
- TLS 고정은 bridge에만 존재; WS는 SSH/Tailscale에 의존.
- 신원 중복: 같은 머신이 여러 인스턴스로 나타남.
- 모호한 역할: UI + node + CLI 기능이 명확하게 분리되지 않음.

---

# 제안된 새 상태 (Clawnet)

## 하나의 프로토콜, 두 개의 역할

역할 + 범위를 가진 단일 WS 프로토콜.

- **역할: node** (기능 호스트)
- **역할: operator** (제어 플레인)
- operator를 위한 선택적 **범위**:
  - `operator.read` (상태 + 보기)
  - `operator.write` (에이전트 실행, 전송)
  - `operator.admin` (구성, 채널, 모델)

### 역할 동작

**Node**

- 기능 등록 가능 (`caps`, `commands`, permissions).
- `invoke` 명령 수신 가능 (`system.run`, `camera.*`, `canvas.*`, `screen.record` 등).
- 이벤트 전송 가능: `voice.transcript`, `agent.request`, `chat.subscribe`.
- config/models/channels/sessions/agent 제어 플레인 API 호출 불가.

**Operator**

- 범위로 게이팅된 전체 제어 플레인 API.
- 모든 승인 수신.
- OS 작업을 직접 실행하지 않음; node로 라우팅.

### 핵심 규칙

역할은 디바이스가 아닌 연결별입니다. 디바이스는 두 역할을 별도로 열 수 있습니다.

---

# 통합 인증 + 페어링

## 클라이언트 신원

모든 클라이언트가 제공하는 것:

- `deviceId` (안정적, 디바이스 키에서 파생).
- `displayName` (사람이 읽을 수 있는 이름).
- `role` + `scope` + `caps` + `commands`.

## 페어링 흐름 (통합)

- 클라이언트가 비인증 상태로 연결.
- Gateway가 해당 `deviceId`에 대한 **페어링 요청**을 생성.
- Operator가 프롬프트를 받음; 승인/거부.
- Gateway가 다음에 바인딩된 자격 증명 발급:
  - 디바이스 공개 키
  - 역할
  - 범위
  - 기능/명령
- 클라이언트가 토큰을 저장하고, 인증된 상태로 재연결.

## 디바이스 바인딩 인증 (bearer 토큰 재생 방지)

선호: 디바이스 키페어.

- 디바이스가 한 번 키페어를 생성.
- `deviceId = fingerprint(publicKey)`.
- Gateway가 nonce를 전송; 디바이스가 서명; gateway가 검증.
- 토큰은 문자열이 아닌 공개 키(소유 증명)에 발급됨.

대안:

- mTLS (클라이언트 인증서): 가장 강력하지만, 운영 복잡성이 높음.
- 단기 bearer 토큰은 임시 단계로만 (조기 순환 + 철회).

## 자동 승인 (SSH 휴리스틱)

약한 고리를 피하기 위해 정확하게 정의. 하나를 선호:

- **로컬 전용**: 클라이언트가 loopback/Unix 소켓을 통해 연결할 때 자동 페어링.
- **SSH를 통한 챌린지**: gateway가 nonce를 발급; 클라이언트가 SSH로 이를 가져와 증명.
- **물리적 존재 윈도우**: gateway 호스트 UI에서 로컬 승인 후, 짧은 윈도우(예: 10분) 동안 자동 페어링 허용.

항상 자동 승인을 로깅 + 기록.

---

# 모든 곳에서 TLS (개발 + 운영)

## 기존 bridge TLS 재사용

현재 TLS 런타임 + 핑거프린트 고정 사용:

- `src/infra/bridge/server/tls.ts`
- `src/node-host/bridge-client.ts`의 핑거프린트 검증 로직

## WS에 적용

- WS 서버가 동일한 cert/key + 핑거프린트로 TLS 지원.
- WS 클라이언트가 핑거프린트를 고정할 수 있음 (선택적).
- Discovery가 모든 엔드포인트에 대해 TLS + 핑거프린트를 알림.
  - Discovery는 위치 힌트일 뿐; 신뢰 앵커가 아님.

## 이유

- 기밀성을 위한 SSH/Tailscale 의존도 감소.
- 원격 모바일 연결을 기본적으로 안전하게 만듦.

---

# 승인 재설계 (중앙화)

## 현재

승인이 node 호스트에서 발생 (mac app node 런타임). 프롬프트가 node가 실행되는 곳에 나타남.

## 제안

승인이 **gateway에서 호스팅**되고, UI가 operator 클라이언트에 전달됨.

### 새로운 흐름

1. Gateway가 `system.run` 의도를 수신 (에이전트).
2. Gateway가 승인 레코드 생성: `approval.requested`.
3. Operator UI가 프롬프트를 표시.
4. 승인 결정이 gateway로 전송: `approval.resolve`.
5. 승인되면 Gateway가 node 명령을 호출.
6. Node가 실행하고, `invoke-res`를 반환.

### 승인 의미론 (강화)

- 모든 operator에게 브로드캐스트; 활성 UI만 모달을 표시 (다른 것은 토스트를 받음).
- 첫 번째 해결이 승리; gateway가 이미 해결된 후속 해결을 거부.
- 기본 타임아웃: N초 후 거부 (예: 60초), 이유를 로깅.
- 해결에는 `operator.approvals` 범위 필요.

## 이점

- 프롬프트가 사용자가 있는 곳에 나타남 (mac/phone).
- 원격 node에 대한 일관된 승인.
- Node 런타임이 headless로 유지; UI 의존성 없음.

---

# 역할 명확성 예시

## iPhone 앱

- **Node 역할**: mic, camera, voice chat, location, push-to-talk.
- 선택적 **operator.read**: 상태 및 채팅 보기.
- 선택적 **operator.write/admin**: 명시적으로 활성화된 경우에만.

## macOS 앱

- 기본적으로 Operator 역할 (제어 UI).
- "Mac node"가 활성화되면 Node 역할 (system.run, screen, camera).
- 두 연결에 동일한 deviceId -> 병합된 UI 항목.

## CLI

- 항상 Operator 역할.
- 하위 명령에 따라 범위 파생:
  - `status`, `logs` -> read
  - `agent`, `message` -> write
  - `config`, `channels` -> admin
  - approvals + pairing -> `operator.approvals` / `operator.pairing`

---

# 신원 + slug

## 안정적 ID

인증에 필수; 변경 불가.
선호:

- 키페어 핑거프린트 (공개 키 해시).

## 귀여운 slug (랍스터 테마)

사람이 읽는 레이블 전용.

- 예: `scarlet-claw`, `saltwave`, `mantis-pinch`.
- gateway 레지스트리에 저장, 편집 가능.
- 충돌 처리: `-2`, `-3`.

## UI 그룹화

역할 간 동일한 `deviceId` -> 단일 "인스턴스" 행:

- 배지: `operator`, `node`.
- 기능 + 마지막 확인 표시.

---

# 마이그레이션 전략

## Phase 0: 문서화 + 정렬

- 이 문서 게시.
- 모든 프로토콜 호출 + 승인 흐름 인벤토리.

## Phase 1: WS에 역할/범위 추가

- `connect` 파라미터에 `role`, `scope`, `deviceId` 확장.
- node 역할에 대한 허용 목록 게이팅 추가.

## Phase 2: Bridge 호환성

- bridge 계속 실행.
- WS node 지원을 병렬로 추가.
- config 플래그 뒤에서 기능 게이팅.

## Phase 3: 중앙 승인

- WS에 승인 요청 + 해결 이벤트 추가.
- mac app UI를 프롬프트 + 응답하도록 업데이트.
- Node 런타임이 UI 프롬프팅 중지.

## Phase 4: TLS 통합

- bridge TLS 런타임을 사용하여 WS에 TLS 구성 추가.
- 클라이언트에 고정 추가.

## Phase 5: bridge 지원 중단

- iOS/Android/mac node를 WS로 마이그레이션.
- bridge를 폴백으로 유지; 안정화 후 제거.

## Phase 6: 디바이스 바인딩 인증

- 모든 비로컬 연결에 키 기반 신원 필수.
- 철회 + 순환 UI 추가.

---

# 보안 참고 사항

- 역할/허용 목록이 gateway 경계에서 시행됨.
- operator 범위 없이는 "전체" API를 얻는 클라이언트 없음.
- _모든_ 연결에 페어링 필수.
- TLS + 고정이 모바일의 MITM 위험을 줄임.
- SSH 자동 승인은 편의 기능; 여전히 기록 + 철회 가능.
- Discovery는 절대 신뢰 앵커가 아님.
- 기능 주장은 플랫폼/유형별 서버 허용 목록에 대해 검증됨.

# 스트리밍 + 대용량 페이로드 (node 미디어)

WS 제어 플레인은 작은 메시지에 적합하지만, node는 다음도 수행합니다:

- 카메라 클립
- 화면 녹화
- 오디오 스트림

옵션:

1. WS 바이너리 프레임 + 청킹 + 백프레셔 규칙.
2. 별도의 스트리밍 엔드포인트 (여전히 TLS + 인증).
3. 미디어가 많은 명령을 위해 bridge를 더 오래 유지하고, 마지막에 마이그레이션.

드리프트를 피하기 위해 구현 전에 하나를 선택.

# 기능 + 명령 정책

- Node가 보고한 caps/commands는 **주장**으로 취급됨.
- Gateway가 플랫폼별 허용 목록을 시행.
- 새로운 명령은 operator 승인이나 명시적 허용 목록 변경 필요.
- 타임스탬프와 함께 변경 사항 감사.

# 감사 + rate limiting

- 로그: 페어링 요청, 승인/거부, 토큰 발급/순환/철회.
- 페어링 스팸과 승인 프롬프트에 rate-limit.

# 프로토콜 위생

- 명시적 프로토콜 버전 + 오류 코드.
- 재연결 규칙 + 하트비트 정책.
- Presence TTL 및 마지막 확인 의미론.

---

# 미해결 질문

1. 두 역할을 모두 실행하는 단일 디바이스: 토큰 모델
   - 역할별(node vs operator) 별도 토큰 권장.
   - 동일한 deviceId; 다른 범위; 더 명확한 철회.

2. Operator 범위 세분화
   - read/write/admin + approvals + pairing (최소 실행 가능).
   - 나중에 기능별 범위 고려.

3. 토큰 순환 + 철회 UX
   - 역할 변경 시 자동 순환.
   - deviceId + 역할별 철회 UI.

4. Discovery
   - 현재 Bonjour TXT를 WS TLS 핑거프린트 + 역할 힌트를 포함하도록 확장.
   - 위치 힌트로만 취급.

5. 교차 네트워크 승인
   - 모든 operator 클라이언트에 브로드캐스트; 활성 UI가 모달 표시.
   - 첫 번째 응답이 승리; gateway가 원자성을 시행.

---

# 요약 (TL;DR)

- 현재: WS 제어 플레인 + Bridge node 전송.
- 문제: 승인 + 중복 + 두 개의 스택.
- 제안: 명시적 역할 + 범위를 가진 하나의 WS 프로토콜, 통합 페어링 + TLS 고정, gateway 호스팅 승인, 안정적 디바이스 ID + 귀여운 slug.
- 결과: 더 간단한 UX, 더 강력한 보안, 적은 중복, 더 나은 모바일 라우팅.
