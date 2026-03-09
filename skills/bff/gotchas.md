# BFF Gotchas

## ECONNRESET — Pod 교체 시 keep-alive 소켓 강제 종료

### 현상
Pay BE 배포 시 Pod IP가 교체되면서 BFF가 유지하던 TCP keep-alive 연결이 강제 종료(RST 패킷)되어 `ECONNRESET` 에러 발생.

### 원인
```
BFF ──[keep-alive TCP]──> Pay BE Pod (old IP)
           ↓ 배포 발생
Pay BE Pod 종료 → 커널이 RST 패킷 전송
           ↓
BFF: ECONNRESET (연결이 살아있다고 믿었는데 갑자기 끊김)
```

HTTP connection pool에서 keep-alive 소켓을 재사용하려는 순간, 이미 서버 측에서 종료된 **stale socket**이라 RST를 받아 에러 발생.

**Kubernetes 환경에서 Pod 교체 시 정상적으로 예측 가능한 현상이다.**

### 방어 방법

#### 1. BFF 클라이언트: ECONNRESET retry (가장 중요, 즉효)

idempotent 요청이라면 안전하게 재시도 가능.

```typescript
import axiosRetry from 'axios-retry';

axiosRetry(axiosInstance, {
  retries: 3,
  retryCondition: (error) => {
    return error.code === 'ECONNRESET' || error.code === 'ECONNREFUSED';
  },
  retryDelay: axiosRetry.exponentialDelay,
});
```

#### 2. Pay BE 서버: keepAliveTimeout 조정

서버의 `keepAliveTimeout`을 **로드밸런서(ALB/NLB) 타임아웃보다 짧게** 설정.
(LB가 먼저 끊으면 같은 문제 발생 → 서버 타임아웃 < LB 타임아웃)

```typescript
// Node.js Express
server.keepAliveTimeout = 60000;  // 60초 (ALB default 60s보다 짧게)
server.headersTimeout = 65000;    // keepAliveTimeout + 5초
```

#### 3. Pay BE 서버: Graceful Shutdown

```yaml
# Kubernetes deployment
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 5"]  # 새 연결 유입 차단 후 대기
```

```typescript
// SIGTERM 수신 시 기존 keep-alive 연결 빠르게 종료 유도
process.on('SIGTERM', () => {
  server.keepAliveTimeout = 1;
});
```

#### 4. 인프라: 서비스 메시 draining

Istio 등 서비스 메시의 `terminationDraining` 기능으로 기존 연결을 드레인한 후 Pod 종료.

### 우선순위

| 레이어 | 방법 | 효과 |
|--------|------|------|
| BFF 클라이언트 | ECONNRESET retry | 즉각적, 가장 효과적 |
| Pay BE 서버 | keepAliveTimeout 조정 | Stale socket 줄임 |
| Pay BE 서버 | preStop + SIGTERM 처리 | 배포 중 연결 정리 |
| 인프라 | 서비스 메시 draining | 완전한 무중단 배포 |
