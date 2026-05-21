---
name: frontend-design
description: High-quality UI design principles and aesthetic guidance for Compose Multiplatform and modern frontend development. Use this skill when designing user interfaces, creating UI components, improving visual design, or establishing design systems. Triggers on tasks involving UI design, component styling, visual aesthetics, design system creation, or when user mentions "UI design", "設計畫面", "美化介面", "component design", "visual design", "design guide".
---

# Frontend Design Skill

高品質前端 UI 設計指南，基於 Anthropic 官方 Frontend Design Skill 並適配 Compose Multiplatform。

## 使用方式

```
/frontend-design [元件名稱或畫面描述]
```

## 設計哲學

你是一位擁有世界級美學品味的設計工程師。設計必須：

- **獨特性**: 永遠不要建立看起來「模板化」或「千篇一律」的 UI
- **藝術指導**: 每個專案都需要清晰、一致的視覺語言
- **細節執著**: 魔鬼藏在細節中，從動效到間距都要精心設計

---

## 核心設計原則

### 1. Typography（字體）

**原則**: 選擇獨特且有個性的字體

| 禁止 | 建議 |
|------|------|
| Arial, Helvetica | 系統字體搭配明確層級 |
| Inter, Roboto (過度通用) | 跨平台可用的特色字體 |
| 預設字體配置 | MaterialTheme.typography 自訂 |

**Compose 實踐**:
```kotlin
// 定義清晰的字體層級
val Typography = Typography(
    headlineLarge = TextStyle(
        fontWeight = FontWeight.Bold,
        fontSize = 32.sp,
        letterSpacing = (-0.5).sp  // 緊湊標題
    ),
    bodyLarge = TextStyle(
        fontSize = 16.sp,
        lineHeight = 24.sp  // 舒適閱讀行高
    )
)
```

### 2. Color & Theme（色彩）

**原則**: 使用 CSS 變數/Theme 維護一致的調色盤

| 禁止 | 建議 |
|------|------|
| 硬編碼顏色值 | 使用 Theme colors |
| 俗套配色 (藍灰白) | 強主色 + 銳利點綴色 |
| 無一致性的隨機顏色 | 明確的色彩系統 |

**Compose 實踐**:
```kotlin
// 定義品牌色彩系統
private val LightColors = lightColorScheme(
    primary = Color(0xFF6750A4),
    secondary = Color(0xFF625B71),
    tertiary = Color(0xFF7D5260),  // 點綴色
    surface = Color(0xFFFFFBFE),
    background = Color(0xFFFFFBFE)
)

// 自訂擴展顏色
val ColorScheme.accent: Color
    get() = Color(0xFFFF6B35)  // 銳利橘色點綴
```

### 3. Motion（動效）

**原則**: 優先高影響動效，而非散亂的微互動

| 禁止 | 建議 |
|------|------|
| 到處都是微動效 | 聚焦在入場/頁面切換 |
| 無意義的彈跳 | 有目的的引導動畫 |
| 分散注意力 | 強化資訊層級 |

**Compose 實踐**:
```kotlin
// 列表項目交錯進場
LazyColumn {
    itemsIndexed(items) { index, item ->
        AnimatedVisibility(
            visible = true,
            enter = fadeIn(
                animationSpec = tween(
                    durationMillis = 300,
                    delayMillis = index * 50  // 交錯延遲
                )
            ) + slideInVertically(
                initialOffsetY = { it / 2 }
            )
        ) {
            ItemCard(item)
        }
    }
}
```

### 4. Spatial Composition（空間構圖）

**原則**: 打破可預測的對稱佈局

| 禁止 | 建議 |
|------|------|
| 完美對稱 | 不對稱佈局創造視覺張力 |
| 元素孤立 | 適度重疊增加層次 |
| 擁擠佈局 | 大量負空間留白 |

**Compose 實踐**:
```kotlin
// 使用負空間創造呼吸感
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(horizontal = 24.dp)  // 充足邊距
) {
    Spacer(modifier = Modifier.height(48.dp))  // 大量頂部留白

    Text(
        text = title,
        style = MaterialTheme.typography.headlineLarge
    )

    Spacer(modifier = Modifier.height(32.dp))  // 區塊間距

    // 內容...
}
```

