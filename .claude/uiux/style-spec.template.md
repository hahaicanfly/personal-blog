# Style Spec Template

> **每個畫面實作前必須填寫此規格，經 UIUX Agent 評審後才可進入實作。**
>
> 複製此模板，填入具體內容，存為 `design-system/pages/{screen-name}.md`

---

## Screen: [畫面名稱]

### 基本資訊

| 項目 | 內容 |
|-----|------|
| **Screen ID** | `screen_xxx` |
| **所屬流程** | 例：點餐流程 |
| **前置畫面** | 例：HomeScreen |
| **後續畫面** | 例：MenuListScreen |
| **設計師** | @uiux-agent |
| **開發者** | 待指派 |
| **狀態** | 草圖 / 評審中 / 核准 / 實作中 / 完成 |

---

## 1. 使用者目標

### 主要目標
用戶來到這個畫面想要達成什麼？
- [ ] 目標 1：_______
- [ ] 目標 2：_______

### 核心操作（Primary Action）
用戶最可能/最應該執行的操作是什麼？

| 操作 | 優先級 | 觸發方式 |
|-----|-------|---------|
| ______ | Primary | 按鈕 / 滑動 / 點擊 |
| ______ | Secondary | |
| ______ | Tertiary | |

### 成功指標
- 用戶在 ___ 秒內完成 ___
- 錯誤率低於 ___%
- 放棄率低於 ___%

---

## 2. Layout（區塊結構）

### 視覺結構圖（ASCII Wireframe）

```
┌─────────────────────────────────┐
│        [Top App Bar]            │  ← 固定 / 滾動
├─────────────────────────────────┤
│                                 │
│        [Hero / Header]          │  ← 區塊 A
│                                 │
├─────────────────────────────────┤
│                                 │
│        [Main Content]           │  ← 區塊 B（可滾動）
│                                 │
│                                 │
├─────────────────────────────────┤
│        [Bottom Action]          │  ← 固定底部
└─────────────────────────────────┘
```

### 區塊定義

| 區塊 | 內容 | 高度/比例 | 是否固定 |
|-----|------|---------|---------|
| Top App Bar | 標題、返回、操作 | 56dp / 64dp | 固定 |
| Hero | | | |
| Main Content | | flex | 可滾動 |
| Bottom Action | | 80dp | 固定 |

### 主要區域說明

**區塊 A：[名稱]**
- 用途：______
- 包含元件：______
- 特殊行為：______

**區塊 B：[名稱]**
- 用途：______
- 包含元件：______
- 特殊行為：______

---

## 3. Components（元件清單）

### 元件列表

| 元件 | 類型 | 狀態 | Props / 參數 |
|-----|-----|-----|-------------|
| `TopAppBar` | 導航 | default, scrolled | title, onBack, actions |
| `MenuItemCard` | 卡片 | default, selected, disabled, loading | item, onSelect, quantity |
| `PrimaryButton` | 按鈕 | default, hover, pressed, disabled, loading | label, onClick, enabled |
| | | | |

### 各元件狀態詳述

#### 元件：`[元件名稱]`

| 狀態 | 視覺表現 | 觸發條件 |
|-----|---------|---------|
| Default | 背景 surface, 文字 onSurface | 初始 |
| Hover | 背景 surfaceVariant | 滑鼠移入 |
| Pressed | scale 0.98, 背景變深 | 點擊中 |
| Selected | 邊框 primary, 背景 primaryContainer | 選中後 |
| Disabled | opacity 0.5, 無互動 | enabled=false |
| Loading | 內容替換為 spinner | isLoading=true |

---

## 4. Design Tokens

### 顏色

| Token | 色碼 | 用途 |
|-------|------|-----|
| `background` | #FFFBF5 | 頁面背景 |
| `surface` | #FFFFFF | 卡片背景 |
| `primary` | #E85D04 | 主要操作、強調 |
| `secondary` | #2D6A4F | 次要操作 |
| `onSurface` | #1C1B1F | 主要文字 |
| `onSurfaceVariant` | #49454F | 次要文字 |
| `error` | #B3261E | 錯誤狀態 |
| `outline` | #79747E | 邊框 |

### 字體

| Token | 大小 | 行高 | 字重 | 用途 |
|-------|------|------|------|-----|
| `headlineLarge` | 32sp | 40sp | Bold | 頁面標題 |
| `titleLarge` | 22sp | 28sp | SemiBold | 區段標題 |
| `titleMedium` | 16sp | 24sp | Medium | 卡片標題 |
| `bodyLarge` | 16sp | 24sp | Regular | 主要內文 |
| `bodyMedium` | 14sp | 20sp | Regular | 次要內文 |
| `labelSmall` | 11sp | 16sp | Medium | 標籤、輔助 |

