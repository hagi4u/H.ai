# E2E 테스트 작성 가이드

> KREAM E2E 모노레포(`kream-e2e`) 기반.
> Playwright + playwright-bdd + Gherkin(한국어).

---

## 도구 스택

| 도구 | 역할 |
|------|------|
| **Playwright** | 브라우저 자동화, Locator API, assertion |
| **playwright-bdd** | Gherkin feature ↔ step 바인딩 (`createBdd()`) |
| **Chrome DevTools Protocol (CDP)** | 배포 환경에서 DOM 구조 사전 조사 |
| **Gherkin** | 시나리오 정의 (영어 키워드 + 한국어 본문) |

---

## 핵심 워크플로우: CDP 조사 → Playwright Locator 작성

### 왜 CDP를 먼저 사용하는가

**목적: AI 어시스턴트의 토큰 소모를 최소화하기 위한 사전 조사.**

소스 코드만으로는 실제 렌더링된 DOM 구조를 정확히 알 수 없다.
셀렉터를 추측하면서 Playwright 테스트를 작성하면 시행착오가 반복되고 토큰이 낭비된다.

CDP로 **실제 DOM, computed style, 요소 존재 여부를 한 번에 전부 조사**한 뒤,
그 결과를 테이블로 정리해서 넘기면 → AI가 **한 번에 정확한 Playwright Locator 코드를 작성**할 수 있다.

```
[비효율] 소스 코드 추측 → 셀렉터 작성 → 실패 → 수정 → 재실행 → ... (토큰 낭비)
[효율]   CDP 조사 (1회) → 정확한 셀렉터 확보 → Playwright 코드 한 번에 완성
```

### Step 1: CDP로 DOM 조사

Chrome DevTools MCP 또는 Playwright의 `page.context().newCDPSession(page)`를 사용하여
대상 페이지의 실제 DOM 상태를 확인한다.

**조사 항목:**

| 확인 대상 | CDP 명령 |
|-----------|----------|
| 요소 존재 여부 | `DOM.getDocument` → `DOM.querySelectorAll` |
| 속성 값 | `DOM.getAttributes` |
| Computed Style | `CSS.getComputedStyleForNode` |
| JS 평가 | `Runtime.evaluate` |

**조사 결과를 테이블로 정리:**

```markdown
| 요소 | 셀렉터 | 존재 | 주요 속성/스타일 |
|------|--------|------|-----------------|
| 이미지 탭 | `.media-container img` | O | alt="탭명", naturalWidth > 0 |
| Lottie 탭 | `.media-container svg` | O | 내부 g/path 존재 |
| 텍스트 탭 | `li.li_tab a.tab:not(:has(.media-container))` | O | textContent 존재 |
```

### Step 2: 셀렉터 상수 정의

CDP 조사 결과를 기반으로 steps 파일 상단에 셀렉터를 정리한다.

```typescript
// CDP로 확인한 DOM 구조 기반 셀렉터
const SELECTORS = {
  mediaContainer: ".media-container",
  mediaImage: ".media-container img",
  mediaSvg: ".media-container svg",
  srOnly: ".media-container .sr-only",
  allTabs: "li.li_tab a.tab",
  tabName: ".tab_name",
} as const;
```

### Step 3: Playwright Locator API로 step 구현

**CDP는 조사 도구일 뿐, 테스트 코드에서는 Playwright Locator API를 사용한다.**

```typescript
// DOM 요소 존재/가시성 → locator + toBeVisible
Then("이미지 미디어 탭이 노출됨을 확인한다", async ({ page }) => {
  const imgLocator = page.locator(SELECTORS.mediaImage);
  await expect(imgLocator.first()).toBeVisible({ timeout: 10000 });
  expect(await imgLocator.count()).toBeGreaterThan(0);
});

// JS 속성 확인 → locator.evaluate()
Then("미디어 탭 이미지가 정상 로드됨을 확인한다", async ({ page }) => {
  const images = page.locator(SELECTORS.mediaImage);
  const count = await images.count();
  for (let i = 0; i < count; i++) {
    const naturalWidth = await images.nth(i).evaluate(
      (img: HTMLImageElement) => img.naturalWidth
    );
    expect(naturalWidth).toBeGreaterThan(0);
  }
});

// Computed Style 확인 → locator.evaluate(getComputedStyle)
Then("aspect-ratio가 적용됨을 확인한다", async ({ page }) => {
  const containers = page.locator(SELECTORS.mediaContainer);
  const count = await containers.count();
  for (let i = 0; i < count; i++) {
    const aspectRatio = await containers.nth(i).evaluate(
      (el) => getComputedStyle(el).aspectRatio
    );
    expect(aspectRatio).not.toBe("auto");
  }
});
```

### CDP 조사 ↔ Playwright 작성 매핑 정리

| CDP 조사 방법 | Playwright 작성 방법 |
|--------------|---------------------|
| `DOM.querySelectorAll` (존재 확인) | `page.locator(selector)` + `toBeVisible()` / `.count()` |
| `DOM.getAttributes` (속성 확인) | `locator.getAttribute()` |
| `CSS.getComputedStyleForNode` (스타일) | `locator.evaluate(el => getComputedStyle(el).prop)` |
| `Runtime.evaluate` (JS 값) | `locator.evaluate()` 또는 `page.evaluate()` |
| 요소 내 자식 탐색 | `locator.locator(childSelector)` |
| 부정 셀렉터 (요소 없음) | `:not(:has(...))` 또는 `.count()` === 0 |

---

## 레포 구조 & 파일 컨벤션

