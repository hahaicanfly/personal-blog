---
title: "Compose 深坑錄——我在 Jetpack Compose 踩過的那些坑"
date: 2026-05-21
weight: 2
series: ["MaiNeu 開發旅程"]
tags: ["android", "jetpack-compose", "coroutines", "kotlin", "debugging"]
categories: ["技術版"]
description: "LaunchedEffect key 語義、CancellationException 必須 rethrow、FlowRow+weight 版面炸裂——7 個真實踩過的 Compose/Coroutines 坑"
showToc: true
TocOpen: false
draft: false
---

# Compose 深坑錄——我在 Jetpack Compose 踩過的那些坑

*MaiNeu 開發旅程 第二篇*

---

## 「我以為我會了」

Jetpack Compose 我在做 MaiNeu 之前就用過。幾個小型的 side project，感覺還行——聲明式 UI 很直覺，`remember` 和 `State` 的概念也不難懂。

然後 MaiNeu 讓我知道，「用過」和「真正理解」之間有多遠。

這篇記錄我在 MaiNeu 裡真實踩過的 Compose 和 Coroutines 坑——每一個都是真實的 bug、真實的卡關、真實的「啊原來是這樣」。

---

## 坑一：LaunchedEffect 的 key，我根本沒有真正理解

### 症狀

MaiNeu 有一個菜單解析流程：用戶拍完照片，App 開始解析，中間有 loading state，解析完成後顯示結果。有一段時間，這個流程出現了一個奇怪的 bug：

**用戶快速連拍兩張照片，第一張的解析結果會覆蓋第二張的 loading state，UI 最後顯示的是第一張的結果，但狀態欄顯示「解析中」。**

### 原因

我的代碼大概長這樣：

```kotlin
@Composable
fun ParsingScreen(photoPath: String, ...) {
    LaunchedEffect(Unit) {  // 問題在這裡
        viewModel.startParsing(photoPath)
    }
}
```

`LaunchedEffect(Unit)` 的意思是：這個 Effect 只在 Composable 第一次進入 composition 時執行一次，之後即使 `photoPath` 改變，它也不會重新執行。

但我以為 `Unit` 是「總是執行」——這完全搞反了。

### 正確理解

`LaunchedEffect` 的 key 語義是：

- **key 不變 → Effect 不重啟**（舊的協程繼續跑）
- **key 改變 → 取消舊協程 + 啟動新協程**

`Unit` 永遠不變，所以 `LaunchedEffect(Unit)` 只會執行一次。

正確寫法應該是：

```kotlin
LaunchedEffect(photoPath) {  // photoPath 改變時，取消舊解析、開始新解析
    viewModel.startParsing(photoPath)
}
```

這樣每次 `photoPath` 改變，LaunchedEffect 就會自動取消前一個正在進行的解析協程，並重新開始。**不需要手動管理協程的 cancel——Compose 幫你做了。**

### 反思

這個坑讓我真正理解了「key 改變 = cancellation + 重啟」不是文件裡的一句話，而是一個必須用身體記住的行為規則。後來我把這條規則寫進了 MaiNeu 的 Invariants 文件（`INV-COR-003`），防止以後再犯。

---

## 坑二：CancellationException 必須 rethrow

### 症狀

這是一個更隱藏、更危險的 bug。

MaiNeu 的解析流程用 Kotlin Coroutines 寫，中間有多個 suspend 呼叫：上傳圖片、呼叫 API、解析 JSON。我用了 `runCatching` 包起來處理錯誤：

```kotlin
viewModelScope.launch {
    val result = runCatching {
        apiClient.parseMenu(photoBytes)
    }
    result.fold(
        onSuccess = { data -> updateUiWithResult(data) },
        onFailure = { error -> updateUiWithError(error) }
    )
}
```

**問題：用戶離開頁面時，UI 會卡在 loading 狀態。**

### 原因

當用戶離開頁面，`viewModelScope` 被取消，`parseMenu` 的協程拋出 `CancellationException`。這個 exception 進入了 `runCatching` 的 catch 路徑，然後被 `result.fold { onFailure = { error -> ... } }` 當成普通錯誤處理。

`onFailure` 裡，我把 `error` 顯示成一個錯誤訊息，然後把 `isLoading` 設回 `false`。

但問題是：**協程的取消機制依賴 `CancellationException` 的傳播**。如果你在 `onFailure` 裡把它吃掉（當作普通錯誤處理），協程不知道自己被取消了，會繼續執行後續邏輯。

### 正確寫法

```kotlin
result.fold(
    onSuccess = { data -> updateUiWithResult(data) },
    onFailure = { error ->
        // CancellationException 必須重新拋出，讓協程正確取消
        if (error is CancellationException) throw error
        updateUiWithError(error)
    }
)
```

