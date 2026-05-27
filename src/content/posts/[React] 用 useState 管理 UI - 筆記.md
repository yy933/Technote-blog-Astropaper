---
title: "[React] 用 useState 管理 UI - 筆記"
pubDatetime: 2025-05-15T03:55:28.000Z
tags: ["JavaScript","React.js","React Hook"]
description: "Table of contents Declarative vs. Imperative UI Imperative（..."
hackmd_id: "r1e96CKxxe"
---

## Table of contents


## Declarative vs. Imperative UI
* Imperative（命令式）：一步步告訴系統怎麼改變 UI（如 DOM 操作 show()、hide()）。
* Declarative（宣告式）：描述 UI 在某個狀態應該長怎樣，React 負責讓它發生。
* 生活化的例子：
Imperative 像是你開車告訴駕駛每個轉彎；Declarative 像你坐計程車只告訴司機目的地。

## 表單互動例子說明
使用者操作會導致 UI 狀態變化：
* 輸入文字 → 按鈕啟用
* 按下 Submit → 顯示 Spinner，表單與按鈕停用
* 請求成功 → 顯示成功訊息，隱藏表單
* 請求失敗 → 顯示錯誤訊息，重新啟用表單

## Thinking about UI declaratively：React 實作流程（5 步驟）
### Think in React
1. 列出視覺狀態（Visual States）
1. 定義狀態轉換的觸發事件
1. 用 `useState` 表示狀態
1. 移除非必要的狀態（避免衝突）
1. 與事件處理器連結設定狀態

### Step 1: 列出視覺狀態（Visual States）
常見狀態：
* `empty`：輸入框無內容，按鈕停用
* `typing`：輸入中，按鈕啟用
* `submitting`：送出中，顯示 spinner，UI 鎖定
* `success`：成功訊息取代表單
* `error`：顯示錯誤訊息，回到可輸入狀態

💡 小技巧：用 status props 模擬狀態進行畫面 Mock（不用邏輯即可先設計 UI）

### Step 2: 定義狀態轉換的觸發事件
* 使用者輸入 → typing 或 empty
* 點擊送出 → submitting
* 成功回應 → success
* 失敗回應 → error

畫出「狀態轉移圖」釐清流程（圓圈表示狀態、箭頭表示觸發事件）。如下所示:
![螢幕擷取畫面 2025-05-13 114045](https://hackmd.io/_uploads/SkG20VlWgl.png)

### Step 3：用`useState` 表示狀態
剛開始會寫很多state，可能會導致狀態衝突（ex: `isTyping` 和 `isSubmitting`同時 `true`），先寫出來再重構:
```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [isEmpty, setIsEmpty] = useState(true);
const [isTyping, setIsTyping] = useState(false);
const [isSubmitting, setIsSubmitting] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
const [isError, setIsError] = useState(false);
```

### Step 4：移除非必要的狀態（避免衝突）
:arrow_right: 判斷原則（Refactoring State Checklist）：
1. 是否存在矛盾？
❌ `isTyping` 與 `isSubmitting` 同時為 `true` 不合理
✅ 改為 `status = 'typing' | 'submitting' | 'success'`，只會有一種狀態成立

1. 資訊是否重複？
❌ `isEmpty` 與 `isTyping` 容易打架
✅ 改為 `answer.length === 0` 判斷是否為空即可

1. 是否可由其他狀態推得？
❌ `isError` 多餘
✅ `error !== null` 已能表示是否有錯誤

:warning: 避免「不可能的 UI 狀態」：
如果你同時設 `isError: true` 且 `isTyping: false`，畫面會顯示錯誤訊息，但輸入框卻被鎖住，使用者完全無法修正錯誤 —— 這就是「不合理的狀態組合」，要透過整理 `state` 結構來避免這種情況。

### Step 5：與事件處理器連結設定狀態
* onChange → 更新 answer 並清除錯誤
* onSubmit → 設為 submitting，執行 async 請求
* async 成功 → 設為 success
* async 失敗 → 設為 error 並顯示訊息

##  延伸工具建議：Storybook / Living Styleguides
一次展示所有狀態的元件畫面，有助設計、測試與開發。
```jsx
{['empty', 'typing', 'submitting', 'success', 'error'].map(status => (
  <Form status={status} />
))}
```

## Recap
Declarative programming就是描述每一種視覺狀態的樣子，而不是用指令逐步操作UI。

開發 React 元件時的步驟：
* 辨識所有視覺狀態
* 定義觸發狀態變化的使用者/電腦的行為
* 使用 `useState` 表示狀態內容
* 移除非必要狀態，避免 bug 與矛盾
* 與事件處理器連結，觸發`setState`