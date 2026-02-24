---
summary: "네트워크 허브: 게이트웨이 인터페이스, 페어링, 탐색 및 보안 관련 문서 모음"
read_when:
  - 네트워크 아키텍처 및 보안 개요를 확인하고 싶을 때
  - 로컬 vs 테일넷(Tailnet) 접속 또는 페어링 문제를 디버깅할 때
  - 네트워크 관련 주요 문서 목록이 필요할 때
title: "네트워크"
---

# 네트워크 허브 (Network Hub)

이곳은 OpenClaw가 로컬호스트, LAN, 그리고 테일넷 환경에서 기기들을 연결하고, 페어링하며, 보안을 유지하는 방식에 대한 핵심 문서들을 모아놓은 곳입니다.

## 핵심 모델 (Core Model)

- [게이트웨이 아키텍처](/concepts/architecture)
- [게이트웨이 프로토콜](/gateway/protocol)
- [게이트웨이 가이드](/gateway)
- [웹 인터페이스 및 바인드 모드](/web)

## 페어링 및 아이덴티티

- [페어링 개요 (DM + 노드)](/channels/pairing)
- [게이트웨이 노드 페어링](/gateway/pairing)
- [디바이스 CLI (페어링 + 토큰 교체)](/cli/devices)
- [페어링 CLI (DM 승인)](/cli/pairing)

**로컬 신뢰 원칙:**
- 로컬 연결(루프백 또는 게이트웨이 호스트 자신의 테일넷 주소)은 원활한 사용자 경험을 위해 페어링 승인이 자동화될 수 있습니다.
- 로컬이 아닌 테일넷/LAN 클라이언트는 반드시 명시적인 페어링 승인이 필요합니다.

## 탐색 및 전송 레이어 (Discovery & Transports)

- [기기 탐색 및 전송 방식](/gateway/discovery)
- [Bonjour / mDNS](/gateway/bonjour)
- [원격 접속 (SSH)](/gateway/remote)
- [테일스케일 (Tailscale) 활용](/gateway/tailscale)

## 노드 및 플랫폼

- [노드 개요](/nodes)
- [브리지 프로토콜 (레거시 노드 전용)](/gateway/bridge-protocol)
- [노드 가이드: iOS](/platforms/ios)
- [노드 가이드: Android](/platforms/android)

## 보안 (Security)

- [보안 개요](/gateway/security)
- [게이트웨이 설정 레퍼런스](/gateway/configuration)
- [트러블슈팅](/gateway/troubleshooting)
- [Doctor (자가 진단 도구)](/gateway/doctor)
---
