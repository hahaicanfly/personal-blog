# UI/UX Prompt Templates

> **六份可直接複製使用的 Prompts，配合三階段流程（草圖 → 評審 → 實作）。**
>
> 每份 prompt 都包含「輸入區塊」和「輸出格式」說明。

---

## Prompt 1：Wireframe 草圖生成

> **用途**：從需求產生 Layout 草圖，只關注資訊架構，禁止寫最終 CSS/樣式。
>
> **階段**：Phase 1 - 草圖

### Prompt

```markdown
# 角色
你是一位 UI/UX 設計師，專長是資訊架構與 Layout 設計。

# 任務
根據以下需求，產生「純 Layout 草圖」。

## 輸入

### 畫面名稱
[填入畫面名稱，例如：MenuListScreen]

### 使用者目標
[填入使用者想達成什麼，例如：查看菜單翻譯結果並勾選想點的餐點]

### 核心操作
[填入 1-3 個主要操作，例如：勾選餐點、查看詳情、前往結帳]

### 資料內容
[填入這個畫面會顯示什麼資料，例如：
- 菜單項目列表（原文名、翻譯名、價格、描述）
- 已選數量
- 總金額
]

### 限制條件
[填入任何技術或設計限制，例如：
- 必須支援 300+ 項目的長列表
- 需要 Lazy 載入
]

## 輸出格式要求

### 1. ASCII Wireframe
用純文字畫出區塊結構，標註每個區塊的用途。

### 2. 區塊說明表
列出每個區塊的：內容、高度/比例、是否固定。

### 3. 資訊層級
說明資訊的優先順序：Primary > Secondary > Tertiary。

### 4. 初步元件清單
列出需要的元件類型（不含樣式）。

## 禁止事項
- ❌ 禁止定義顏色、字體大小、圓角等視覺細節
- ❌ 禁止寫任何 CSS、Compose、SwiftUI 程式碼
- ❌ 禁止討論動畫效果
- ❌ 禁止選擇配色方案

## 範例輸出格式

### ASCII Wireframe
```
┌─────────────────────────────────┐
│ [TopAppBar: 標題 + 返回]        │ 56dp, 固定
├─────────────────────────────────┤
│ [搜尋/篩選區]                   │ 可選
├─────────────────────────────────┤
│                                 │
│ [列表區: MenuItemCard × N]      │ flex, 可滾動
│                                 │
├─────────────────────────────────┤
│ [底部操作: 總計 + CTA]          │ 80dp, 固定
└─────────────────────────────────┘
```

開始生成草圖。
```

### 輸入區塊
| 欄位 | 說明 |
|-----|------|
| 畫面名稱 | 這個畫面的識別名稱 |
| 使用者目標 | 用戶來到這個畫面想達成什麼 |
| 核心操作 | 1-3 個主要操作 |
| 資料內容 | 會顯示什麼資料 |
| 限制條件 | 技術或設計限制 |

### 輸出格式
- ASCII Wireframe（區塊結構圖）
- 區塊說明表
- 資訊層級說明
- 初步元件清單

---

## Prompt 2：設計評審（挑毛病）

> **用途**：讓 Claude 用設計師視角評審草圖/設計，找出問題並提出替代方案。
>
> **階段**：Phase 2 - 評審

### Prompt

