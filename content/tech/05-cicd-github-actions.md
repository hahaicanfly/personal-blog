---
title: "一個人維護四個環境——GitHub Actions CI/CD 實戰"
date: 2026-05-21
weight: 5
series: ["MaiNeu 開發旅程"]
tags: ["devops", "github-actions", "ci-cd", "cloudflare-workers", "android"]
categories: ["技術版"]
description: "三層環境晉升體系、環境獨立 D1 database 設計、commit SHA 強制指定、PR 沒指定 base 的事故、AI 自動 PR Review 的 false positive"
showToc: true
TocOpen: false
draft: false
---

# 一個人維護四個環境——GitHub Actions CI/CD 實戰

*MaiNeu 開發旅程 第五篇*

---

## 從「知道 CI 是什麼」到「設計三層環境晉升體系」

在做 MaiNeu 之前，我對 CI/CD 的認知是：「有個系統在每次 push 之後自動跑測試，公司有人在管。」

MaiNeu 只有我一個人，沒有 DevOps 工程師，我必須自己設計、配置、維護。

---

## 為什麼需要多個環境

第一次「改壞 production」讓我理解了這件事。我在修一個 API 的 response 格式，本地測試沒問題，推上去之後某個邊界案例在線上出現，破壞了現有用戶的資料呈現。花了一個小時 rollback，又花了兩個小時修正再部署。

那一個小時，App 是壞的。

---

## 三層環境架構

```
開發（本地 / Dev flavor）
    ↓ push → 自動部署
UT（Unit Test 環境，api-ut.maineu.com）
    ↓ 手動核准 + 確認測試通過
Staging（api-staging.maineu.com）
    ↓ 手動核准 + 完整驗收
Production（api.maineu.com）
```

| 環境 | 用途 |
|------|------|
| **UT** | 開發沙盒，資料可以隨意重設 |
| **Staging** | 盡量接近 production，用來做最終驗收 |
| **Production** | 真實用戶，只有充分測試後才能推進 |

### 最重要的原則：每個環境獨立的資料庫

如果 UT 和 Production 共用同一個 D1 database，在 UT 環境做的測試（寫入、刪除、改動 schema）就會直接影響 Production 的資料。更糟糕的是，資料只是靜靜地被污染，沒有任何報錯。

每個環境獨立 database 有代價：需要在每個環境分別執行 migration。但這個代價遠小於「production 資料被污染」的風險。

---

## GitHub Actions 工作流設計

| 工作流 | 觸發條件 | 作用 |
|--------|---------|------|
| `backend-deploy.yml` | push 到 dev/staging/main | 自動部署後端到對應環境 |
| `backend-promote.yml` | 手動 dispatch | 把特定 commit 晉升到下一環境 |
| `backend-ci.yml` | Workers 路徑有變更的 PR | 跑測試 + TypeScript 型別檢查 + ESLint |
| `claude-pr-review.yml` | PR 開啟或同步 | Claude AI 自動代碼審查 |

### backend-promote：跨環境晉升必須指定 commit SHA

如果你只是「把 staging 分支更新到最新」，你可能晉升了一個在 UT 還沒完整測試的版本。

解法：`backend-promote.yml` 要求指定 commit SHA：

```yaml
on:
  workflow_dispatch:
    inputs:
      commit_sha:
        description: 'Exact commit SHA to promote (get from UT deployment log)'
        required: true
      target_environment:
        type: choice
        options: [staging, production]
```

晉升時必須指定「我要晉升的是哪個 commit」，強制你確認 UT 日誌。

---

## 踩過的 CI/CD 坑

### 坑一：PR 沒有指定 `--base`，被合進了錯的分支

```bash
# ❌ 讓 GitHub 猜預設 base
gh pr create --title "feat: menu parsing"

# ✅ 明確指定 base 分支
gh pr create --title "feat: menu parsing" --base main
```

有一次沒有指定 `--base main`，GitHub 把 PR 的 base 設成了另一個 feature branch。PR merge 之後，代碼進了別的 feature branch 而不是 main。這個 mistake 花了一個下午修正。

### 坑二：commit 誤落 master

沒有確認當前分支，在 master 上直接 commit——這個 mistake 發生了不只一次。

解法：每次 commit 前跑 `git branch --show-current`。這成了 MaiNeu 的 Invariant（`INV-GIT-001`）：**每次 commit 前必須確認當前分支，PR merge 操作後尤其要確認。**

### 坑三：feature branch 從另一個 feature branch 開出

```bash
# ❌ 在 feature/a 分支上
git checkout -b feature/b  # 從 feature/a 開出！

# ✅ 永遠從 master 開新 branch
git checkout master && git pull
git checkout -b feature/xxx
```

PR diff 包含了 feature/a 的所有改動，review 非常混亂。這成了 `INV-GIT-005`。

---

## Claude PR Review：AI 進入 CI/CD 流水線

每次有 PR 開啟或更新，`claude-pr-review.yml` 自動觸發，呼叫 Claude Code Action，把審查結果貼在 PR comment 裡：

```markdown
## Code Review Report

### Critical Issues (必須修復)
...

### Warnings (建議修復)
...
```

有幾次，這個自動審查發現了我自己 review 時沒有注意到的問題。但也有 false positive：AI 有時會把「symlink 存在但 ls 沒顯示」誤報為「檔案不存在」。

**教訓：reviewer 的 Block 級問題，接受前必須人工驗證。自動審查是輔助，不是替代人類判斷。**

---

## DevOps 對我的意義

做 CI/CD 的最大收穫，不是「學了幾個 YAML 設定」，而是建立了工程嚴謹性的習慣：

- 每個改動都有 PR，都有審查記錄
- 每個環境的狀態都是可追溯的（哪個 commit 在哪個環境）
- 每次到 production 的晉升都有人工確認

一個人維護四個環境，聽起來很累。但有了 CI/CD 框架，大部分工作是自動的——我只需要做決策。

---

*下一篇：[Auth 的那些坑——JWT、Session 管理、OAuth Fusion 的血淚教訓](/tech/06-auth-jwt-oauth/)*
