---
title: "[Supabase] Realtime Subscription (即時訂閱) - 筆記"
pubDatetime: 2026-07-23T09:54:05.868Z
tags: ["Supabase","WebSocket","Database","React.js","Backend"]
description: "Table of contents 什麼是 Realtime Subscription ? Realtime Subs..."
hackmd_id: "Skw8DU1Sfg"
---

## Table of contents

## 什麼是 Realtime Subscription ?
**Realtime Subscription（即時訂閱）** 是 Supabase 以 WebSocket 所提供的核心功能。

* **功能**：**「讓前端應用程式不需要手動刷頁面，就能自動接收資料庫變更（INSERT / UPDATE / DELETE）的即時推播。」**
* 傳統的 HTTP 請求是「前端問，後端答」；而 Realtime 則是建立一條持續連線，由「資料庫主動告訴前端資料變了」。



## 1. 傳統拉取 vs. 即時訂閱

| 模式 | 運作機制 | 優缺點與適用情境 |  
| :--- | :--- | :--- |  
| **傳統請求<br>(Request / Polling)** | 透過 HTTP 請求發送，或使用 `setInterval` 定期輪詢伺服器。 | 🔴 浪費頻寬，無法達到真正的即時，且會有延遲時間。 |  
| **即時訂閱<br>(Realtime Subscription)** | 前端與 Supabase 建立 **WebSocket** 長連線，持續監聽事件。 | 🟢 零延遲，資料庫有變化時**主動推播 (Push)**，體驗極佳。 |



## 2. 核心應用場景

Realtime 非常適合用於需要 **即時互動與同步數據** 的產品功能：

1. **聊天室 / 訊息系統**：訊息送出後，對方的畫面無感自動更新。  
2. **協作工具（如 Notion, Figma, Trello）**：多人在同一頁面同時編輯看板或文件。  
3. **數據儀表板 (Dashboard)**：如 Dashboard，當有人下單時，統計圖表自動向上跳動。  
4. **即時通知與推播**：小紅點通知、按讚數即時增加、訂單狀態更新。



## 3. 程式碼實作：React + Supabase 訂閱流程

在 React 中使用 Realtime 時，通常會在 `useEffect` 內部進行頻道（Channel）訂閱，並在元件卸載時解除訂閱以釋放記憶體：

```javascript
import { useEffect, useState } from "react"
import supabase from "./supabase-client"

export default function RealtimeDashboard() {
  const [sales, setSales] = useState([])

  useEffect(() => {
    // 1. 建立並監聽指定資料表的頻道
    const channel = supabase
      .channel("sales-realtime-changes") // 自訂頻道名稱
      .on(
        "postgres_changes",
        {
          event: "*", // 監聽所有變更事件: 'INSERT', 'UPDATE', 'DELETE' 或 '*'
          schema: "public",
          table: "sales_deals", // 欲監聽的資料表名稱
        },
        (payload) => {
          console.log("資料庫發生變更了！", payload)
          // payload 包含 new (新資料), old (舊資料), eventType (變更類型)
          
          // 2. 收到通知後可直接更新 State 或重新 fetchMetrics()
        }
      )
      .subscribe()

    // 3. 元件卸載時，務必清除 Channel 釋放資源
    return () => {
      supabase.removeChannel(channel)
    }
  }, [])

  return <div>...</div>
}
```

## 4. payload資料結構  
在 Supabase 的 Realtime 訂閱中，當資料庫發生變更時，接收到的 `payload` 是一個 JavaScript 物件。

### 標準 payload 的資料結構  
以 `sales_deals` 資料表為例，當有人新增、修改或刪除資料時，`payload` 的格式長這樣：

```javascript
{
  // 1. 變更類型：'INSERT' | 'UPDATE' | 'DELETE'
  eventType: 'INSERT', 

  // 2. 變更後的「新資料」（新增或修改時才有，刪除時為 null）
  new: {
    id: "a1b2c3d4-...",
    name: "Alice",
    value: 15000,
    created_at: "2026-07-23T09:30:00Z"
  },

  // 3. 變更前的「舊資料」（修改或刪除時才有，新增時為 {} 或 null）
  old: {},

  // 4. 資料庫對應的 schema 與 table 名稱
  schema: 'public',
  table: 'sales_deals',

  // 伺服器發送事件的時間戳記
  commit_timestamp: '2026-07-23T09:30:00.123Z',
  errors: null
}
```

