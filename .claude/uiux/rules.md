# UI/UX 設計規則

> **本規則為強制性指引，所有 UI 實作必須遵守。**
>
> 適用技術棧：Compose Multiplatform (Kotlin)、React/Tailwind、SwiftUI

---

## 1. 風格與一致性

### 1.1 Design Tokens 強制使用

```kotlin
// ✅ 正確：使用 Design Token
Text(
    color = MaterialTheme.colorScheme.primary,
    style = MaterialTheme.typography.bodyLarge
)
padding(Spacing.md)  // 16.dp

// ❌ 錯誤：硬編碼數值
Text(color = Color(0xFF6750A4))
padding(16.dp)  // 直接寫數字
```

**規則**：
- 顏色：只使用 `MaterialTheme.colorScheme.*` 或專案定義的 `AppColors.*`
- 字體：只使用 `MaterialTheme.typography.*` 或 `MenuTextStyles.*`
- 間距：只使用 `Spacing.xs/sm/md/lg/xl/xxl`（4dp 倍數）
- 圓角：只使用 `AppShapeTokens.*`

### 1.2 元件一致性

| 元件類型 | 要求 |
|---------|-----|
| 按鈕 | 統一使用 `AnimatedButton` 或 Material3 Button |
| 卡片 | 統一圓角 `16.dp`，陰影 `2-8.dp` |
| 輸入框 | 統一樣式，包含 label 和 error state |
| 圖示 | 只用一套圖示庫（Material Icons / Heroicons） |

### 1.3 禁止 Random Styling

```kotlin
// ❌ 禁止：隨機顏色、隨機圓角
RoundedCornerShape(13.dp)  // 為什麼是 13？
Color(0xFFABCDEF)  // 沒有意義的顏色

// ✅ 正確：使用語義化 Token
RoundedCornerShape(AppShapeTokens.card)
AppColors.Primary
```

---

## 2. 排版與留白

### 2.1 Grid 系統

```kotlin
// 遵循 8dp Grid（4dp 微調）
padding(horizontal = Spacing.lg)  // 24.dp
padding(vertical = Spacing.md)    // 16.dp
```

**Mobile 佈局**：
- 邊距：`16dp`（Spacing.md）
- 卡片間距：`12dp`（Spacing.sm + Spacing.xs）
- 區塊間距：`24dp`（Spacing.lg）

**Desktop 佈局**：
- 最大內容寬度：`1200dp`
- 邊距：`24-48dp`

### 2.2 Spacing Scale（強制）

| Token | 值 | 用途 |
|-------|-----|-----|
| `Spacing.xs` | 4dp | 元素內部微間距 |
| `Spacing.sm` | 8dp | 相關元素間距 |
| `Spacing.md` | 16dp | 區塊內間距、標準 padding |
| `Spacing.lg` | 24dp | 區塊間距 |
| `Spacing.xl` | 32dp | 主要區塊分隔 |
| `Spacing.xxl` | 48dp | 頁面區段分隔 |

### 2.3 文字階層

```kotlin
// 標題層級（不可跳級）
MaterialTheme.typography.displayLarge   // Hero 標題
MaterialTheme.typography.headlineLarge  // 頁面標題 (H1)
MaterialTheme.typography.headlineMedium // 區段標題 (H2)
MaterialTheme.typography.titleLarge     // 子區段 (H3)
MaterialTheme.typography.titleMedium    // 卡片標題
MaterialTheme.typography.bodyLarge      // 主要內文
MaterialTheme.typography.bodyMedium     // 次要內文
MaterialTheme.typography.labelSmall     // 輔助說明
```

**規則**：
- 每個層級大小差距至少 2-4sp
- 行高：body 1.5-1.75，heading 1.2-1.3
- 每行最多 65-75 字元（中文 35-40 字）

---

## 3. 互動細節

### 3.1 狀態完整性（必須實作所有狀態）

| 狀態 | 視覺變化 | 實作要求 |
|-----|---------|---------|
| **Default** | 基準樣式 | 必須定義 |
| **Hover** | 背景淺色、微微放大 | `Modifier.hoverable()` |
| **Focus** | 可見的 focus ring (2dp) | 鍵盤 accessible |
| **Active/Pressed** | 深色背景、scale 0.98 | `Modifier.clickable()` |
| **Disabled** | 50% opacity、無互動 | `enabled = false` |
| **Loading** | 內容替換為 spinner | 禁用互動 |

```kotlin
// Compose 實作範例
@Composable
fun InteractiveCard(
    enabled: Boolean = true,
    isLoading: Boolean = false,
    onClick: () -> Unit
) {
    val interactionSource = remember { MutableInteractionSource() }
    val isPressed by interactionSource.collectIsPressedAsState()

    Card(
        modifier = Modifier
            .scale(if (isPressed) 0.98f else 1f)
            .alpha(if (enabled) 1f else 0.5f)
            .clickable(
                enabled = enabled && !isLoading,
                interactionSource = interactionSource,
                indication = null  // 自訂 indication
            ) { onClick() }
    ) {
        if (isLoading) {
            CircularProgressIndicator()
        } else {
            // Content
        }
    }
}
```

### 3.2 Loading 狀態

