---
title: "Auth 的那些坑——JWT、Session 管理、OAuth Fusion 的血淚教訓"
date: 2026-05-21
weight: 6
series: ["MaiNeu 開發旅程"]
tags: ["android", "authentication", "jwt", "oauth", "session-management", "debugging"]
categories: ["技術版"]
description: "AuthManager singleton state 未重置、navigateBasedOnAuthStatus 判斷順序錯誤、clearFusion 漏清 error state、applyCloudData 覆蓋本地設定——6 個 Auth 系統的真實血淚坑"
showToc: true
TocOpen: false
draft: false
---

# Auth 的那些坑——JWT、Session 管理、OAuth Fusion 的血淚教訓

*MaiNeu 開發旅程 第六篇*

---

## Authentication：看起來簡單，實際上是地雷區

Authentication 是每個 App 都必須做的功能，理論上也是最成熟的——OAuth 2.0、JWT、Session 管理，業界有一大堆標準做法。

然而，MaiNeu 的 Auth 系統前後迭代了至少六個月，踩了無數坑。

不是因為原理不懂，而是因為**邊界案例無窮無盡**：用戶的設備重開機、Token 在 App 執行中間過期、同一個 Email 用不同方式登入、網路在 Auth 流程中途斷線……

---

## 坑一：AuthManager Singleton，restoreSession 忘了重置 state

### 症狀

用戶登出之後，重新打開 App，登入頁面會閃一下然後直接跳到主頁面——即使用戶沒有輸入任何帳號密碼。

### 根本原因

`AuthManager` 是 Koin singleton，有一個 `StateFlow` `_authState` 記錄認證狀態。

用戶登出時，我清除了 Token Storage（刪掉 EncryptedSharedPreferences 裡的 token），但**忘了重置 `_authState`**。

結果：下次打開 App，`restoreSession()` 去 Token Storage 找 token，沒找到，但 `_authState.value.isLoggedIn` 還是 `true`（上次登入時設的）。`LaunchActivity` 看到 `isLoggedIn=true`，直接跳到主頁面。

### 正確做法

```kotlin
suspend fun restoreSession() {
    val token = tokenStorage.getAccessToken()
    if (token == null || token.isExpired()) {
        // 必須重置 state！不能只靠 Token Storage 為空
        _authState.update { AuthState.empty() }
        return
    }
    // ...
}
```

**教訓：singleton 的 in-memory state 和 Storage 必須同步。任何 logout 或 session 清除操作，必須同時清除兩個地方。**

---

## 坑二：navigateBasedOnAuthStatus 判斷邏輯順序錯誤

### 症狀

某些 returning user（已登入的用戶重新打開 App）會被送到登入頁面，然後卡在那裡。

### 根本原因

```kotlin
// ❌ 錯誤的判斷順序
fun navigateBasedOnAuthStatus() {
    if (!hasRegisteredAccount()) {
        navigate(LoginActivity)  // 只檢查「是否有過帳號」
        return
    }
    navigate(MainActivity)  // 有帳號 → 主頁面，但 token 可能過期！
}
```

`hasRegisteredAccount()` 只檢查「這個設備曾經有過帳號」，不檢查「token 是否有效」。Returning user 的 token 可能過期（超過 7 天沒打開 App）。

```kotlin
// ✅ 先檢查 token 是否有效
fun navigateBasedOnAuthStatus() {
    val token = tokenStorage.getAccessToken()
    if (token != null && !token.isExpired()) {
        navigate(MainActivity)  // Token 有效，進主頁面
        return
    }
    if (hasRegisteredAccount()) {
        navigate(LoginActivity, rememberedEmail = true)  // 有帳號但 token 過期
        return
    }
    navigate(OnboardingActivity)  // 新用戶
}
```

---

## 坑三：所有 Auth 請求必須帶 deviceId

### 症狀

用戶成功登入，但之後所有 API 請求都回傳 `DEVICE_NOT_FOUND` 錯誤。

