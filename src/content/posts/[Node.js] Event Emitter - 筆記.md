---
title: "[Node.js] Event Emitter - 筆記"
pubDatetime: 2026-05-25T00:13:22.000Z
modDatetime: 2026-05-25T10:04:23.073Z
tags: ["Node.js"]
description: "Table of contents :memo: 什麼是Event Emitter 簡單來說，Event Emitte..."
hackmd_id: "HJznOSbxzg"
---

## Table of contents

## :memo: 什麼是Event Emitter  
簡單來說，Event Emitter（事件觸發器） 是一種在程式設計中非常常見的設計模式（Design Pattern），它就像是一個廣播電台。

當程式中某個地方發生了特定的事情（例如：使用者點擊按鈕、檔案下載完成、或是伺服器收到請求），這個廣播電台就會**發送一個「信號」（稱為 Event）**。**而其他對這件事感興趣的程式片段，可以提前向電台「訂閱」（稱為 Listen），當信號發出時，這些訂閱者就會立刻執行對應的動作。**

這在程式架構中被稱為**發布-訂閱模式（Publish-Subscribe Pattern）**。 


## :memo: 核心概念  
Event Emitter 的運作主要圍繞在以下三個動作：

* **Listen（監聽 / 訂閱）**：**告訴 Event Emitter，如果某個事件發生了，請通知我，並執行我指定的函式**（這個函式通常稱為 Callback 或是 Listener）。
* **Emit（觸發 / 廣播）**：當某件事情真的發生時，**負責把事件送出去，並把相關的資料一起傳給所有正在監聽的人**。
* **Event Name（事件名稱）**：用來**區分不同事件的「標籤」**（通常是字串），例如 `click`、`data_received`、`error`。
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

💡 重要特性：在 Node.js 中，事件名稱（如 `'emailRequest'`）本質上只是字串，不需要事先宣告或定義。它在第一次被 `.on()` 或 `.emit()` 使用的瞬間，就會在記憶體中自動建立。

</blockquote>



## :memo: 為什麼需要 Event Emitter？  
最主要的原因是**為了達到 「解耦」（Decoupling）**。

* 沒有 Event Emitter 時：A 模組做完事情後，必須親自去呼叫 B 模組、C 模組和 D 模組的函式。A 必須知道 B、C、D 的存在，這讓程式碼變得很緊密、很難維護。
* 有了 Event Emitter 後：A 模組做完事情，只需要大喊一聲：「我做完了！（Emit）」，至於誰想聽這個消息？A 完全不需要知道。B、C、D 只要自己去聽這個消息就好。這讓模組與模組之間變得非常獨立。

## :memo: 實際範例（Node.js）  
在 Node.js 中，Event Emitter 是內建的核心模組（events），非常多功能（例如 Stream、HTTP 伺服器）都是根據它建立的。

以下是一個簡單的範例：
```javascript
// import EventEmitter
import { EventEmitter } from 'node:events'

const customerDetails = {
  fullName: 'Meryl Sheep',
  email: 'baah@thedevilwearswool.com',
  phone: 12345678910
}

// create the emitter
const emailRequestEmitter = new EventEmitter()

// define the listener function
function generateEmail(customer) {
    console.log(`Email generated for ${customer.email}`)
}

// register the listener
emailRequestEmitter.on('emailRequest', generateEmail)
emailRequestEmitter.on('emailRequest', () => console.log('task assigned'))
emailRequestEmitter.on('emailRequest', () => console.log('email logged'))

// emit the event
setTimeout(()=> {
    emailRequestEmitter.emit('emailRequest', customerDetails)
}, 2000)
```

### 說明  
這段程式碼展示了 Event Emitter 的核心價值：**一對多（One-to-Many）的廣播機制**。**當觸發一次事件時，多個不同的獨立任務會被依序（同步地）觸發並執行。**
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

雖然我們常常把 Event Emitter 拿來處理背景的非同步任務（例如上面範例放在 `setTimeout` 裡面），**但 `.emit()`觸發監聽器的過程本身是「同步依序」執行的。**

**意即，當 `emit` 被呼叫時，它會按照 `.on()` 註冊的順序，執行完第一個 Function，才執行第二個，再執行第三個。**

</blockquote>


逐段拆解這段程式碼在記憶體與執行時（Runtime）實際上做了什麼：

