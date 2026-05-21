---
title: "安全工程啟蒙——從「不要 hardcode 密碼」到 5 層防禦架構"
date: 2026-05-21
weight: 4
series: ["MaiNeu 開發旅程"]
tags: ["security", "hmac", "jwt", "oauth", "owasp", "android", "backend"]
categories: ["技術版"]
description: "HMAC 請求簽名防 Replay Attack、email_verified 的 Google/Apple 型別差異、Auth Retry Storm 的 Mutex+Cooldown 解法、OWASP 2025 審計 15 個問題"
showToc: true
TocOpen: false
draft: false
---

# 安全工程啟蒙——從「不要 hardcode 密碼」到 5 層防禦架構

*MaiNeu 開發旅程 第四篇*

---

## 我對安全的原始認知

做 MaiNeu 之前，我對安全工程的認知大概是：

- 不要 hardcode API Key
- 用 HTTPS
- 密碼要 hash 存儲
- ……然後就沒了

做了 MaiNeu 之後，我跑了一次 OWASP 2025 安全審計，發現了 15 個問題。這個過程讓我對安全設計的理解從「幾條規則」變成了「系統性的防禦思維」。

---

## App Secret + HMAC：防 Replay Attack

MaiNeu 最初的 API 設計是在每個請求的 header 裡帶一個 `X-App-Secret`——一個只有 App 和後端知道的字串。感覺足夠了。

然後有人問我：**如果有人攔截了一個合法請求，然後完整複製這個請求重新發送（Replay Attack），你怎麼辦？**

我沒有好答案。

### Replay Attack 是什麼

攻擊者攔截一個合法的 HTTP 請求（header + body），把這個請求一模一樣地重新發送給後端。對後端來說，這個請求看起來完全合法——secret 是對的，body 是合理的 API 請求。

### HMAC 請求簽名

解法是把請求的內容、時間戳記、和一次性隨機字串一起 hash，產生唯一的簽名：

```
Signature = HMAC-SHA256(
    secret,
    timestamp + nonce + method + path + bodyHash
)
```

三個元素缺一不可：

| 元素 | 防禦目標 |
|------|---------|
| **timestamp** | 後端只接受 ±5 分鐘內的請求，過期直接拒絕 |
| **nonce** | 一次性字串，存在 Cloudflare KV，同一個 nonce 不能使用兩次 |
| **bodyHash** | SHA-256 of body，確保請求內容沒有被竄改 |

理解 HMAC 設計的邏輯之後，安全設計從「遵守規則」變成了「理解威脅模型」。

---

## email_verified 的 Google/Apple 型別陷阱

### 症狀

Google 回傳：
```json
{ "email": "user@gmail.com", "email_verified": true }
```

Apple 回傳：
```json
{ "email": "user@privaterelay.appleid.com", "email_verified": "true" }
```

**Google 的是 `boolean`，Apple 的是字串 `"true"`。**

如果你用 `if (emailVerified === true)`（嚴格比較），Apple 的就會被當成未驗證，拒絕登入。

### 為什麼 email_verified 很重要

沒有驗證的 email，不能用來確認用戶身份。如果攻擊者能用任意 email 創建 OAuth 帳號，然後嘗試和現有帳號合併，可能造成帳號劫持。後端規則：`email_verified` 為 false 的 OAuth 登入，不允許和現有帳號合併。

### 修法

```typescript
function parseEmailVerified(value: boolean | string): boolean {
    if (typeof value === 'boolean') return value;
    if (typeof value === 'string') return value === 'true';
    return false;  // 未知格式，保守處理
}
```

---

## Auth Retry Storm：Mutex + Cooldown

### 症狀

App 啟動時，幾秒內大量認證請求打到後端——因為 4 個 Manager 同時初始化，同時發現 Token 無效，同時呼叫 `authenticate()`。

### 解法

```kotlin
class AuthManager {
    private val authMutex = Mutex()
    private var cooldownUntil: Long = 0

    suspend fun getValidToken(): String {
        // 冷卻期內直接拒絕，不打 API
        if (System.currentTimeMillis() < cooldownUntil) {
            throw MenuApiException.RateLimited(retryAfterMs = 5000)
        }

        // Mutex 確保同一時間只有一個 authenticate() 在跑
        return authMutex.withLock {
            val token = tokenStorage.getAccessToken()
            if (token != null && !token.isExpired()) {
                token.value  // 已有有效 token，直接回傳
            } else {
                authenticate()
            }
        }
    }
}
```

第一個搶到 Mutex 的去認證，其他三個等待。認證完成後，剩下三個進去發現 Token 已有效，直接回傳。`authenticate()` 最多只跑一次。

---

## SocketTimeoutException 不等於「無網路」

`SocketTimeoutException` 的含義是：**連接建立了，但伺服器在超時時間內沒有回應**。可能是伺服器慢、後端處理時間長、或網路擁塞——不是「沒有網路」。

把它包含在「無網路」判斷裡，會誤判「有網路但後端慢」的情況，顯示不正確的離線提示。

```kotlin
// ✅ 正確：SocketTimeoutException 屬於「伺服器錯誤」，不是「無網路」
fun isNetworkError(e: Throwable): Boolean =
    e is UnknownHostException ||
    e is ConnectException ||
    e.message?.contains("Unable to resolve host") == true
    // SocketTimeoutException 不在這裡
```

---

## OWASP 審計：讓安全問題變得可見

系統性地把 OWASP Top 10 逐項對照自己的代碼，我發現了 15 個問題：

| 問題 | 嚴重性 | 狀態 |
|------|--------|------|
| API Key hardcoded in local config files | Critical | ✅ 已修 |
| 缺少 HMAC Request Signing | High | ✅ 已修 |
| 日誌輸出包含 Token 子字串 | High | ✅ 已修 |
| 缺少 Play Integrity 裝置驗證 | Medium | ✅ 已修 |
| OAuth email_verified 未處理 | Medium | ✅ 已修 |
| Auth 請求缺少 Rate Limit | Medium | ✅ 已修 |

重要的不是「修了幾個」，而是做了審計之後，安全問題變得可見了。

---

## 安全設計的核心思維

**安全設計不是「遵守規則清單」，而是「理解攻擊者的視角」。**

每一條安全規則背後都有一個具體的威脅：

| 規則 | 防禦的威脅 |
|------|-----------|
| HMAC 簽名 | Replay Attack |
| email_verified 檢查 | 帳號劫持 |
| Timing-safe 比較 | Timing Attack |
| Nonce 防重放 | Token 複製 |

當你理解了威脅，你就能在遇到新場景時做出正確的判斷，而不是死背規則。

---

*下一篇：[一個人維護四個環境——GitHub Actions CI/CD 實戰](/tech/05-cicd-github-actions/)*
