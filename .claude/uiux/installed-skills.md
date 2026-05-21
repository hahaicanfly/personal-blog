# UI/UX Skills 狀態清單

> 本專案使用 Claude Code 內建的 Skills 系統（`.claude/skills/`），非外部 marketplace。
>
> **注意**：Claude Code v2.1.20 目前沒有 `/plugin` 命令或 marketplace 系統。
> `ccplugins/awesome-claude-code-plugins` 和 `claude-plugins.dev` 並非 Claude Code 官方功能。

---

## 已安裝的 UI/UX 相關 Skills

### 1. ui-ux-pro-max

| 項目 | 值 |
|------|-----|
| **名稱** | ui-ux-pro-max |
| **來源** | 專案內建 (`.claude/skills/ui-ux-pro-max/`) |
| **版本** | N/A（本地 Skill） |
| **安裝狀態** | ✅ 已安裝 |
| **觸發指令** | `/ui-ux-pro-max` 或自動觸發（design, UI, UX 相關關鍵詞） |

**功能特點**：
- 50+ UI 風格（glassmorphism, minimalism, brutalism, neumorphism...）
- 97 種色彩調色盤
- 57 種字體配對
- 99 條 UX 準則
- 25 種圖表類型
- 支援 9 種技術棧（React, Vue, Next.js, SwiftUI, Flutter, Jetpack Compose...）
- Python CLI 搜尋工具：`python3 .claude/skills/ui-ux-pro-max/scripts/search.py`

**使用方式**：
```bash
# 產生完整設計系統
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "food menu restaurant" --design-system -p "MaiNeu"

# 搜尋特定領域
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "accessibility" --domain ux
```

---

### 2. frontend-design

| 項目 | 值 |
|------|-----|
| **名稱** | frontend-design |
| **來源** | 專案內建 (`.claude/skills/frontend-design/`) |
| **版本** | N/A（本地 Skill） |
| **安裝狀態** | ✅ 已安裝 |
| **觸發指令** | `/frontend-design` |

**功能特點**：
- 基於 Anthropic 官方 Frontend Aesthetics Cookbook
- 適配 Compose Multiplatform / Kotlin
- 設計哲學：獨特性、藝術指導、細節執著
- 五大核心原則：Typography, Color, Motion, Spatial Composition, Visual Details
- 禁止通用字體、俗套配色、可預測佈局

**使用方式**：
```
/frontend-design [元件名稱或畫面描述]
```

---

## 已安裝的 UI/UX 相關 Agents

### 1. ui-ux-designer

| 項目 | 值 |
|------|-----|
| **名稱** | ui-ux-designer |
| **來源** | 專案內建 (`.claude/agents/ui-ux-designer.md`) |
| **模型** | opus |
| **觸發詞** | UI、UX、設計、界面、畫面、流程 |

**職責**：
- 用戶流程設計
- 界面規劃
- 設計系統維護
- 互動設計定義

---

## 如何使用

### 方法 1：直接呼叫 Skill
```
/ui-ux-pro-max
/frontend-design
```

### 方法 2：使用 Agent（在對話中）
提及觸發詞即可自動啟用：
- "幫我設計 UI"
- "這個畫面的 UX 流程..."
- "界面設計建議"

### 方法 3：使用 Python CLI
```bash
cd /Users/a17/AIproject/MaiNeu
python3 .claude/skills/ui-ux-pro-max/scripts/search.py --help
```

---

## 未來擴充

如果 Claude Code 未來支援 marketplace：
- 可考慮安裝社群 UI/UX Skills
- 當前本地 Skills 已能滿足大部分需求

---

*最後更新：2026-01-27*
*Claude Code 版本：2.1.20*
