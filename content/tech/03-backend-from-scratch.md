---
title: "後端從零開始——一個 Android 工程師如何讀懂 Cloudflare Workers"
date: 2026-05-21
weight: 3
series: ["MaiNeu 開發旅程"]
tags: ["backend", "cloudflare-workers", "typescript", "hono", "d1", "gemini-api"]
categories: ["技術版"]
description: "Cloudflare Workers Smart Placement 地區限制、D1 migration 失敗處理、Gemini KeyPool 設計、timing-safe 比較——Android 工程師自學後端的血淚記錄"
showToc: true
TocOpen: false
draft: false
---

# 後端從零開始——一個 Android 工程師如何讀懂 Cloudflare Workers

*MaiNeu 開發旅程 第三篇*

---

## 後端，那個「別人負責的東西」

在做 MaiNeu 之前，我對後端的理解大概是：

「有個 server 在某個地方，它會接收請求，然後回傳 JSON。」

這個理解對於一個純前端工程師來說已經夠了——你只需要知道怎麼呼叫 API，不需要知道 API 的背後發生了什麼。

但 MaiNeu 只有我一個人，沒有「後端工程師」這個角色。API 要自己寫，資料庫要自己設計，部署要自己管。

---

## 為什麼選 Cloudflare Workers

| 選項 | 評估 |
|------|------|
| Firebase Cloud Functions | 熟悉，但冷啟動慢，設定繁瑣 |
| AWS Lambda | 業界標準，但學習曲線陡，免費額度少 |
| **Cloudflare Workers** | 幾乎沒有冷啟動，全球邊緣節點，免費每天 10 萬次請求 |

MaiNeu 的核心使用場景是「拍照 → 立即翻譯」，延遲對用戶體驗影響很大。選 Cloudflare Workers 之後，這個決定帶來了幾個意外的坑。

---

## Gemini API 地區限制：Smart Placement 的陷阱

某些用戶的菜單解析請求一直失敗，從錯誤日誌來看是 Gemini API 回傳 `User location is not supported for the API use`。

但我的 API Key 是美國帳號申請的，為什麼有地區問題？

### 根本原因

Cloudflare **Smart Placement** 會把你的 Worker 部署到「對 upstream 延遲最低」的地區。用戶在亞洲時，Cloudflare 可能選擇香港（HKG）或新加坡（SIN）節點——而 Gemini API 在這些地區有限制。

### 解法

```toml
[placement]
mode = "smart"
hint = "wnam"  # 強制使用北美西區
```

**關鍵陷阱**：頂層的 `[placement]` 不會被 `[env.*]` 繼承。每個環境都要獨立宣告：

```toml
[env.staging.placement]
mode = "smart"
hint = "wnam"

[env.production.placement]
mode = "smart"
hint = "wnam"
```

我以為頂層設定就夠了，結果 production 部署後又炸了一次。這個 bug 花了好幾個小時才找到。

---

## D1 資料庫：Migration 失敗後別重試 Apply

Cloudflare D1 是底層 SQLite 的資料庫服務。`wrangler d1 migrations apply` 執行 migration，但如果中途失敗（例如欄位名稱重複），Wrangler **不會自動回滾**。

問題：migration 被標記為「已執行」，但資料庫是半套用的狀態。再次 apply 報「migration 已套用」，但資料庫其實不完整。

**解法**：

```bash
# ❌ 不要重試 apply
wrangler d1 migrations apply DB_NAME

# ✅ 用 execute 手動補跑剩餘的 SQL
wrangler d1 execute DB_NAME --command "ALTER TABLE users ADD COLUMN new_field TEXT"
```

這條規則後來寫進了 MaiNeu 開發規範：**D1 migration 失敗時，改用 `execute --command` 手動修復，永遠不要重試 apply**。

---

## Gemini API 的 KeyPool 策略

Gemini API 有 Rate Limit，免費 tier 每天有解析次數上限。解法是維護多個 API Key，Round-robin 輪流使用：

```typescript
class KeyPool {
    private keys: string[];
    private cooldowns: Map<string, number> = new Map();
    private currentIndex = 0;

    getNextKey(): string | null {
        const now = Date.now();
        for (let i = 0; i < this.keys.length; i++) {
            const idx = (this.currentIndex + i) % this.keys.length;
            const key = this.keys[idx];
            const cooldownUntil = this.cooldowns.get(key) ?? 0;
            
            if (now > cooldownUntil) {
                this.currentIndex = (idx + 1) % this.keys.length;
                return key;
            }
        }
        return null;  // 所有 key 都在冷卻中
    }

    markCooling(key: string, retryAfterMs: number) {
        this.cooldowns.set(key, Date.now() + retryAfterMs);
    }
}
```

當一個 Key 被 rate limit（HTTP 429），把它放入指數退避冷卻，自動切換到下一個 Key。API 容量翻了三倍，代碼複雜度幾乎沒增加。

---

## JWT 不是一個魔法 token

```
base64(header) . base64(payload) . base64url(signature)
```

JWT 是一個被簽名的 JSON。**任何人都能 decode payload，但只有知道 secret 的人才能驗證 signature 是否有效。**

這意味著 JWT 裡的 payload 不能放敏感資訊。MaiNeu 用雙 Token 系統：Access Token（30 分鐘效期）+ Refresh Token（7 天效期）。

---

## Timing-safe 比較

```typescript
// ❌ 普通字串比較：易受 Timing Attack
if (requestSecret === expectedSecret) { ... }

// ✅ Cloudflare Workers 提供 timingSafeEqual
const encoder = new TextEncoder();
const a = encoder.encode(requestSecret);
const b = encoder.encode(expectedSecret);
if (a.length !== b.length) return false;
const result = await crypto.subtle.timingSafeEqual(a, b);
```

攻擊者可以透過測量「比較時間」猜測 secret 內容。Timing-safe 比較確保不論字元是否相同，比較時間都一樣。

---

## 第三方 API 整合：永遠打真實 endpoint

Frankfurter 匯率 API 的文件描述 v1 格式，但我呼叫的是 v2，response 格式完全不同。我花了六天才發現——因為 feature flag 讓 UI 入口隱形，根本沒有真實流量打到這個路徑。

**規矩**：任何第三方 API 整合，動工前必須用 `curl` 打至少一個真實 endpoint，把 raw response 存起來。不要相信只看過文件沒打過的 endpoint。

---

## 後端讓我的 Android 開發思維變了

做後端之後，看 API 的角度完全不同了：

- 這個 endpoint 的 rate limit 是什麼？超了，Android 端怎麼處理？
- 後端回傳 null 欄位的情況什麼時候發生？
- 後端 migration 改了欄位名稱，Android 的 `@SerialName` 需不需要更新？

後端不再是黑箱，而是另一個我能看懂的系統。

---

*下一篇：[安全工程啟蒙——從「不要 hardcode 密碼」到 5 層防禦架構](/tech/04-security-engineering/)*