### 根本原因

MaiNeu 的 Token 和 `deviceId` 綁定。我新增一個 Auth 端點時，忘記在 `AuthRequest` 裡加上 `deviceId`。後端產生的 Token 裡 `deviceId` 是空字串，之後所有 API 請求都失敗。

```kotlin
// ❌ 漏了 deviceId
data class AnonymousAuthRequest(val appVersion: String, val platform: String)

// ✅ 必須帶 deviceId
data class AnonymousAuthRequest(
    val deviceId: String,
    val appVersion: String,
    val platform: String
)
```

**教訓：所有 Auth 請求（Login、Register、Anonymous、OAuth）都必須帶 `deviceId`——這是 Invariant，不是可選的欄位。**

---

## 坑四：clearFusion() 漏了清除 error state

### 症狀

用戶嘗試 Google OAuth 融合帳號，點取消後繼續其他操作，突然出現舊的 fusion 失敗錯誤訊息，出現在完全不相關的頁面上。

### 根本原因

```kotlin
// ❌ 漏了清除 error
fun clearFusion() {
    _authState.update { state -> state.copy(fusionToken = null) }
}

// ✅ 同時清除 error
fun clearFusion() {
    _authState.update { state -> state.copy(fusionToken = null, error = null) }
}
```

**教訓：清除流程狀態時，必須清除所有相關的 sub-state，包括 error。**

---

## 坑五：link 帳號失敗是「預期行為」，不是錯誤

### 症狀

登入流程短暫顯示一個錯誤訊息然後馬上消失，讓用戶困惑。

### 根本原因

`linkEmailAccount` 失敗（因為 Email 已存在）是 fallthrough 邏輯，是「預期的失敗路徑」。但我設了 `error` 狀態，UI 短暫顯示錯誤訊息，然後登入成功清除了它。

```kotlin
// ❌ link 失敗時設了 error（但這是預期行為）
onFailure { error ->
    _authState.update { it.copy(isLoading = false, error = error.message) }
}

// ✅ link 失敗是預期行為，不設 error
onFailure { error ->
    _authState.update { it.copy(isLoading = false) }
    // 繼續 fallthrough 到登入流程
}
```

**教訓：「失敗」不等於「錯誤」。區分「預期的失敗路徑」（user 不需要知道）和「真正的錯誤」（需要顯示）。**

---

## 坑六：applyCloudData 覆蓋本地偏好設定

### 症狀

用戶的 App 語言在每次啟動後被重置。

### 根本原因

`applyCloudData()` 把整個 `UserPreferences` 物件用雲端版本覆蓋，包括了 `appLanguage`（App 界面語言）這個從未同步到後端的設定。

```kotlin
// ❌ 整個覆蓋，包括只存本機的設定
fun applyCloudData(cloudPrefs: UserPreferences) {
    _prefs = cloudPrefs
}

// ✅ 只 merge 雲端同步的欄位
fun applyCloudData(cloudPrefs: UserPreferences) {
    _prefs = _prefs.copy(
        targetLanguage = cloudPrefs.targetLanguage,
        resultLayout = cloudPrefs.resultLayout
        // 其他欄位保留本地值
    )
}
```

**教訓：設計「雲端同步」功能時，必須明確定義哪些欄位是雲端同步的，哪些是本機獨立的——這個邊界要在 data model 層定義清楚。**

---

## Auth 系統設計的核心洞察

**Auth 不只是「登入/登出」，而是「管理信任關係的整個生命週期」。**

六個月的 Auth 開發讓我學到：好的 Auth 系統的共同特點是每個決定都有明確的理由——不是「這樣寫感覺對」，而是「這樣設計是因為我們預期這個場景的用戶行為是 X，所以用 Y 策略處理」。

---

*下一篇：[設計 AI 工作流——14 個 Agent 組成的虛擬開發團隊](/tech/07-ai-workflow-harness/)*
