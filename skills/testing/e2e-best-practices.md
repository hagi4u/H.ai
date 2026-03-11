# E2E 테스트 모범사례

> KREAM FE팀 E2E 레포 방향성 수립을 위한 조사 정리.
> 적용 대상: 크림 웹(Nuxt3), 크림페이(Next.js)

---

## 도구 선택: Playwright 권장

| 항목 | Playwright | Cypress |
|------|-----------|---------|
| 브라우저 | Chrome, Firefox, **Safari(WebKit)** | Chrome, Edge, Firefox (Safari 없음) |
| 병렬 실행 | 네이티브 sharding | 유료 플랜만 |
| 멀티탭/멀티도메인 | 기본 지원 | 제한적 |
| 결제 팝업(PG) | ✅ 리디렉션 + 팝업 처리 가능 | 어려움 |
| Nuxt3 통합 | `@nuxt/test-utils` 공식 지원 | 별도 설정 |
| Next.js 통합 | 공식 템플릿 존재 | 가능 |

**Playwright 선택 이유 (KREAM 맥락):**
- 모바일 사용자 비중 → Safari/WebKit 커버 필수
- 결제 플로우 → PG사 팝업/리디렉션 처리 네이티브 지원
- `storageState`로 로그인 세션 재사용 → 인증 반복 최소화

---

## 레포 구조

```
e2e/
├── auth/
│   ├── login.spec.ts
│   └── signup.spec.ts
├── product/
│   ├── search.spec.ts          # 검색 → 필터 → 상품 진입
│   └── product-detail.spec.ts  # 상품 상세 → 입찰/즉시구매 CTA
├── trade/
│   ├── buy-now.spec.ts         # 즉시구매 플로우 (Happy Path)
│   ├── bidding.spec.ts
│   └── checkout.spec.ts
├── mypage/
│   └── order-history.spec.ts
├── fixtures/
│   ├── auth.fixture.ts         # 로그인 세션 fixture
│   └── product.fixture.ts
└── pages/                      # Page Object Model
    ├── ProductDetailPage.ts
    ├── CheckoutPage.ts
    └── LoginPage.ts
```

**원칙: 피처 기반 파일 구분 + 유저 시나리오 관점으로 작성**
- E2E는 Happy Path 중심, 예외/에러 케이스는 Unit/Integration에 위임

---

## 로컬 / CI 분리 전략

### 태그 기반 분리

```typescript
test('즉시구매 핵심 플로우 @smoke', ...)       // PR마다 실행 (~3분 목표)
test('전체 결제 시나리오 @regression', ...)    // merge 후 전체 회귀
test('실계좌 결제 테스트 @local-only', ...)    // CI 제외, 로컬만
```

### CI 트리거별 범위

| 트리거 | 실행 범위 | 목표 시간 |
|--------|---------|---------|
| PR 오픈/업데이트 | `@smoke` 태그만 | ~3분 |
| main 브랜치 merge | `@regression` 전체 | sharding으로 ~10분 |
| 야간 cron | 전체 + 다중 브라우저 | 제한 없음 |

### 환경 파일 분리

```
.env.local    # 개인 테스트 계정, 실제 스테이징 URL
.env.ci       # CI 전용 계정, 마스킹된 데이터
.env.staging  # 스테이징 검증
```

`.env*` 전체 `.gitignore` 등록 필수. CI 시크릿은 GitHub Actions Secrets로 관리.

---

## 민감 데이터 처리

1. **전용 테스트 계정** — 실 사용자 데이터 절대 금지
2. **결제 데이터** — PG사 테스트 카드 번호 사용 (`4242 4242 4242 4242` 류), Sandbox 환경 활용
3. **실계좌 연동** — `@local-only` 태그로 분리, CI 실행 제외
4. **스크린샷/trace 마스킹** — 카드번호, 개인정보 필드는 `maskTextContent` 옵션 활용

---

## BDD(Gherkin/Cucumber) 미사용