#### 1. 建立事件與監聽器的綁定（`.on`）
```javascript
emailRequestEmitter.on('emailRequest', generateEmail)
emailRequestEmitter.on('emailRequest', () => console.log('task assigned'))
emailRequestEmitter.on('emailRequest', () => console.log('email logged'))
```
在 `emailRequestEmitter` 物件內部，其實建立了一個類似於物件（Object）的結構，把事件名稱當作 Key，監聽器函式們當作一個陣列（Array）存起來。看起來就像這樣：
```javascript
// 概念上的內部結構
emailRequestEmitter._events = {
  emailRequest: [
    generateEmail,
    () => console.log('task assigned'),
    () => console.log('email logged')
  ]
}
```

#### 2. 非同步觸發與參數傳遞（.emit）
```javascript
setTimeout(() => {
  emailRequestEmitter.emit('emailRequest', customerDetails)
}, 2000)
```
* 非同步等待：程式會先等待 2 秒鐘。
* 觸發（Emit）：2 秒一到，`.emit()` 被呼叫。它做的事情非常簡單——去內部尋找 `emailRequest` 這個標籤，發現後面排了 3 個函式。
* 參數傳遞：它會依序（同步地）執行這 3 個 函式，並且把 `customerDetail`s 這個物件當作參數，塞進每一個函式裡面。

#### 執行結果分析  
2 秒鐘過後，終端機（Terminal）會依序印出以下三行：
```
Email generated for baah@thedevilwearswool.com
task assigned
email logged
```
## :memo: 對比前端`addEventListener`  
| 功能 | 後端 Node.js (EventEmitter) | 前端 瀏覽器 (DOM) |  
| :--- | :--- | :--- |  
| **誰在負責發信號** | `emailRequestEmitter` (你自己建立的物件) | `button` (網頁上的按鈕物件) |  
| **註冊監聽的方法** | `.on()` | `.addEventListener()` |  
| **事件名稱（標籤）** | `'emailRequest'` | `'click'` 或 `'submit'` |  
| **觸發時執行的動作** | `generateEmail` (Callback 函式) | `(event) => { ... }` (Callback 函式) |  
| **是誰來發射信號** | 由你在程式碼中手動呼叫 `.emit()` | 由瀏覽器在使用者真正點擊滑鼠時幫你觸發 |

### 誰來Emit
#### 1. 前端的 addEventListener  
在前端，通常只需要負責「監聽」  
`button.addEventListener('click', () => { console.log('按鈕被點擊了！') })`  
通常不需要在程式碼裡自己寫 `button.click()` 去觸發它，因為觸發的動作是由「使用者（或瀏覽器）」來完成的。當使用者的滑鼠真的點下去那一刻，瀏覽器就會在背後默默執行類似 `.emit('click')` 的動作。

#### 2. 後端的 EventEmitter  
在後端 Node.js 中，因為沒有畫面的按鈕，也沒有真正的「使用者滑鼠」可以點擊，所以「監聽」跟「發射」這兩個動作，通常都必須由你的程式碼手動包辦。

就像上面的範例：

* 先用 `.on('emailRequest', ...)` 綁定。
* 再用 `setTimeout` 加上 `.emit('emailRequest')` 發送。

## :memo: 後端專案中，通常是什麼時候會呼叫 `.emit()`？  
既然都是伺服器在執行，什麼時候會需要手動呼叫 `.emit()` 呢？最常見的場景就是 **「當某個漫長的工作完成時，用來通知其他部門」**。

常見的實際例子（例如使用者註冊）：
```javascript
// 1. 這裡也是伺服器在「監聽」
userSystem.on('user_registered', (user) => {
  // 負責寄歡迎信、負責建立預設資料、負責發送優惠券
});

// 2. 當有新使用者註冊時，伺服器執行的主要流程
async function handleUserRegistration(req, res) {
  try {
    // A. 把帳號密碼寫入資料庫 
    await db.save(req.body); 

    // B. 資料庫確定成功後，伺服器才手動觸發事件
    userSystem.emit('user_registered', req.body);

    // C. 馬上回應前端：「註冊成功！」
    res.send('Register Success!');
  } catch (error) {
    res.status(500).send('Registration Failed');
  }
}
```
在這個例子中：

* 觸發（Emit） 的時機：資料庫成功寫入使用者的那一刻。
* 監聽（On） 的時機：伺服器一啟動就準備好了。


---




<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>