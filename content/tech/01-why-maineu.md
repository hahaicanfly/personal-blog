---
title: "為什麼做 MaiNeu？一個 Android 工程師的 Side Project 起點"
date: 2026-05-21
weight: 1
series: ["MaiNeu 開發旅程"]
tags: ["android", "side-project", "product", "ai", "gemini", "cloudflare"]
categories: ["技術版"]
description: "從一頓挫折的旅遊晚餐，到決定自己做 App。第一個 prototype、後端選型、FCM 手動實作 JWT。"
showToc: true
TocOpen: false
draft: false
---

# 為什麼做 MaiNeu？一個 Android 工程師的 Side Project 起點

*MaiNeu 開發旅程 第一篇*

---

## 那頓飯的挫折

事情的起點是一頓普通的旅遊晚餐。

菜單是日文的。我打開 Google Translate 的相機功能，對著菜單掃，螢幕上出現一堆漂浮的中文字——「炭火燒烤特選和牛」、「季節野菜天婦羅」——但這些字是貼在原始日文上方的，版面混亂，根本沒辦法看清楚哪道菜是什麼價格，有沒有我不能吃的成分。

我用相機翻譯，翻了三次，最後還是靠著用手指比著菜單、用破碎的英文跟服務生溝通。

回到台灣後，我一直在想一個問題：為什麼沒有一個 App，能把菜單掃描進去，然後用清楚的列表呈現每道菜的名稱、翻譯、價格？為什麼每次用翻譯 App 看菜單，都是一種「勉強能用」的體驗？

這個問題在我腦子裡放了幾個月。2026 年初，我決定自己做。

---

## 第一版長什麼樣子

最初的想法極其簡單：

1. 讓用戶拍一張菜單照片
2. 傳送到 Gemini API
3. 把結果顯示出來

第一個 prototype 大概花了兩個週末完成。代碼簡陋到我現在不敢看：一個 Activity，一個 ViewModel，一個直接在 ViewModel 裡面呼叫 Gemini API 的函式，回傳一個字串，直接顯示在 `Text()` 裡。

沒有 loading state。沒有錯誤處理。沒有任何架構可言。

但它可以用。

我把第一張測試菜單（一張日式拉麵店的菜單圖片）丟進去，Gemini 在大約三秒後回傳了一份結構化的 JSON，裡面有每道菜的日文名稱、中文翻譯、價格——甚至包含了一些過敏原提示。

那個瞬間讓我意識到：這件事是可以做到的。

---

## 第一個真實的技術選型困難

很快地，「可以用」和「做成產品」之間的距離變得非常明顯。

**第一個大問題是：後端要放哪裡？**

我是 Android 工程師，幾乎沒有後端經驗。最初的版本是直接從 App 呼叫 Gemini API——這意味著 API Key 必須放在 App 裡。這是一個非常明顯的安全問題：只要有人反編譯 APK，就能拿到 API Key。

我知道這個問題，但不知道怎麼解。

第一個想法是 Firebase Cloud Functions。我以前聽過，但從來沒用過。研究了幾天之後，發現要用 Firebase Functions 需要 Node.js 背景，而且冷啟動時間對這種「即時翻譯」的場景來說可能太長。

後來在某篇技術文章裡看到了 Cloudflare Workers——一個跑在全球邊緣節點上的無伺服器平台，幾乎沒有冷啟動，而且有相當慷慨的免費額度。

我決定試試。

---

## 學習後端的第一週

第一週寫後端的感受，可以用一個詞形容：**完全失控**。

TypeScript 我會一點（以前碰過一些 Web 開發），但 Cloudflare Workers 的環境和我熟悉的 Node.js/Browser 環境都不一樣。它有自己的限制：不能用 `fs` 模組（沒有檔案系統）、`setTimeout` 行為不同、很多 npm 套件因為依賴 Node.js 原生模組而無法使用。

第一個讓我卡很久的問題是：**如何在 Cloudflare Workers 裡呼叫 Firebase Cloud Messaging 發送推播通知。**

正常做法是用 `firebase-admin` SDK。但 `firebase-admin` 依賴 Node.js 的 `https` 和 `crypto` 模組，在 Workers 環境不能用。

解法最後是：用 Web Crypto API 手動簽發 JWT（JSON Web Token），然後用這個 JWT 換取 FCM 的 OAuth2 Token，再用這個 Token 呼叫 FCM HTTP v1 API。

整個流程手動實作，用的是 `jose`（一個支援 Web Standards 的 JWT 函式庫）。這讓我第一次真正理解了 JWT 的結構——不是「它是一個 token」，而是「它是一個被簽名的 JSON，任何人都能驗證簽名是否有效」。

---

## 第一個讓我真的有「這是產品」感覺的時刻

有一天，我把 App 裝在手機上，帶著它去一家我常去的日式餐廳。

用 CameraX 實時掃描模式——App 打開相機，鏡頭對著菜單，系統自動偵測文字區域並框選，輕觸快門，三秒後螢幕上出現一份清楚的點餐介面：每道菜有中文名稱、日文原名、價格，以及一個寫著「含芝麻」的小標籤。

我點了一道菜，把 App 遞給同行的朋友看，他說：「這比 Google Translate 好用多了。」

這句話讓我決定繼續做下去。

---

## 這個系列要記錄什麼

接下來幾篇，我會寫 MaiNeu 開發過程中真實遇到的困難：

- **Compose 的深坑**：協程取消機制如何讓 UI 卡死、`LaunchedEffect` 的 key 語義、FlowRow 的版面問題
- **後端從零開始**：Cloudflare Workers 的限制、Gemini API 地區問題、D1 database 的 migration 管理
- **安全工程**：從「不要 hardcode 密碼」到真正實作 HMAC 請求簽名
- **CI/CD 與三層環境**：GitHub Actions 的設計、環境隔離為什麼重要
- **Auth 的血淚**：JWT、Session 管理、OAuth Fusion 的邊界案例
- **AI 工作流設計**：如何從「用 AI 寫代碼」進化到「設計 AI 的工作軌道」
- **Phase 2 的故事**：一個功能從設計到「暫不做」的全過程

這些都是真實發生的事，有具體的 bug、具體的解法、具體的反思。

不是教程，而是一份工程日誌。

---

*下一篇：[Compose 深坑錄——我在 Jetpack Compose 踩過的那些坑](/tech/02-compose-deep-dives/)*