```
kream-e2e/
├── features/                  ← 공유 Gherkin 시나리오
│   └── home/
│       ├── ranking.feature
│       └── tab-media.feature
├── web/
│   ├── steps-play/            ← playwright-bdd step 구현체
│   │   ├── home/
│   │   │   ├── ranking.steps.ts
│   │   │   └── tab-media.steps.ts
│   │   ├── shared/
│   │   │   ├── navigation.steps.ts   ← 공용 Given/When
│   │   │   └── utils.ts
│   │   └── base.steps.ts     ← 전역 공용 step
│   ├── fixture/               ← auth storageState
│   └── playwright.config.ts
├── mweb/                      ← 모바일 웹 E2E
└── common/inventory/          ← 테스트 사용자 데이터
```

**매핑 규칙:**
- `features/{domain}/{name}.feature` ↔ `web/steps-play/{domain}/{name}.steps.ts`
- 공용 step (페이지 열기, 스크롤, 대기 등)은 `shared/` 또는 `base.steps.ts`에 정의
- 중복 step 정의 금지 — 기존 step 확인 후 작성

---

## Feature 파일 작성 규칙

```gherkin
@tab-media
Feature: HOME 탭 미디어(Lottie/이미지) 검증
  홈 탭의 Lottie 애니메이션과 이미지 미디어가 정상 동작하는지 검증한다

  Background:
    Given "/" 페이지를 연다

  @web @P0 @logout
  Scenario: 이미지 미디어 탭이 정상 렌더링된다
    Then 이미지 미디어 탭이 노출됨을 확인한다
    And 미디어 탭 이미지가 정상 로드됨을 확인한다
```

| 규칙 | 설명 |
|------|------|
| 키워드 | **영어** (`Feature`, `Scenario`, `Given`, `When`, `Then`) |
| 본문 | **한국어** |
| Feature 태그 | 도메인 식별자 (`@tab-media`, `@ranking`) |
| Scenario 태그 | 플랫폼 + 우선순위 + 인증 (`@web @P0 @logout`) |
| Background | 공통 전제조건 (페이지 열기 등) |
| `@P0` | 스모크 — PR마다 실행 |
| `@P1` | 회귀 — merge 후 전체 |
| `@P2` | 보조 — 야간 cron |

---

## Steps 파일 작성 패턴

```typescript
import { expect } from "@playwright/test";
import { createBdd } from "playwright-bdd";

const { Given, When, Then } = createBdd();

// 1. CDP 조사 기반 셀렉터 상수
const SELECTORS = { ... } as const;

// 2. When: 사용자 액션
When("페이지를 하단으로 스크롤한다", async ({ page }) => {
  await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
  await page.waitForTimeout(500);
});

// 3. Then: 검증 (Playwright assertion)
Then("이미지가 노출됨을 확인한다", async ({ page }) => {
  await expect(page.locator(SELECTORS.image).first()).toBeVisible();
});
```

**작성 원칙:**
- `page.locator()` + `expect(locator)` 우선 사용
- DOM 속성/스타일 확인이 필요하면 `locator.evaluate()` 사용
- `page.evaluate()`는 스크롤 등 부수효과 액션에만 사용
- CDP 직접 호출(`newCDPSession`)은 테스트 코드에 넣지 않음
- `waitForTimeout`은 스크롤 안정화 등 최소한으로만 사용 (500ms 이하)

---

## 실행 명령어

```bash
cd ~/Developments/kream-e2e/web

# feature → spec 생성
pnpm bddgen

# 전체 실행
pnpm play

# 태그 필터링
npx playwright test --grep @tab-media

# 스모크 (P0)
pnpm test:smoke

# UI 모드 (디버깅)
npx playwright test --ui
```

---

## 새 테스트 추가 체크리스트

1. **CDP로 대상 페이지 DOM 조사** → 셀렉터, 속성, 스타일 확인
2. **조사 결과를 테이블로 정리** → 어떤 요소가 어떤 상태인지 기록
3. **`features/{domain}/{name}.feature` 작성** → 시나리오 정의
4. **기존 step 검색** → `shared/`, `base.steps.ts`에 이미 있는지 확인
5. **`steps-play/{domain}/{name}.steps.ts` 작성** → 셀렉터 상수 + Locator API
6. **`pnpm bddgen`** → spec 파일 생성
7. **`npx playwright test --grep @태그`** → 실행 확인

---

## 로컬 / CI 분리 전략

### 태그 기반 분리

| 트리거 | 실행 범위 | 태그 |
|--------|---------|------|
| PR 오픈/업데이트 | 스모크 | `@P0 @web` |
| main merge | 전체 회귀 | `@web` 전체 |
| 야간 cron | 다중 브라우저 | 전체 |

### 환경 변수

| 변수 | 용도 |
|------|------|
| `E2E_BASE_URL` | 테스트 대상 URL (dev-branch-8 등) |
| `E2E_USER_ID` | 인벤토리 사용자 ID |
| `E2E_VIDEO` | 비디오 녹화 (`on`) |
| `BROWSER` | 브라우저 선택 |

---

## QA팀 ↔ FE팀 역할 분리

| 역할 | QA팀 | FE팀 |
|------|------|------|
| Feature 시나리오 정의 | 리뷰어 (누락 케이스 검토) | 주도 |
| Steps 코드 작성 | — | 주도 |
| CI 파이프라인 | — | 주도 |
| 회귀 테스트 실행 | 주도 | 환경 지원 |
| 실패 분석 | 버그 판별 | 코드/환경 이슈 판별 |

---

## 민감 데이터 처리

1. **전용 테스트 계정** — 실 사용자 데이터 절대 금지
2. **결제 데이터** — PG사 테스트 카드 + Sandbox 환경
3. **`.env*` 전체 `.gitignore`** — CI 시크릿은 GitHub Actions Secrets
4. **사용자 인벤토리** — `common/inventory/` JSON 파일, `${user1.email}` 형태로 참조