**도입하지 않는다.**

Gherkin은 QA/기획자가 시나리오를 직접 작성할 때 의미가 있다.
KREAM FE팀은 시나리오 정의와 코드 작성 모두 FE가 주도하므로 불필요한 레이어다.

- 기존 `kream-e2e` 레포(Cucumber 기반)는 QA 중심 설계 → 참조하지 않음
- 개발자 친화적인 Playwright 네이티브 방식으로 작성

**작성 스타일** — 주석으로 의도 명시:
```typescript
test('상품 즉시구매 후 주문 완료 확인', async ({ page }) => {
  // Given: 로그인된 사용자
  await loginUser(page, testUser)
  // When: 상품 즉시구매 플로우 실행
  await page.goto('/products/12345')
  await page.getByRole('button', { name: '즉시구매' }).click()
  // Then: 주문 완료 페이지 확인
  await expect(page.getByText('주문이 완료되었습니다')).toBeVisible()
})
```

---

## QA팀 ↔ FE팀 역할 분리

| 역할 | QA팀 | FE팀 |
|------|------|------|
| 테스트 시나리오 정의 | **리뷰어** (누락 케이스 검토) | 주도 |
| 테스트 코드 작성 | — | 주도 |
| CI 파이프라인 설정 | — | 주도 |
| 회귀 테스트 실행 | 주도 (FE가 만든 E2E 실행) | 환경 지원 |
| 실패 시 분석 | 버그 판별 | 코드/환경 이슈 판별 |
| UI 변경 시 업데이트 | 알림 | 즉시 수정 |

- 시나리오 작성은 FE 주도, QA는 리뷰어로 참여 (직접 실행하는 역할이므로 누락 케이스를 가장 잘 발견)
- FE가 E2E를 만들면 QA는 그것을 회귀 검증 수단으로 활용
- 자동화 커버된 케이스는 QA 수동 회귀 목록에서 제외 → QA 리소스 확보

## 유관자 미팅

> Bobby가 주관해서 유관자를 모아 이 방향성을 공유하고 합의할 예정.

**미팅 목적**: E2E 도입 방향성 공유 및 역할 합의
**예상 참석자**: FE팀, QA팀 (유관자)
**상태**: 🔜 예정

---

## 커버리지 가시화

### Playwright 내장 도구
```bash
npx playwright test --reporter=html
npx playwright show-report  # 통과/실패/스킵 시각화
```

- **Trace Viewer**: 스텝별 스크린샷 + 네트워크 요청 + 콘솔 타임라인
- `--reporter=github` 플래그로 PR 코멘트 자동화

### 커버리지 매핑 테이블 (팀 관리용)

```
플로우                 | 자동화 | 태그          | 담당
──────────────────────────────────────────────────
로그인 (이메일)        | ✅     | @smoke        | FE
로그인 (소셜)          | ❌     |               |
상품 검색 → 필터       | ✅     | @smoke        | FE
즉시구매 (카드)        | ✅     | @regression   | FE
입찰 → 낙찰           | ❌     |               |
마이페이지 주문내역    | 🔄     |               | QA
```

---

## 실행 로드맵 (KREAM FE팀)

### Phase 1 (1~2개월): 기반 구축
- 별도 E2E 레포 생성 (메인 레포와 분리)
- Playwright 세팅 + `@nuxt/test-utils` 통합
- 핵심 Happy Path 3~5개: 로그인, 상품 상세 진입, 즉시구매
- `.env.local` / `.env.ci` 분리 + GitHub Secrets 연동

### Phase 2 (3~4개월): 확장
- `@smoke` / `@regression` 태그 체계 수립
- QA팀과 커버리지 매핑 테이블 작성
- PR 스모크 자동화 + 야간 전체 회귀 실행

### Phase 3: 최적화
- HTML Report → Slack 알림 연동
- BDD 도입 재검토
- 결제 시나리오 → PG Sandbox 완전 자동화