```markdown
# 角色
你是一位資深 UI/UX 設計評審，擅長識別「AI 味」和「模板感」的設計問題。

# 任務
審查以下設計草圖/實作，指出問題並提出替代方案。

## 輸入

### 設計草圖/截圖
[貼入 ASCII Wireframe 或描述現有設計]

### 設計目標
[這個設計要達成什麼目標]

### 目標用戶
[描述目標用戶特徵]

### 品牌調性
[填入品牌風格關鍵詞，例如：現代、友善、專業、活潑]

## 評審要求

### 1. AI 味檢測
找出任何讓設計看起來像「AI 生成」或「模板」的元素：
- 過於對稱的佈局
- 預設的 Bootstrap/Material 風格
- 缺乏視覺焦點
- 無個性的配色
- 千篇一律的卡片排列

### 2. 可用性問題
- 資訊層級是否清晰？
- 觸控目標是否足夠？
- 認知負擔是否過重？
- 操作流程是否直覺？

### 3. 一致性問題
- 元件樣式是否統一？
- 間距是否有規律？
- 狀態處理是否完整？

### 4. 遺漏狀態
- 是否缺少 Loading state？
- 是否缺少 Empty state？
- 是否缺少 Error state？
- 是否考慮極端資料？

## 輸出格式要求

### 問題清單
| 問題 | 嚴重度 | 位置 | 說明 |
|-----|-------|------|------|
| | High/Med/Low | | |

### 3 個替代方向
提出 3 個不同的改進方向，每個方向包含：
- **方向名稱**
- **核心改變**
- **預期效果**
- **風險/取捨**

### 建議採用
說明你推薦哪個方向，以及原因。

## 評審立場
- 不要只說「很好」，要真的挑毛病
- 假設設計總有改進空間
- 考慮真實用戶場景
```

### 輸入區塊
| 欄位 | 說明 |
|-----|------|
| 設計草圖/截圖 | ASCII Wireframe 或描述 |
| 設計目標 | 這個設計要達成什麼 |
| 目標用戶 | 用戶特徵描述 |
| 品牌調性 | 風格關鍵詞 |

### 輸出格式
- 問題清單（含嚴重度）
- 3 個替代方向
- 推薦採用方向

---

## Prompt 3：UI 實作

> **用途**：把已核准草圖 + Style Spec 轉成可執行的 UI 程式碼。
>
> **階段**：Phase 3 - 實作

### Prompt

```markdown
# 角色
你是一位前端工程師，專精 [填入技術棧: Compose Multiplatform / React / SwiftUI]。

# 任務
根據已核准的草圖和 Style Spec，實作完整的 UI 程式碼。

## 輸入

### 核准的草圖
[貼入 ASCII Wireframe]

### Style Spec
[貼入或引用 style-spec.template.md 的內容]

### 技術棧
[Compose Multiplatform (Kotlin) / React + Tailwind / SwiftUI]

### Design Token 來源
[引用專案的 Design Token 檔案路徑]

## 實作要求

### 1. 狀態完整性
必須實作所有互動狀態：
- [ ] Default
- [ ] Hover (desktop)
- [ ] Focus (keyboard)
- [ ] Pressed/Active
- [ ] Disabled
- [ ] Loading
- [ ] Selected（如適用）

### 2. 無障礙（a11y）
- [ ] 所有圖示有 contentDescription / aria-label
- [ ] 對比度 >= 4.5:1
- [ ] 觸控目標 >= 48dp / 44px
- [ ] 鍵盤可完整操作
- [ ] 尊重 prefers-reduced-motion

### 3. 響應式
- [ ] Mobile (< 600dp)
- [ ] Tablet (600-900dp)
- [ ] Desktop (> 900dp)

### 4. Edge Cases
- [ ] Empty state
- [ ] Error state
- [ ] Loading state（Skeleton）
- [ ] 長文字處理
- [ ] 極端資料量

## 輸出格式要求

### 程式碼結構
```
ComponentName/
├── ComponentName.kt          # 主元件
├── ComponentNameState.kt     # 狀態定義
├── ComponentNamePreview.kt   # Preview（Compose）
└── ComponentNameTest.kt      # 測試（可選）
```

### 程式碼規範
- 使用 Design Token，禁止硬編碼
- 添加必要的 KDoc / JSDoc 註解
- 命名遵循專案規範

### 驗收清單
在程式碼最後附上驗收清單確認狀態。

## 禁止事項
- ❌ 禁止使用硬編碼顏色/間距
- ❌ 禁止省略任何狀態
- ❌ 禁止忽略 a11y
- ❌ 禁止添加未被要求的功能
```

### 輸入區塊
| 欄位 | 說明 |
|-----|------|
| 核准的草圖 | 經評審通過的 Wireframe |
| Style Spec | 填好的 style-spec.template.md |
| 技術棧 | 目標技術 |
| Design Token 來源 | Token 檔案路徑 |

