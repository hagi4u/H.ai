# 생체인식 × 웹뷰 연동 (iOS / Android)

> 크림페이 컨텍스트: Next.js BFF + 크림 앱 WebView 구조에서 생체인식 결제 구현

---

## 전체 흐름

```
크림 앱 (iOS/Android)
  └── WKWebView / WebView (크림페이 Next.js)
         │
         │  1. 결제하기 버튼 클릭
         ▼
  [Next.js 클라이언트]
         │
         │  2. JS Bridge 호출 → 생체인식 요청
         ▼
  [네이티브 앱 생체인식 UI]
         │
         │  3. 인증 성공 콜백 수신
         ▼
  [Next.js 클라이언트]
         │
         │  4. POST /api/pay/confirm (세션 포함)
         ▼
  [Next.js BFF]
         │
         │  5. 생체인식 완료 플래그 검증 후 결제 백엔드 호출
         ▼
  [KREAM 결제 백엔드]
```

---

## FE 래퍼 (플랫폼 분기)

```typescript
// lib/native-bridge.ts

function detectPlatform(): 'ios' | 'android' | 'web' {
  const ua = navigator.userAgent
  if (/iPhone|iPad/.test(ua)) return 'ios'
  if (/Android/.test(ua)) return 'android'
  return 'web'
}

export function requestBioAuth(): Promise<boolean> {
  return new Promise((resolve) => {
    const requestId = crypto.randomUUID()
    const platform = detectPlatform()

    // Native → JS 콜백 수신
    window.__bioCallback__ = (id: string, success: boolean) => {
      if (id === requestId) {
        delete window.__bioCallback__
        resolve(success)
      }
    }

    if (platform === 'ios') {
      window.webkit?.messageHandlers?.bioAuth?.postMessage({ requestId })
    } else if (platform === 'android') {
      window.NativeBridge?.requestBioAuth(requestId)
    } else {
      resolve(false) // 웹 전용 폴백
    }
  })
}
```

```typescript
// 결제 버튼에서 사용
async function handlePayment() {
  const authed = await requestBioAuth()
  if (!authed) return
  await fetch('/api/pay/confirm', { method: 'POST', ... })
}
```

**requestId 패턴이 중요한 이유**: 네이티브 콜백은 비동기이므로 다중 요청 시 응답 매칭 필요.

---

## iOS — WKWebView + LocalAuthentication

```
JS → window.webkit.messageHandlers.bioAuth.postMessage({ requestId })
   → Swift: WKScriptMessageHandler.userContentController()
   → LAContext().evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics)
   → webView.evaluateJavaScript("window.__bioCallback__(id, success)")
```

### Swift 구현 핵심

```swift
// 1. 브릿지 등록
contentController.add(self, name: "bioAuth")

// 2. 메시지 수신 → 생체인식 호출
func userContentController(_ controller: WKUserContentController,
                           didReceive message: WKScriptMessage) {
    let requestId = (message.body as? [String: Any])?["requestId"] as? String
    let context = LAContext()
    context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics,
                           localizedReason: "결제를 위해 생체인식을 확인합니다") { success, _ in
        DispatchQueue.main.async {
            let js = "window.__bioCallback__('\(requestId!)', \(success))"
            self.webView.evaluateJavaScript(js, completionHandler: nil)
        }
    }
}
```

**필수 설정**:
- `Info.plist`: `NSFaceIDUsageDescription` 추가
- Xcode Capabilities: Face ID 엔타이틀먼트 추가

---

## Android — WebView + BiometricPrompt

```
JS → window.NativeBridge.requestBioAuth(requestId)
   → @JavascriptInterface 메서드 호출
   → BiometricPrompt.authenticate()
   → webView.evaluateJavascript("window.__bioCallback__(id, true)")
```

### Kotlin 구현 핵심

