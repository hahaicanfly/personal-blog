---
title: "Phase 2 的故事——從 WebSocket 到「暫不做」的產品決策"
date: 2026-05-21
weight: 8
series: ["MaiNeu 開發旅程"]
tags: ["product", "architecture", "websocket", "sharing", "feature-flags", "durable-objects"]
categories: ["技術版"]
description: "分享連結 72 小時過期設計、Durable Objects OrderBroker、PollingOrderRepository 到 WebSocketOrderRepository 零 UI 改動替換架構、「暫不做」決定的理由"
showToc: true
TocOpen: false
draft: false
---

# Phase 2 的故事——從 WebSocket 到「暫不做」的產品決策

*MaiNeu 開發旅程 第八篇（完結篇）*

---

## Phase 2 的起點

MaiNeu Phase 1 的核心：一個人，掃描菜單，看到翻譯結果。

但我一直在想一個場景：**一群朋友在餐廳，其中一個人掃了菜單，其他人怎麼一起看、一起點餐？**

現實中，大家輪流把手機傳來傳去。Phase 2 的設計目標：**讓一個人掃描的菜單，能同時給同桌的所有人看，並且一起點餐。**

---

## Phase 2-A：分享連結

### 設計

```
用戶掃描菜單
  → 點「分享」
  → 後端產生 share_code（6位字元，如 X7K2M9）
  → 生成分享連結 https://maineu.com/m/X7K2M9
  → 用戶用 LINE / WhatsApp 傳給朋友
  → 朋友打開連結，在瀏覽器看到菜單
```

### 連結過期時間：72 小時

安全角度：永久連結有風險，餐廳的菜單和價格會改變。
用戶角度：朋友可能隔天才點開，連結還能用嗎？

決定：**72 小時後過期**。這個數字基於「用餐場景的時間框架」——用餐完之後 72 小時，這個菜單連結已經沒有意義了。

### Feature Flag 策略

Phase 2-A 實作完成後：先開 Flag 在 UT 環境測試，Staging 和 Production 的 Flag 暫時關閉。

好處：代碼已在 production 環境跑，後端 API 可以先完整測試，之後要開放只需要 toggle flag。

副作用：**被 flag 隱藏的功能，很容易忘記測試**。第三方 API 的 response 格式在 flag 關閉期間改變了，我六天後才發現（詳見[第三篇](/tech/03-backend-from-scratch/)）。

---

## Phase 2-B：共享點餐——雄心壯志的設計

### 後端：Durable Objects + WebSocket

Cloudflare Workers 是無狀態的，但 WebSocket 需要「持久連接」。Cloudflare 的 **Durable Objects** 解決了這個矛盾：

```
用戶 A 連接 WebSocket → OrderBroker (order_id=abc)
用戶 B 連接 WebSocket → OrderBroker (order_id=abc)
用戶 A 點了一道菜 → PUT /orders/abc/items/123
後端更新資料庫 → 發訊息給 OrderBroker
OrderBroker 廣播給所有 WebSocket 客戶端
用戶 B 的 App 即時更新
```

後端實作完成：`OrderBroker` Durable Object、WebSocket endpoint `/orders/:id/stream`、kill switch `SHARED_ORDER_WS_ENABLED`（萬一撐不住，即時降級到 polling）。

### Android 端：可替換的 Repository 抽象

為了未來的 WebSocket 整合，特別設計了一個抽象：

```kotlin
// 介面讓 P5（WebSocket）可以替換進來，不需要改動 UI 層
interface OrderRepository {
    fun observeSummary(): Flow<OrderSummary>
}

// P4 實作：用 Polling（每 5 秒查詢一次）
class PollingOrderRepository : OrderRepository {
    override fun observeSummary(): Flow<OrderSummary> = flow {
        while (true) {
            val summary = apiClient.getOrderSummary(orderId)
            emit(summary)
            delay(5000)
        }
    }
}

// P5 計劃實作：換成 WebSocket，UI 不用改
class WebSocketOrderRepository : OrderRepository {
    override fun observeSummary(): Flow<OrderSummary> = channelFlow {
        // WebSocket 連接...
    }
}
```

UI 只依賴 `OrderRepository` 介面，未來把 `PollingOrderRepository` 換成 `WebSocketOrderRepository`，UI 零改動。

---

## 「暫不做」的決定

Phase 2-B 架構設計做完之後，我問自己：

**「現在做這個值得嗎？」**

現實情況：
- MaiNeu 尚未正式上線，沒有真實用戶資料
- Phase 1 的核心功能（菜單掃描、翻譯）還沒有在真實場景中大量驗證
- **用戶研究是空的**——我從來沒有問過真實用戶「你希望和朋友分享菜單的方式是什麼」

這個功能完全基於我的直覺和假設。

**決定：P4（共享點餐 UI）+ P5（WebSocket）暫不做。** 等 Phase 1 正式上線、有了真實用戶之後，再根據實際使用資料決定優先級。

這個決定讓我省下了至少一個月的開發時間，把資源集中在「確保 Phase 1 夠好、能上線」。

---

## 有意識的技術債

那個「暫不做」的 WebSocket 代碼，現在仍然在 production 後端跑著（kill switch 關閉）。`OrderBroker` 部署了，但沒有任何 Android 客戶端連接它。

這是一個「有意識的技術債」——我知道這個代碼在那裡，知道它的成本，知道什麼時候要把它打開。

技術債的問題不是「有沒有」，而是「有沒有意識到、有沒有管理」。

把它寫在 ADR 裡：

```markdown
# ADR — Phase 2-B WebSocket 暫緩啟用

決定：WebSocket 後端已實作並部署（kill switch 關閉）。
Android P4/P5 暫不開發，等 Phase 1 上線後重新評估。

理由：沒有真實用戶資料佐證這個功能的需求強度。

重啟條件：Phase 1 上線後，如果有 >10% 的用戶從分享連結開啟、
且有用戶明確表達「想和同桌一起點餐」，則提升 Phase 2-B 優先級。
```

---

## 這個系列的反思

Phase 2 的故事讓我理解了：

**產品開發不只是「把功能做完」，而是「把對的功能在對的時候做完」。**

「做完」很容易。WebSocket 後端、Durable Objects、Android 端的 Repository 抽象——這些我都做了。

「對的時候」更難。在沒有用戶、沒有資料的情況下，「這個功能用戶需要嗎」的答案只能是猜測。

最健康的做法，是把「做了但暫時不開放」和「完全沒做」分開管理——前者是有意識的技術投資，後者是有意識的資源節約。都比「模糊地半做半不做」要好。

---

*系列完結。MaiNeu 還在開發中，還沒上線。接下來的故事——上線、第一批真實用戶的回饋、Phase 2-B 最終是否做——等它們發生之後再寫。*