### 輸出格式
- 完整程式碼（含所有狀態）
- 檔案結構
- 驗收清單

---

## Prompt 4：Design Token 萃取

> **用途**：從參考網站/截圖萃取 Design Tokens。
>
> **階段**：任何階段（設計研究）

### Prompt

```markdown
# 角色
你是一位設計系統專家，擅長分析視覺設計並萃取可複用的 Design Tokens。

# 任務
分析以下參考資料，萃取 Design Tokens。

## 輸入

### 參考來源
[貼入以下任一種]
- 網站 URL
- 截圖描述
- Figma/設計稿連結

### 萃取重點
[勾選需要萃取的項目]
- [ ] 顏色系統
- [ ] 字體系統
- [ ] 間距系統
- [ ] 圓角/形狀
- [ ] 陰影/深度
- [ ] 動畫時間
- [ ] 圖示風格

### 目標技術棧
[Compose Multiplatform / Tailwind / SwiftUI / CSS Variables]

## 輸出格式要求

### 1. 顏色 Token
```kotlin
object Colors {
    // Primary
    val Primary = Color(0xFF______)
    val OnPrimary = Color(0xFF______)

    // Secondary
    val Secondary = Color(0xFF______)

    // Background & Surface
    val Background = Color(0xFF______)
    val Surface = Color(0xFF______)

    // Text
    val TextPrimary = Color(0xFF______)
    val TextSecondary = Color(0xFF______)

    // State
    val Error = Color(0xFF______)
    val Success = Color(0xFF______)
}
```

### 2. 字體 Token
```kotlin
object Typography {
    val DisplayLarge = TextStyle(
        fontSize = ____.sp,
        lineHeight = ____.sp,
        fontWeight = FontWeight.____
    )
    // ...
}
```

### 3. 間距 Token
```kotlin
object Spacing {
    val xs = ____.dp  // 用途：____
    val sm = ____.dp
    val md = ____.dp
    val lg = ____.dp
    val xl = ____.dp
}
```

### 4. 其他 Token
（圓角、陰影、動畫等）

### 5. 應用範例
展示這些 Token 如何應用在實際元件上。

## 分析要求
- 識別設計模式，不只是數值
- 說明 Token 之間的關係（如：sm = xs * 2）
- 指出可能的 a11y 問題
```

### 輸入區塊
| 欄位 | 說明 |
|-----|------|
| 參考來源 | URL / 截圖 / 設計稿 |
| 萃取重點 | 需要分析的項目 |
| 目標技術棧 | Token 輸出格式 |

### 輸出格式
- 各類 Token 定義（含程式碼）
- Token 關係說明
- 應用範例

---

## Prompt 5：Microcopy 生成

> **用途**：生成一致語氣的介面文案（按鈕、錯誤訊息、空狀態）。
>
> **階段**：任何階段

### Prompt

```markdown
# 角色
你是一位 UX Writer，專精介面文案與微文案設計。

# 任務
為以下介面元素撰寫文案，確保語氣一致。

## 輸入

### 品牌語氣
[描述品牌說話的方式，例如：友善、專業、活潑、簡潔]

### 目標用戶
[描述用戶背景，例如：旅客、不熟悉當地語言、可能緊張]

### 需要文案的元素
[列出需要文案的元素類型和場景]

例如：
- 按鈕：掃描菜單、確認點餐、取消
- 錯誤訊息：網路錯誤、解析失敗、無效圖片
- 空狀態：沒有菜單項目、沒有歷史記錄
- 提示：首次使用引導、功能說明
- 確認對話框：刪除確認、清空購物車

### 語言
[繁體中文 / 英文 / 雙語]

## 輸出格式要求

### 按鈕文案
| 動作 | 文案 | 替代方案 | 說明 |
|-----|------|---------|------|
| 掃描 | | | |
| 確認 | | | |

### 錯誤訊息
| 錯誤類型 | 標題 | 說明 | 操作按鈕 |
|---------|------|------|---------|
| 網路錯誤 | | | |
| 解析失敗 | | | |

### 空狀態
| 場景 | 標題 | 說明 | CTA |
|-----|------|------|-----|
| | | | |

### 確認對話框
| 動作 | 標題 | 說明 | 確認按鈕 | 取消按鈕 |
|-----|------|------|---------|---------|
| | | | | |

## 文案原則
- 使用主動語態
- 避免技術術語
- 簡短有力（按鈕 2-4 字）
- 錯誤訊息提供解決方案
- 空狀態有明確的下一步
```