### 5. Visual Details（視覺細節）

**原則**: 運用漸層、紋理、陰影營造氛圍

| 禁止 | 建議 |
|------|------|
| 純平面色塊 | 微妙漸層增加深度 |
| 無陰影設計 | 適當陰影建立層次 |
| 生硬邊緣 | 精緻圓角和過渡 |

**Compose 實踐**:
```kotlin
// 漸層背景
Box(
    modifier = Modifier
        .fillMaxSize()
        .background(
            brush = Brush.verticalGradient(
                colors = listOf(
                    MaterialTheme.colorScheme.surface,
                    MaterialTheme.colorScheme.surfaceVariant.copy(alpha = 0.3f)
                )
            )
        )
)

// 精緻卡片陰影
Card(
    elevation = CardDefaults.cardElevation(
        defaultElevation = 2.dp,
        hoveredElevation = 8.dp
    ),
    shape = RoundedCornerShape(16.dp)
) { /* ... */ }
```

---

## 禁止事項 (Anti-Patterns)

### 絕對禁止

1. **通用字體**: 不要使用 Arial、Helvetica、預設 sans-serif
2. **俗套配色**: 避免千篇一律的藍灰白商務風
3. **可預測佈局**: 不要只用對稱置中的樣板佈局
4. **模板感設計**: 每個設計都必須有獨特的藝術指導

### 警告標誌

如果你的設計看起來像：
- Bootstrap/Material 預設樣式 → **重新設計**
- 任何人都能想到的佈局 → **更有創意**
- 沒有視覺焦點 → **建立層級**

---

## MaiNeu 專案適用指南

### 品牌定位

- **核心價值**: 讓點餐變簡單、有自信
- **視覺調性**: 現代、清晰、友善、專業
- **目標感受**: 使用者拿出手機會覺得「這 App 好用又好看」

### 設計規範

```kotlin
// 專案色彩系統
object MaiNeuColors {
    val Primary = Color(0xFF6750A4)     // 主色：柔和紫
    val Accent = Color(0xFFFF6B35)      // 點綴：活力橘
    val Success = Color(0xFF4CAF50)     // 成功：清新綠
    val Surface = Color(0xFFFFFBFE)     // 表面：乾淨白
    val OnSurface = Color(0xFF1C1B1F)   // 文字：深灰黑
}

// 間距系統
object Spacing {
    val xs = 4.dp
    val sm = 8.dp
    val md = 16.dp
    val lg = 24.dp
    val xl = 32.dp
    val xxl = 48.dp
}

// 圓角系統
object Radius {
    val sm = 8.dp
    val md = 12.dp
    val lg = 16.dp
    val xl = 24.dp
}
```

### 元件設計原則

| 元件 | 設計要點 |
|------|----------|
| MenuItemCard | 清晰的原文/翻譯對照、價格突出、勾選狀態明確 |
| HomeScreen | 簡潔的 CTA、友善的空狀態設計 |
| OrderDisplayScreen | 大字清晰、適合展示給服務生、雙語並列 |
| LoadingOverlay | 有趣的載入動畫、清晰的進度提示 |

---

## 設計審查清單

在完成設計前，檢查以下項目：

```
□ 字體是否有明確層級？標題與內文對比足夠？
□ 色彩是否使用 Theme 系統？有無硬編碼顏色？
□ 關鍵操作有無適當的動效引導？
□ 佈局是否有視覺焦點？留白是否充足？
□ 細節是否到位？圓角、陰影、過渡是否精緻？
□ 整體是否有獨特的藝術指導？還是看起來像模板？
```

---

## 參考資源

- [Anthropic Frontend Aesthetics Cookbook](https://github.com/anthropics/claude-cookbooks/blob/main/coding/prompting_for_frontend_aesthetics.ipynb)
- [Material Design 3](https://m3.material.io/)
- [Compose Multiplatform 官方文檔](https://www.jetbrains.com/lp/compose-multiplatform/)

---

*此 Skill 基於 Anthropic 官方 Frontend Design Skill 並適配 Compose Multiplatform/Kotlin 技術棧*
