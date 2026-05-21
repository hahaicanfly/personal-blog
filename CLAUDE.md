# personal-blog — MaiNeu 開發紀實

技術部落格，記錄獨立開發 MaiNeu（AI 菜單翻譯 App）的全過程。

## Tech Stack

| 層級 | 技術 |
|------|------|
| 靜態網站 | Hugo (extended) |
| 主題 | PaperMod |
| 部署 | Cloudflare Pages + Wrangler（GitHub Actions，~25s） |
| 網域 | blog.maineu.com |
| 內容 | Markdown，雙系列（`content/tech/` 技術版、`content/general/` 一般讀者版） |

## 客製樣式

- 主題樣式覆寫放在 `assets/css/extended/custom.css`（Hugo 會 merge 在 PaperMod 之上）。
- 設計語言：溫暖琥珀強調色（呼應菜單／食物）、卡片化文章列表、CJK 友善排版、柔和動效（尊重 `prefers-reduced-motion`）。
- 顏色變數集中在 `:root` 與 `.dark`，改色只需動 `--accent` 系列變數。

## 常用指令

```bash
hugo server          # 本機預覽（含 draft 用 -D）
hugo --gc --minify   # 產生 public/
```

## 設計／前端 AI 團隊（從 Menu-Superpower 搬入）

### Agents（`.claude/agents/`）
- `ui-ux-designer` — 界面設計、用戶流程、設計規範
- `uiux-agent` — UI 草圖／評審專責，走「草圖 → 評審 → 實作」三階段（見 `.claude/uiux/WORKFLOW.md`），不直接寫 production code

### Skills（`.claude/skills/`）
| Skill | 用途 |
|-------|------|
| `ui-ux-pro-max` | 完整設計系統生成（50 風格、21 調色盤、50 字體配對、9 stacks） |
| `frontend-design` | 高品質 UI 設計原則與美學指導 |
| `web-design-guidelines` | 逐行檢查 a11y 與 Web 介面規範 |
| `tailwind-v4-shadcn` | Tailwind v4 + shadcn/ui 設定模式 |
| `vercel-react-best-practices` | React/Next.js 效能規則 |
| `vercel-composition-patterns` | React 組合模式 |
| `next-best-practices` | Next.js App Router 慣例 |
| `next-cache-components` | Next.js 16 Cache Components |
| `next-upgrade` | Next.js 版本升級遷移 |

### UI/UX 三階段工作流（`.claude/uiux/`）
所有較大的畫面改動建議走：**草圖（ASCII wireframe）→ 評審（3 個方向）→ 實作**。
參考 `WORKFLOW.md`、`rules.md`、`prompt-templates.md`、`style-spec.template.md`。

> 註：Vercel React/Next 系列 skill 對 Hugo 靜態站非必要，作為日後若改用 React 技術棧或撰寫前端教學文章時的通用工具箱保留。

## Git

- 禁止直接 commit 到 `main`，一律 feature 分支 + PR。
- Atomic commits，commit message 用英文。
