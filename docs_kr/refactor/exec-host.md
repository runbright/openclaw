---
summary: "리팩토링 계획: exec 호스트 라우팅, node 승인 및 headless 러너"
read_when:
  - Designing exec host routing or exec approvals
  - Implementing node runner + UI IPC
  - Adding exec host security modes and slash commands
title: "Exec 호스트 리팩토링"
---

# Exec 호스트 리팩토링 계획

## 목표

- **sandbox**, **gateway**, **node** 간 실행을 라우팅하기 위해 `exec.host` + `exec.security` 추가.
- 기본값을 **안전하게** 유지: 명시적으로 활성화하지 않는 한 교차 호스트 실행 없음.
- 실행을 선택적 UI(macOS 앱)가 있는 **headless 러너 서비스**로 분리하고 로컬 IPC를 통해 연결.
- **에이전트별** 정책, 허용 목록, ask 모드 및 node 바인딩 제공.
- 허용 목록과 함께 또는 없이 작동하는 **ask 모드** 지원.
- 크로스 플랫폼: Unix 소켓 + 토큰 인증 (macOS/Linux/Windows 동등).

## 비목표

- 레거시 허용 목록 마이그레이션이나 레거시 스키마 지원 없음.
- node exec를 위한 PTY/스트리밍 없음 (집계된 출력만).
- 기존 Bridge + Gateway를 넘어선 새 네트워크 레이어 없음.

## 결정 (확정)

- **구성 키:** `exec.host` + `exec.security` (에이전트별 오버라이드 허용).
- **Elevation:** `/elevated`을 gateway 전체 접근의 별칭으로 유지.
- **Ask 기본값:** `on-miss`.
- **승인 저장소:** `~/.openclaw/exec-approvals.json` (JSON, 레거시 마이그레이션 없음).
- **러너:** headless 시스템 서비스; UI 앱이 승인을 위한 Unix 소켓 호스팅.
- **Node 신원:** 기존 `nodeId` 사용.
- **소켓 인증:** Unix 소켓 + 토큰 (크로스 플랫폼); 나중에 필요시 분리.
- **Node 호스트 상태:** `~/.openclaw/node.json` (node id + 페어링 토큰).
- **macOS exec 호스트:** macOS 앱 내에서 `system.run` 실행; node 호스트 서비스가 로컬 IPC를 통해 요청 전달.
- **XPC 헬퍼 없음:** Unix 소켓 + 토큰 + peer 검사 유지.

## 핵심 개념

### 호스트

- `sandbox`: Docker exec (현재 동작).
- `gateway`: gateway 호스트에서 exec.
- `node`: Bridge를 통한 node 러너에서 exec (`system.run`).

### 보안 모드

- `deny`: 항상 차단.
- `allowlist`: 일치하는 것만 허용.
- `full`: 모든 것 허용 (elevated와 동등).

### Ask 모드

- `off`: 절대 묻지 않음.
- `on-miss`: 허용 목록이 일치하지 않을 때만 물음.
- `always`: 매번 물음.

Ask는 허용 목록과 **독립적**; 허용 목록은 `always` 또는 `on-miss`와 함께 사용 가능.

### 정책 해결 (exec별)

1. `exec.host` 해결 (도구 파라미터 -> 에이전트 오버라이드 -> 전역 기본값).
2. `exec.security` 및 `exec.ask` 해결 (동일한 우선순위).
3. 호스트가 `sandbox`이면 로컬 샌드박스 exec로 진행.
4. 호스트가 `gateway` 또는 `node`이면 해당 호스트에서 보안 + ask 정책 적용.

## 기본 안전성

- 기본 `exec.host = sandbox`.
- `gateway` 및 `node`에 대해 기본 `exec.security = deny`.
- 기본 `exec.ask = on-miss` (보안이 허용하는 경우에만 관련).
- node 바인딩이 설정되지 않으면, **에이전트가 어떤 node든 대상으로 할 수 있음**, 단 정책이 허용하는 경우에만.

