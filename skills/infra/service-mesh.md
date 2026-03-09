# 서비스 메시 (Service Mesh)

## Istio란

Kubernetes 위에서 동작하는 서비스 메시 오픈소스.

마이크로서비스가 많아지면 서비스 간 통신에서 재시도, 타임아웃, TLS, 모니터링 등을
각 앱마다 직접 구현해야 한다. Istio는 이걸 **앱 코드 수정 없이 인프라 레벨에서** 처리한다.

---

## 사이드카 패턴

모든 Pod에 **Envoy 프록시 컨테이너를 하나 더 자동 주입**하는 패턴.

```
┌──────────────────────────┐
│         Pod              │
│  ┌────────┐  ┌────────┐  │
│  │  앱    │  │ Envoy  │  │  ← 사이드카 (Istio가 자동 주입)
│  │컨테이너 │  │프록시  │  │
│  └────────┘  └────────┘  │
└──────────────────────────┘
```

이름 유래: 오토바이 옆에 붙는 보조석(sidecar) → 메인 앱 옆에 하나 더 붙는다는 의미.

모든 인바운드/아웃바운드 트래픽이 Envoy를 경유한다:

```
[BFF Pod]                    [Pay BE Pod]
앱 → Envoy ──네트워크──> Envoy → 앱
```

---

## Istio가 제공하는 기능

| 기능 | 설명 |
|------|------|
| mTLS | 서비스 간 자동 암호화 |
| 재시도 / 타임아웃 | 앱 코드 없이 설정 파일로 |
| Circuit breaker | 특정 서비스 장애 시 빠른 실패 |
| 트래픽 분산 | 카나리 배포, A/B 테스트 |
| 모니터링 | 서비스 간 지연, 에러율 자동 수집 |
| Graceful draining | Pod 종료 시 기존 연결 자연 종료 |

---

## Graceful Draining (ECONNRESET 방어)

Pod 종료 시 Envoy 사이드카가 드레인 모드로 진입:

```
SIGTERM
  → Envoy: 새 요청은 다른 Pod로 라우팅
  → 기존 연결의 in-flight 요청은 완료될 때까지 유지
  → 모두 완료되면 Pod 종료
```

앱 코드 수정 없이 Graceful Shutdown과 동일한 효과. → `bff/gotchas.md` 참조

---

## Istio 사용 여부 확인

```bash
kubectl get pods -n <namespace> -o jsonpath='{.items[*].spec.containers[*].name}'
# 결과에 "istio-proxy" 가 있으면 Istio 사용 중
```

---

## Istio 없을 때 대안

앱/BFF 코드에서 직접 구현해야 하는 것들:
- ECONNRESET retry → `bff/gotchas.md`
- Graceful shutdown (SIGTERM 핸들링)
- Circuit breaker (예: `opossum` 라이브러리)