```kotlin
// ✅ 正確：Skeleton + 禁用操作
when (uiState) {
    is Loading -> {
        SkeletonMenuList()  // 骨架屏
        // 所有按鈕 disabled
    }
    is Success -> MenuList(uiState.items)
    is Error -> ErrorState(onRetry = { viewModel.retry() })
}

// ❌ 錯誤：只有 spinner，沒有佈局預告
CircularProgressIndicator()  // 不知道會載入什麼
```

### 3.3 Empty State（必須設計）

每個列表/資料區都必須有 Empty State：

```kotlin
@Composable
fun EmptyState(
    icon: ImageVector,        // 必須：圖示
    title: String,            // 必須：標題
    description: String,      // 必須：說明
    actionLabel: String?,     // 可選：CTA
    onAction: (() -> Unit)?
)
```

---

## 4. 可用性與 a11y（無障礙）

### 4.1 鍵盤可操作（強制）

```kotlin
// 所有互動元素必須可 focus
Modifier.focusable()

// Tab 順序符合視覺順序
// 不要用 focusOrder 打亂順序

// Enter/Space 可觸發
Modifier.onKeyEvent { event ->
    if (event.key == Key.Enter || event.key == Key.Spacebar) {
        onClick()
        true
    } else false
}
```

### 4.2 對比度（WCAG AA）

| 元素 | 最低對比度 |
|-----|-----------|
| 普通文字 (< 18sp) | 4.5:1 |
| 大文字 (>= 18sp bold 或 24sp) | 3:1 |
| 圖示、UI 元素 | 3:1 |

```kotlin
// 檢查：前景色 vs 背景色
// 工具：WebAIM Contrast Checker

// ❌ 錯誤：淺灰文字在白底
Text(color = Color.LightGray)  // 對比不足

// ✅ 正確：使用 semantic color
Text(color = MaterialTheme.colorScheme.onSurface)
```

### 4.3 Aria / Label（強制）

```kotlin
// 所有圖示按鈕必須有 content description
IconButton(onClick = { /* */ }) {
    Icon(
        Icons.Default.Close,
        contentDescription = "關閉"  // 必填
    )
}

// 圖片必須有 alt
Image(
    painter = painterResource(R.drawable.menu),
    contentDescription = "菜單照片"  // 必填（裝飾性圖片用 null）
)

// 表單欄位必須關聯 label
OutlinedTextField(
    value = value,
    onValueChange = { },
    label = { Text("電子郵件") }  // 必填
)
```

### 4.4 觸控目標

```kotlin
// 最小觸控區域 48x48dp
Modifier
    .size(48.dp)
    .clickable { }

// 或用 minimumInteractiveComponentSize
Modifier.minimumInteractiveComponentSize()
```

---

## 5. 效能與視覺穩定

### 5.1 避免 CLS（Cumulative Layout Shift）

```kotlin
// ✅ 正確：預留空間
AsyncImage(
    model = imageUrl,
    modifier = Modifier
        .fillMaxWidth()
        .aspectRatio(16f / 9f)  // 固定比例
)

// ❌ 錯誤：高度不確定
AsyncImage(
    model = imageUrl,
    modifier = Modifier.fillMaxWidth()  // 載入後高度變化 → CLS
)
```

### 5.2 圖片尺寸預留

```kotlin
// 定義固定尺寸或 aspectRatio
Box(
    modifier = Modifier
        .width(120.dp)
        .height(80.dp)
        .background(Color.Gray.copy(alpha = 0.1f))  // placeholder
) {
    AsyncImage(
        model = imageUrl,
        contentScale = ContentScale.Crop
    )
}
```

### 5.3 動畫效能

```kotlin
// ✅ 使用 transform/opacity（GPU 加速）
Modifier
    .graphicsLayer {
        scaleX = animatedScale
        alpha = animatedAlpha
    }

// ❌ 避免動畫 width/height/padding（觸發 layout）
Modifier.width(animatedWidth.dp)  // 效能差
```

### 5.4 Reduced Motion

```kotlin
// 尊重系統設定
val reduceMotion = LocalReducedMotion.current
val animationDuration = if (reduceMotion) 0 else 300

AnimatedVisibility(
    visible = isVisible,
    enter = fadeIn(animationSpec = tween(animationDuration))
)
```

---

## 6. 檢查清單

### 實作前檢查
- [ ] 是否已定義 Design Tokens？
- [ ] 是否使用專案統一的 Spacing Scale？
- [ ] 是否規劃了所有狀態（loading/empty/error）？

### 實作中檢查
- [ ] 顏色是否來自 Theme？
- [ ] 間距是否使用 Token？
- [ ] 互動元素是否有 hover/focus/active/disabled？
- [ ] 圖示按鈕是否有 contentDescription？

### 實作後檢查
- [ ] 對比度是否符合 4.5:1？
- [ ] 鍵盤是否可完整操作？
- [ ] 圖片是否有預留空間？
- [ ] 動畫是否尊重 reduced motion？

---

## 7. 違規處理

| 違規類型 | 嚴重程度 | 處理方式 |
|---------|---------|---------|
| 硬編碼顏色/間距 | 中 | Code Review 退回 |
| 缺少 contentDescription | 高 | 強制修復 |
| 對比度不足 | 高 | 強制修復 |
| 缺少 Loading/Empty State | 中 | 補充實作 |
| CLS 問題 | 中 | 效能修復 |

---

*最後更新：2026-01-27*
*適用技術棧：Compose Multiplatform, React/Tailwind, SwiftUI*