## 구성 표면

### 도구 파라미터

- `exec.host` (선택적): `sandbox | gateway | node`.
- `exec.security` (선택적): `deny | allowlist | full`.
- `exec.ask` (선택적): `off | on-miss | always`.
- `exec.node` (선택적): `host=node`일 때 사용할 node id/이름.

### 구성 키 (전역)

- `tools.exec.host`
- `tools.exec.security`
- `tools.exec.ask`
- `tools.exec.node` (기본 node 바인딩)

### 구성 키 (에이전트별)

- `agents.list[].tools.exec.host`
- `agents.list[].tools.exec.security`
- `agents.list[].tools.exec.ask`
- `agents.list[].tools.exec.node`

### 별칭

- `/elevated on` = 에이전트 세션에 `tools.exec.host=gateway`, `tools.exec.security=full` 설정.
- `/elevated off` = 에이전트 세션에 이전 exec 설정 복원.

## 승인 저장소 (JSON)

경로: `~/.openclaw/exec-approvals.json`

목적:

- **실행 호스트** (gateway 또는 node 러너)에 대한 로컬 정책 + 허용 목록.
- UI를 사용할 수 없을 때의 Ask 폴백.
- UI 클라이언트를 위한 IPC 자격 증명.

제안된 스키마 (v1):