### 2. 三種事件情境的 new 與 old 差異  
不同動作下，`payload.new` 和 `payload.old` 裡面裝的東西會不一樣：

#### A. 新增資料 (`eventType: 'INSERT'`)
* `payload.new`：包含剛剛被新增進去的那一筆完整資料。
* `payload.old`：為 `{}` 或 `null`。

```javascript
// 可以直接拿 payload.new 塞進 State 裡面：
setDeals((prev) => [...prev, payload.new]);
```

#### B. 更新資料 (eventType: 'UPDATE')
* `payload.new`：更新後的最新完整資料。
* `payload.old`：更新前的主鍵資訊（例如 `{ id: "a1b2c3d4-..." }`）。

```javascript
// 在 State 陣列中尋找並替換該筆資料：
setDeals((prev) => 
  prev.map((deal) => (deal.id === payload.new.id ? payload.new : deal))
);
```
#### C. 刪除資料 (eventType: 'DELETE')
* `payload.new`：為 `null`。
* `payload.old`：被刪除的那筆資料的主鍵/ID（例如 `{ id: "a1b2c3d4-..." }`）。

```javascript
// 根據 payload.old.id 將被刪除的資料過濾掉：
setDeals((prev) => prev.filter((deal) => deal.id !== payload.old.id));
```

### 實戰範例：在 React 中處理 payload  
把上面的觀念組合起來，在 `useEffect` 中處理 Realtime 事件時就可以寫得非常乾淨：

```javascript
useEffect(() => {
  const channel = supabase
    .channel("sales-deals-changes")
    .on(
      "postgres_changes",
      { event: "*", schema: "public", table: "sales_deals" },
      (payload) => {
        console.log("收到事件類型:", payload.eventType);

        if (payload.eventType === "INSERT") {
          console.log("新增的資料為:", payload.new);
          // TODO: 將 payload.new 加入 state
        } else if (payload.eventType === "UPDATE") {
          console.log("修改後的資料為:", payload.new);
          // TODO: 用 payload.new 更新原本的 state
        } else if (payload.eventType === "DELETE") {
          console.log("被刪除的資料 ID 為:", payload.old.id);
          // TODO: 從 state 移除該 ID
        }
      }
    )
    .subscribe();

  return () => {
    supabase.removeChannel(channel);
  };
}, []);
```

## 5. 注意事項

<blockquote class="my-6 p-4 bg-red-50 dark:bg-red-950/30 border-l-4 border-red-500 rounded-r-md text-red-900 dark:text-red-200 blocknoted-fix">

:warning: Supabase 預設為了安全性與效能考量，不會自動為所有資料表開啟 Realtime。

</blockquote>

開啟步驟：
* 進入 Supabase Dashboard -> Database -> Publications。
* 找到 supabase_realtime 設定區塊。
* 將需要即時監聽的資料表（例如 sales_deals）的 Realtime 開關切換為啟用（Enable）。

## 5. 模式比較：直接更新 State vs. 觸發 Refetch  
收到變更通知時，處理前端 UI 狀態通常有兩種策略：

| 策略 | 做法 | 適用情境 | 優點 | 缺點 |  
| :--- | :--- | :--- | :--- | :--- |  
| **重新請求 (Refetch)** | 收到 Payload 後，直接重新呼叫 `fetchData()` | 適用於有**聚合計算 (sum, group by)** 或資料關聯較複雜的圖表/表格 | 數據最準確，資料庫怎麼算畫面就怎麼呈現 | 會多消耗網路請求與頻寬  
| **直接改 State (Optimistic)** | 使用 `payload.new` 手動將新資料 `.concat()` 或 `.map()` 進陣列 | 適用於列表、聊天訊息等**一對一**呈現單筆資料的介面 | 體感速度極快，使用者體驗非常好 | 如果失敗需要寫邏輯把資料「還原 (Rollback)」




<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon:  本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>