```kotlin
class NativeBridge(private val activity: AppCompatActivity, private val webView: WebView) {
    @JavascriptInterface
    fun requestBioAuth(requestId: String) {
        activity.runOnUiThread { showBiometricPrompt(requestId) }
    }

    private fun showBiometricPrompt(requestId: String) {
        val prompt = BiometricPrompt(activity, ContextCompat.getMainExecutor(activity),
            object : BiometricPrompt.AuthenticationCallback() {
                override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
                    callbackToJS(requestId, true)
                }
                override fun onAuthenticationFailed() { callbackToJS(requestId, false) }
                override fun onAuthenticationError(code: Int, msg: CharSequence) { callbackToJS(requestId, false) }
            })
        val info = BiometricPrompt.PromptInfo.Builder()
            .setTitle("결제 확인")
            .setNegativeButtonText("취소")
            .build()
        prompt.authenticate(info)
    }

    private fun callbackToJS(requestId: String, success: Boolean) {
        activity.runOnUiThread {
            webView.evaluateJavascript("window.__bioCallback__('$requestId', $success)", null)
        }
    }
}

// 등록
webView.addJavascriptInterface(NativeBridge(this, webView), "NativeBridge")
```

---

## BFF 검증 전략

생체인식은 디바이스 로컬 검증 → BFF가 직접 확인 불가.
**세션 + 타임스탬프 플래그**로 보완:

```typescript
// POST /api/pay/request-bio — 생체인식 완료 후 클라이언트가 호출
export async function POST(req: Request) {
  const session = await getSession()
  session.bioVerified = true
  session.bioVerifiedAt = Date.now()
  await session.save()
  return Response.json({ ok: true })
}

// POST /api/pay/confirm
export async function POST(req: Request) {
  const session = await getSession()
  const elapsed = Date.now() - (session.bioVerifiedAt ?? 0)
  if (!session.bioVerified || elapsed > 30_000) { // 30초 TTL
    return Response.json({ error: 'Bio auth required' }, { status: 403 })
  }
  // 결제 처리
}
```

---

## WebAuthn / Passkey 표준과의 관계

| 방식 | 표준 | 구현 복잡도 | 추천 시점 |
|------|------|:---------:|---------|
| JS Bridge + LAContext/BiometricPrompt | 비표준 (커스텀) | 낮음 | **단기 — 실용적** |
| WebAuthn in WebView | FIDO2 | 높음 | ⚠️ 플랫폼 제약 많음 |
| Native Passkey API | FIDO2 | 중간 | 중기 — 표준화 목표 시 |

### WebAuthn 웹뷰 지원 현황 (2025)

| 환경 | 지원 | 비고 |
|------|:---:|------|
| iOS Safari | ✅ | iOS 16+ |
| iOS WKWebView | ⚠️ | Associated Domains 설정 + iOS 16+ 필요 |
| Android Chrome | ✅ | |
| Android WebView | ❌ | Credential Manager API 별도 연동 필요 |

---

## 플랫폼별 제약 요약

| 항목 | iOS | Android |
|------|-----|---------|
| 생체인식 API | `LocalAuthentication` | `BiometricPrompt` (API 28+) |
| JS → Native | `window.webkit.messageHandlers.X.postMessage()` | `window.X.method()` |
| Native → JS | `webView.evaluateJavaScript()` | `webView.evaluateJavascript()` |
| 세션 격리 | WKWebView ↔ Safari 격리됨 (쿠키 수동 주입 필요) | 동일 |

### 세션 격리 해결
앱이 WebView 로드 시 세션 토큰을 쿠키로 주입:
- iOS: `WKHTTPCookieStore`로 쿠키 수동 설정
- Android: `CookieManager.getInstance().setCookie()`

---

## 보안 체크리스트

- [ ] WebView 로드 URL을 `pay.kream.com`으로 화이트리스트 제한
- [ ] HTTPS 강제 (Android: `android:usesCleartextTraffic="false"`)
- [ ] requestId 30초 TTL 후 자동 해제
- [ ] 생체인식 실패 3회 → PIN 폴백 제공
- [ ] Android `@JavascriptInterface` 어노테이션 누락 확인
- [ ] 앱 백그라운드 진입 시 LAContext 무효화 처리