或者更安全的模式：

```kotlin
viewModelScope.launch {
    try {
        val data = apiClient.parseMenu(photoBytes)
        updateUiWithResult(data)
    } catch (e: CancellationException) {
        throw e  // 永遠不要吞掉這個
    } catch (e: Exception) {
        updateUiWithError(e)
    }
}
```

### 反思

這個 bug 最危險的地方是：它不一定表現為 crash。它可能只是 UI 狀態不對、loading 消不掉、或者記憶體洩漏。協程的取消機制是整個 Kotlin Coroutines 設計裡最精妙、也最容易誤用的部分。`CancellationException` 必須 rethrow——這條規則後來成了 MaiNeu 最重要的 Invariant 之一（`INV-COR-001`）。

---

## 坑三：`@Volatile` 不觸發 Recompose

### 症狀

我在 ViewModel 裡有一個 `@Volatile` 的 Boolean 欄位，用來控制某個 UI 元素的顯示。我在另一個地方更新了這個值，但 UI 沒有重新渲染。

### 原因

Compose 的 recomposition 機制依賴 **Compose 能觀察的狀態**——也就是 `State<T>`、`StateFlow` 或 `MutableState`。`@Volatile` 只是一個 JVM 關鍵字，確保多執行緒的可見性，但 Compose 系統根本不知道這個值改變了。

### 正確做法

```kotlin
// ❌ 不要這樣
@Volatile var isFeatureEnabled = false

// ✅ 應該這樣
private val _isFeatureEnabled = MutableStateFlow(false)
val isFeatureEnabled = _isFeatureEnabled.asStateFlow()
```

在 Composable 裡：

```kotlin
val isFeatureEnabled by viewModel.isFeatureEnabled.collectAsStateWithLifecycle()
```

---

## 坑四：FlowRow + weight(1f) 的版面炸裂

### 症狀

MaiNeu 的菜單解析結果頁面，每道菜顯示為一個卡片。我想用 `FlowRow` 讓卡片自動換行，每張卡片設定 `weight(1f)` 讓它們等寬。結果：

**所有卡片都佔據整行寬度，完全沒有並排。**

### 原因

`FlowRow` 的換行決策是在 intrinsic size measurement 階段做的，這個階段它還不知道 `weight` 會怎麼分配空間。所以它看到每個 item 的「本來大小」，決定要不要換行，但 `weight(1f)` 的效果要到 layout 階段才生效——這時候換行已經決定好了。

### 解法

```kotlin
// ❌ FlowRow + weight 組合不可靠
FlowRow {
    items.forEach { item ->
        MenuItemCard(modifier = Modifier.weight(1f))
    }
}

// ✅ 手動分行，行為可靠
val chunks = items.chunked(2)
Column {
    chunks.forEach { rowItems ->
        Row {
            rowItems.forEach { item ->
                MenuItemCard(modifier = Modifier.weight(1f))
            }
            if (rowItems.size == 1) {
                Spacer(modifier = Modifier.weight(1f))
            }
        }
    }
}
```

---

## 坑五：AnimatedContent 裡的 remember 會重置

當 `AnimatedContent` 切換 content，原本的 Composable 會離開 composition，`remember` 的值也跟著消失。如果你希望 state 跨越 Screen 切換保留，需要把 state 提升到 ViewModel，或使用 `rememberSaveable`。

---

## 坑六：painterResource 不支援 adaptive icon

```
IllegalArgumentException: Only VectorDrawables and rasterized asset types are supported
```

`mipmap-anydpi-v26` 下的 adaptive icon XML 不是可繪製資源。解法：用 `ic_launcher_foreground`（具體的 PNG）加上 `CircleShape` clip。

---

## 坑七：PaddingValues 的三個 overload 不能混用

```kotlin
// ❌ 這會編譯錯誤
ContentPadding(horizontal = 16.dp, top = 8.dp, bottom = 12.dp)

// ✅ horizontal 對稱但 top/bottom 不同時，用 4 參數版本
PaddingValues(start = 16.dp, top = 8.dp, end = 16.dp, bottom = 12.dp)
```

---

## 一個通用的教訓

**這些 bug 在教程裡根本不會出現，因為教程的場景太簡單了。**

真實產品才是最好的老師——不是因為它更難，而是因為它有真實的邊界案例：用戶會快速操作、網路會斷線、螢幕會旋轉、用戶會同時有多個頁面開著。

每一個 bug 都讓我對 Compose 和 Coroutines 的理解再深一層。現在這些規則不是我背的，而是我用身體記住的。

---

*下一篇：[後端從零開始——一個 Android 工程師如何讀懂 Cloudflare Workers](/tech/03-backend-from-scratch/)*
