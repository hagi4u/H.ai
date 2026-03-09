# BFF (Backend for Frontend)

> KREAM Pay는 BFF 패턴을 사용합니다.
> BFF(Next.js) ↔ Pay BE 간 HTTP 통신 관련 지식을 정리합니다.

## 핵심 개념

- BFF는 클라이언트(앱/웹)와 백엔드 사이에 위치하는 중간 서버
- KREAM Pay BFF: Next.js 기반
- Pay BE와 HTTP keep-alive 기반 연결 풀을 사용해 통신

## 주요 이슈

| 이슈 | 원인 | 문서 |
|------|------|------|
| ECONNRESET | Pod 교체 시 stale keep-alive 소켓 | `gotchas.md` |