### 間距

| Token | 值 | 用途 |
|-------|-----|-----|
| `xs` | 4dp | 元素內微間距 |
| `sm` | 8dp | 相關元素間 |
| `md` | 16dp | 標準 padding |
| `lg` | 24dp | 區塊間距 |
| `xl` | 32dp | 主區塊分隔 |
| `xxl` | 48dp | 頁面區段 |

### 圓角（Radius）

| Token | 值 | 用途 |
|-------|-----|-----|
| `sm` | 8dp | 小元件（tag, chip） |
| `md` | 12dp | 按鈕 |
| `lg` | 16dp | 卡片 |
| `xl` | 24dp | 大型容器 |
| `full` | 50% | 圓形 |

### 陰影

| Token | 值 | 用途 |
|-------|-----|-----|
| `elevation.sm` | 2dp | 輕微浮起 |
| `elevation.md` | 4dp | 標準卡片 |
| `elevation.lg` | 8dp | 彈窗、浮動 |
| `elevation.xl` | 16dp | Modal |

---

## 5. Edge Cases

### 空狀態（Empty State）

**觸發條件**：______

**視覺呈現**：
```
┌─────────────────────────────────┐
│                                 │
│         [插圖/圖示]              │
│                                 │
│      [標題：提示訊息]            │
│      [說明：引導文字]            │
│                                 │
│       [ CTA 按鈕 ]              │
│                                 │
└─────────────────────────────────┘
```

**文案**：
- 圖示：______
- 標題：______
- 說明：______
- CTA：______

---

### 錯誤狀態（Error State）

**類型 1：網路錯誤**
- 標題：______
- 說明：______
- 操作：重試 / 離線模式

**類型 2：API 錯誤**
- 標題：______
- 說明：______
- 操作：重試 / 回報問題

**類型 3：驗證錯誤**
- 顯示位置：欄位下方
- 文案樣式：error 色、bodySmall

---

### 長文字處理

| 元素 | 策略 | 最大行數 |
|-----|------|---------|
| 標題 | 截斷 + ellipsis | 2 行 |
| 描述 | 截斷 + ellipsis | 3 行 |
| 價格 | 不截斷，縮小字體 | 1 行 |

---

### 低網速處理

**策略**：
- [ ] Skeleton Loading 預覽佈局
- [ ] 漸進式載入圖片（blur → clear）
- [ ] 操作按鈕顯示 loading 狀態
- [ ] 逾時提示（>10s）

---

### 極端資料

| 情境 | 處理方式 |
|-----|---------|
| 0 筆資料 | 顯示 Empty State |
| 1 筆資料 | 正常顯示 |
| 100+ 筆資料 | LazyColumn + 分頁載入 |
| 超長文字 | 截斷 + tooltip |
| 超大圖片 | 壓縮 + 漸進載入 |
| 無圖片 | 顯示 placeholder |

---

## 6. Acceptance Criteria（驗收項）

### 功能驗收

- [ ] Primary action 可正常執行
- [ ] 所有互動元件有正確的狀態回饋
- [ ] Loading state 正確顯示
- [ ] Error state 有重試機制
- [ ] Empty state 有引導 CTA

### 視覺驗收

- [ ] 顏色來自 Design Token
- [ ] 間距符合 Spacing Scale
- [ ] 字體符合 Typography 層級
- [ ] 圓角/陰影使用 Token

### 可用性驗收

- [ ] 對比度 >= 4.5:1
- [ ] 觸控目標 >= 48dp
- [ ] 鍵盤可完整操作
- [ ] 所有圖示有 contentDescription
- [ ] 尊重 prefers-reduced-motion

### 效能驗收

- [ ] 無 Layout Shift (CLS)
- [ ] 圖片有預留空間
- [ ] 列表使用 Lazy 載入
- [ ] 動畫使用 GPU 加速屬性

---

## 7. 附件

### 參考截圖/設計稿
- [ ] Figma 連結：______
- [ ] 競品參考：______

### 相關文件
- [ ] 用戶故事：______
- [ ] API 規格：______
- [ ] 技術限制：______

---

## 簽核

| 角色 | 姓名/代號 | 日期 | 狀態 |
|-----|----------|------|------|
| UIUX Agent | @uiux-agent | | 草圖 / 評審中 / 核准 |
| 開發者 | | | 待實作 / 實作中 / 完成 |
| 驗收者 | | | 待驗收 / 通過 / 退回 |

---

*模板版本：v1.0*
*最後更新：2026-01-27*