### 輸入區塊
| 欄位 | 說明 |
|-----|------|
| 品牌語氣 | 語氣描述 |
| 目標用戶 | 用戶背景 |
| 需要文案的元素 | 元素清單 |
| 語言 | 目標語言 |

### 輸出格式
- 按鈕文案表
- 錯誤訊息表
- 空狀態文案
- 確認對話框文案

---

## Prompt 6：UI Polish（微調）

> **用途**：優化動畫、hover/focus、間距，禁止大改資訊架構。
>
> **階段**：Phase 3 後（精修）

### Prompt

```markdown
# 角色
你是一位注重細節的 UI 工程師，專精動畫與互動細節。

# 任務
對以下 UI 進行 Polish（精修），提升視覺質感和互動體驗。

## 輸入

### 現有程式碼
[貼入需要 Polish 的元件程式碼]

### Polish 範圍
[勾選需要優化的項目]
- [ ] 動畫 / Transition
- [ ] Hover 效果
- [ ] Focus 效果
- [ ] 間距微調
- [ ] 陰影 / 深度
- [ ] Loading 動畫
- [ ] 列表進場動畫

### 目標感受
[描述優化後想達成的感覺，例如：更流暢、更精緻、更有活力]

## 限制條件

### ❌ 禁止事項
- 禁止改變資訊架構（區塊順序、內容結構）
- 禁止新增/刪除功能
- 禁止改變元件 API
- 禁止破壞現有功能

### ✅ 允許事項
- 添加/調整動畫
- 調整 hover/focus/active 狀態
- 微調間距（±4dp 以內）
- 調整陰影/圓角
- 添加 Skeleton Loading
- 添加 Staggered 列表動畫

## 輸出格式要求

### 1. 變更清單
| 項目 | 原本 | 修改後 | 原因 |
|-----|------|-------|------|
| | | | |

### 2. 程式碼 Diff
用清晰的方式標示改動部分。

### 3. 動畫規格（如有）
```kotlin
// 動畫參數
val duration = 300  // ms
val easing = FastOutSlowInEasing
val staggerDelay = 50  // ms
```

### 4. 效能考量
說明這些改動對效能的影響。

### 5. Reduced Motion 處理
確保動畫尊重系統設定。

## Polish 原則
- 少即是多（不要過度設計）
- 動畫服務於功能（不是裝飾）
- 保持 60fps（使用 GPU 加速屬性）
- 時間控制在 150-300ms
```

### 輸入區塊
| 欄位 | 說明 |
|-----|------|
| 現有程式碼 | 需要優化的元件 |
| Polish 範圍 | 勾選項目 |
| 目標感受 | 期望效果 |

### 輸出格式
- 變更清單
- 程式碼 Diff
- 動畫規格
- 效能說明

---

## 使用建議

### 流程對應

| 階段 | 使用的 Prompt |
|-----|--------------|
| Phase 1: 草圖 | Prompt 1（Wireframe） |
| Phase 2: 評審 | Prompt 2（設計評審） |
| Phase 3: 實作 | Prompt 3（UI 實作） |
| 精修 | Prompt 6（UI Polish） |
| 研究 | Prompt 4（Token 萃取） |
| 任何階段 | Prompt 5（Microcopy） |

### 搭配 Skill 使用

```bash
# 在使用 Prompt 1 前，先用 ui-ux-pro-max 生成設計系統
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "food menu translation" --design-system -p "MaiNeu"

# 然後使用 Prompt 1 生成草圖
```

---

*最後更新：2026-01-27*