```json
{
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

참고:

- 레거시 허용 목록 형식 없음.
- `askFallback`은 `ask`가 필요하고 UI에 도달할 수 없을 때만 적용.
- 파일 권한: `0600`.

## 러너 서비스 (headless)

### 역할

- 로컬에서 `exec.security` + `exec.ask` 시행.
- 시스템 명령 실행 및 출력 반환.
- exec 수명 주기에 대한 Bridge 이벤트 전송 (선택적이지만 권장).

### 서비스 수명 주기

- macOS에서 Launchd/daemon; Linux/Windows에서 시스템 서비스.
- 승인 JSON은 실행 호스트에 로컬.
- UI가 로컬 Unix 소켓을 호스팅; 러너가 필요에 따라 연결.

## UI 통합 (macOS 앱)

### IPC

- `~/.openclaw/exec-approvals.sock`의 Unix 소켓 (0600).
- `exec-approvals.json` (0600)에 저장된 토큰.
- Peer 검사: 동일 UID만.
- 챌린지/응답: nonce + HMAC(token, request-hash)으로 재생 방지.
- 짧은 TTL (예: 10초) + 최대 페이로드 + rate limit.

### Ask 흐름 (macOS 앱 exec 호스트)

1. Node 서비스가 gateway로부터 `system.run`을 수신.
2. Node 서비스가 로컬 소켓에 연결하고 프롬프트/exec 요청을 전송.
3. 앱이 peer + 토큰 + HMAC + TTL을 검증한 다음, 필요한 경우 대화 상자 표시.
4. 앱이 UI 컨텍스트에서 명령을 실행하고 출력을 반환.
5. Node 서비스가 gateway에 출력을 반환.

UI가 없는 경우:

- `askFallback` (`deny|allowlist|full`) 적용.

### 다이어그램 (SCI)

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

## Node 신원 + 바인딩

- Bridge 페어링의 기존 `nodeId` 사용.
- 바인딩 모델:
  - `tools.exec.node`가 에이전트를 특정 node로 제한.
  - 미설정 시, 에이전트가 어떤 node든 선택 가능 (정책이 여전히 기본값을 시행).
- Node 선택 해결:
  - `nodeId` 정확 일치
  - `displayName` (정규화됨)
  - `remoteIp`
  - `nodeId` 접두사 (>= 6자)

## 이벤팅

### 누가 이벤트를 보는가

- 시스템 이벤트는 **세션별**이며 다음 프롬프트에서 에이전트에 표시됨.
- gateway 인메모리 큐에 저장 (`enqueueSystemEvent`).

### 이벤트 텍스트

- `Exec started (node=<id>, id=<runId>)`
- `Exec finished (node=<id>, id=<runId>, code=<code>)` + 선택적 출력 tail
- `Exec denied (node=<id>, id=<runId>, <reason>)`

### 전송

옵션 A (권장):

- 러너가 Bridge `event` 프레임 `exec.started` / `exec.finished`를 전송.
- Gateway `handleBridgeEvent`가 이를 `enqueueSystemEvent`에 매핑.

옵션 B:

- Gateway `exec` 도구가 수명 주기를 직접 처리 (동기만).

## Exec 흐름

### Sandbox 호스트

- 기존 `exec` 동작 (Docker 또는 비샌드박스 시 호스트).
- 비샌드박스 모드에서만 PTY 지원.

### Gateway 호스트

- Gateway 프로세스가 자체 머신에서 실행.
- 로컬 `exec-approvals.json` 시행 (security/ask/allowlist).

### Node 호스트

- Gateway가 `system.run`으로 `node.invoke`를 호출.
- 러너가 로컬 승인을 시행.
- 러너가 집계된 stdout/stderr를 반환.
- 시작/완료/거부에 대한 선택적 Bridge 이벤트.

## 출력 상한

- 결합된 stdout+stderr를 **200k**에서 상한; 이벤트를 위해 **tail 20k** 유지.
- 명확한 접미사로 잘림 (예: `"... (truncated)"`).

## 슬래시 명령

- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- 에이전트별, 세션별 오버라이드; config를 통해 저장하지 않는 한 비지속적.
- `/elevated on|off|ask|full`은 `host=gateway security=full`의 단축키로 유지 (`full`은 승인을 건너뜀).

## 크로스 플랫폼 스토리

- 러너 서비스가 이식 가능한 실행 대상.
- UI는 선택적; 없으면 `askFallback` 적용.
- Windows/Linux가 동일한 승인 JSON + 소켓 프로토콜 지원.

## 구현 단계

### Phase 1: 구성 + exec 라우팅

- `exec.host`, `exec.security`, `exec.ask`, `exec.node`에 대한 구성 스키마 추가.
- `exec.host`를 준수하도록 도구 배관 업데이트.
- `/exec` 슬래시 명령 추가 및 `/elevated` 별칭 유지.

### Phase 2: 승인 저장소 + gateway 시행

- `exec-approvals.json` 리더/라이터 구현.
- `gateway` 호스트에 대한 허용 목록 + ask 모드 시행.
- 출력 상한 추가.

### Phase 3: node 러너 시행

- 허용 목록 + ask를 시행하도록 node 러너 업데이트.
- macOS 앱 UI에 Unix 소켓 프롬프트 브리지 추가.
- `askFallback` 연결.

### Phase 4: 이벤트

- exec 수명 주기를 위한 node -> gateway Bridge 이벤트 추가.
- 에이전트 프롬프트를 위해 `enqueueSystemEvent`에 매핑.

### Phase 5: UI 다듬기

- Mac 앱: 허용 목록 편집기, 에이전트별 전환기, ask 정책 UI.
- Node 바인딩 제어 (선택적).

## 테스트 계획

- 단위 테스트: 허용 목록 매칭 (glob + 대소문자 구분 없음).
- 단위 테스트: 정책 해결 우선순위 (도구 파라미터 -> 에이전트 오버라이드 -> 전역).
- 통합 테스트: node 러너 deny/allow/ask 흐름.
- Bridge 이벤트 테스트: node 이벤트 -> 시스템 이벤트 라우팅.

## 미해결 위험

- UI 사용 불가: `askFallback`이 준수되는지 확인.
- 장시간 실행 명령: 타임아웃 + 출력 상한에 의존.
- 다중 node 모호성: node 바인딩이나 명시적 node 파라미터가 없으면 오류.

## 관련 문서

- [Exec 도구](/tools/exec)
- [Exec 승인](/tools/exec-approvals)
- [Nodes](/nodes)
- [Elevated 모드](/tools/elevated)
