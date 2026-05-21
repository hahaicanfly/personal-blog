---
title: "設計 AI 工作流——14 個 Agent 組成的虛擬開發團隊"
date: 2026-05-21
weight: 7
series: ["MaiNeu 開發旅程"]
tags: ["ai", "claude", "agent", "workflow", "harness-engineering", "productivity"]
categories: ["技術版"]
description: "ExecPlan 跨 session 一致性、14 個角色分明的 Agent 體系、pre-tool-use-guard 的 60 條 deny pattern、Invariants 作為機械化驗證——Harness Engineering 的完整實踐"
showToc: true
TocOpen: false
draft: false
---

# 設計 AI 工作流——14 個 Agent 組成的虛擬開發團隊

*MaiNeu 開發旅程 第七篇*

---

## 從「幫我寫代碼」到「設計工作軌道」

2026 年初開始做 MaiNeu 的時候，我使用 AI 的方式很初級：

「幫我寫一個 ViewModel，需要管理這些狀態。」

然後 Claude 給我一段代碼，我貼上去，繼續下一個任務。AI 是一個非常快的代碼補全工具。

五個月後，MaiNeu 的 AI 工作流長這樣：

```
.claude/
├── agents/      # 14 個角色分明的 AI agent
├── skills/      # 28 個可複用的工作流 skill
├── hooks/       # 5 個自動化守衛
├── protocols/   # 4 個標準化流程
└── rules/       # 5 個強制約束
```

---

## 第一個問題：跨 session 的一致性

MaiNeu 的代碼庫在三個月後變得相當大。有一天，我開始一個新的 session，試圖告訴 Claude 當前的狀態：「上週我做了 X，現在要做 Y，記得 Z 這個限制。」

然後我意識到：Claude 沒有「記憶」——每個 session 都是全新的。

**如果 AI 每次都要從零開始理解上下文，跨 session 的長期工作就非常低效。** 更糟糕的是，這次 session 的 AI 和上次的 AI 可能做出互相衝突的架構決策。

---

## 解法一：ExecPlan

把跨 session 的工作寫成一份結構化的計劃文件，存進 repo，讓每個 session 的 AI 都從這份計劃出發：

```markdown
## Goal
[這個任務要達成什麼]

## Context
[為什麼要做這個，背景是什麼]

## Constraints
[不能做什麼，有什麼限制]

## Step-by-step
1. [步驟一]
2. [步驟二]

## Verification
[如何確認完成，測試什麼]

## Progress Log
[已完成的步驟]

## Decision Log
[過程中做了什麼決定，為什麼]
```

ExecPlan 有一個 10 階段的 lifecycle：

```
PROPOSED → PLANNED → APPROVED → IN_PROGRESS
         → VERIFYING → REVIEWING → DONE
                                 → BLOCKED / REJECTED
```

不同 session 的 AI 都在同一份計劃的軌道上跑，不會各自做各自的事。

---

## 第二個問題：「一個 AI 做所有事」

有了 ExecPlan，跨 session 一致性解決了。但另一個問題出現了：

一個 session 的 AI 需要先寫 code，然後做 review，然後更新文件——每個角色需要不同的視角。結果是 AI 在自己 review 自己的代碼，這不比沒有 review 好多少。

---

## 解法二：角色分明的 Agent 體系

把不同的角色拆成不同的 Agent，每個 Agent 有自己的視角：

| Agent | 視角 | 不做什麼 |
|-------|------|---------|
| `pm` | 用戶需求、商業邏輯 | 不寫代碼 |
| `architect` | 系統設計、模組邊界 | 不關心 UI 細節 |
| `code-reviewer` | 規範符合性、潛在 bug | 不關心業務需求 |
| `security-reviewer` | 攻擊面、安全漏洞 | 不關心 UX |
| `qa-engineer` | 邊界案例、可測試性 | 不實作功能 |

當一個功能完成後：

1. `code-reviewer` 審查是否符合規範
2. `security-reviewer` 審查是否有安全問題
3. `qa-engineer` 審查是否有遺漏的測試案例

三個 Agent 獨立審查，各自從自己的角度發現問題。

---

## 解法三：Hook——防止 AI 犯不可逆的錯誤

AI 可能犯非常基本的錯誤——在 `master` 分支上直接 commit、刪除生產環境資料庫。它沒有「後果意識」。

`pre-tool-use-guard.py` 在 AI 執行任何 Bash 命令之前攔截：

```python
DANGEROUS_PATTERNS = [
    (r'git push.*(--force|-f)', "Force push is blocked"),
    (r'git commit.*', lambda cmd: check_current_branch()),
    (r'cat.*\.env', "Reading .env files is blocked"),
    (r'curl.*\|.*sh', "Remote execution is blocked"),
]
```

有趣的是：第一版的 guard 在 security review 之後發現了 10 個以上的 bypass 路徑：

- `--force` 被擋了，但 `+master`（git push origin +master）沒有
- `cat .env` 被擋了，但 `source .env` 沒有

修完之後，deny pattern 從 10 條膨脹到約 60 條。

**安全設計原則：安全規則的有效性等於「最寬鬆的 bypass 路徑」。有 hook 但有 bypass 的 hook，比沒有 hook 更危險——因為它讓你以為安全了。**

---

## 解法四：Skill——把工作流封裝成可複用命令

MaiNeu 有 28 個 Skill，最有代表性的是 `/feature-pipeline`：

```
/feature-pipeline <feature名稱>

觸發順序：
1. pm agent 分析需求
2. architect agent 設計系統
3. 開發（實作代碼）
4. code-reviewer agent 審查
5. security-reviewer agent 安全審查
6. 更新文件
```

一個命令，觸發整條功能開發流水線。每個環節有明確的輸入和輸出，交接用 `[HANDOFF]` 標記。

---

## 解法五：Invariants——把規則變成機械化驗證

不是：「記得 CancellationException 要 rethrow」（靠記憶，會忘）

而是：「`INV-COR-001`：所有 `catch (e: Exception)` 區塊前必須有 `catch (e: CancellationException) { throw e }`。違反此規則的 PR 不得 merge。」

Invariant 可以被 code-reviewer agent 機械化檢查。不需要靠「有經驗的工程師記得這條規則」，而是靠「自動化工具每次都會檢查」。

MaiNeu 有 13 類、77+ 條 Invariant，覆蓋從協程安全到 Git 操作的各個面向。

---

## 設計 AI 工作流的本質認知

**AI 最擅長的是「執行定義好的流程」。人最擅長的是「定義流程和判斷邊界案例」。**

早期模式：我描述問題 → AI 想辦法解決 → 我評估結果

Harness Engineering 模式：我定義流程（ExecPlan + Invariants + Skills）→ AI 在流程上執行 → 流程本身保證品質

兩者的差異不只是效率，而是**可靠性**。當 AI 在定義好的軌道上跑，它的輸出是可預期的、可審查的、可追溯的。

---

## 一個反直覺的發現

**Hook 的第一版永遠是不夠的。**

這不是因為設計不認真，而是「守衛」和「攻擊」本質上是對抗性的——你只能堵住你能想到的 bypass，但實際上的 bypass 路徑比你能想到的多。

解法不是「設計出完美的第一版」，而是接受「第一版會有漏洞，要有機制讓漏洞被發現並修補」。

---

*下一篇：[Phase 2 的故事——從 WebSocket 到「暫不做」的產品決策](/tech/08-phase2-story/)*
