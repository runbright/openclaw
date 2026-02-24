---
summary: "OpenClaw를 위한 VPS 호스팅 안내 (Oracle/Fly/Hetzner/GCP 등)"
read_when:
  - 게이트웨이를 클라우드 환경에서 실행하고 싶을 때
  - VPS/호스팅 가이드 목록이 필요할 때
title: "VPS 호스팅"
---

# VPS 호스팅 (VPS Hosting)

이 문서는 OpenClaw를 설치할 수 있는 주요 VPS/호스팅 가이드를 모아놓은 허브입니다.

## 호스팅 업체 선택

- **Railway** (원클릭 설치 지원): [Railway 가이드](/install/railway)
- **Northflank** (원클릭 설치 지원): [Northflank 가이드](/install/northflank)
- **Oracle Cloud (Always Free)**: [Oracle 가이드](/platforms/oracle) — 무료 티어(ARM)를 통해 비용 없이 운영 가능합니다. (가입 및 자원 확보가 다소 어려울 수 있음)
- **Fly.io**: [Fly.io 가이드](/install/fly)
- **Hetzner (Docker)**: [Hetzner 가이드](/install/hetzner)
- **GCP (Compute Engine)**: [GCP 가이드](/install/gcp)
- **exe.dev** (VM + HTTPS 프록시): [exe.dev 가이드](/install/exe-dev)
- **AWS (EC2/Lightsail)**: 무료 티어로 잘 작동합니다. [관련 영상 가이드](https://x.com/techfrenAJ/status/2014934471095812547)

## 클라우드 설정 방식

- **게이트웨이는 VPS에서 실행**되며, 모든 상태 정보와 워크스페이스를 관리합니다.
- 사용자는 자신의 노트북이나 스마트폰에서 **제어 UI** 또는 **Tailscale/SSH**를 통해 접속합니다.
- VPS를 데이터의 핵심으로 간주하고, 상태 정보와 워크스페이스를 정기적으로 **백업**하십시오.
- **보안 설정**: 게이트웨이를 루프백(`loopback`)으로 유지하고 SSH 터널이나 Tailscale Serve를 통해 접속하는 것이 가장 안전합니다. 외부 노출 시에는 반드시 강력한 토큰이나 비밀번호를 설정하십시오.

상세 원격 접속 방법: [게이트웨이 원격 접속](/gateway/remote)

## VPS와 노드 함께 사용하기

게이트웨이는 클라우드(VPS)에 두면서, 여러분의 기기(Mac/iOS/Android)를 **노드(Node)**로 페어링할 수 있습니다. 이렇게 하면 클라우드에 있는 게이트웨이가 로컬 기기의 화면, 카메라, 캔버스 기능을 제어할 수 있게 됩니다.

관련 도움말: [노드 개요](/nodes), [노드 CLI](/cli/nodes)
---
