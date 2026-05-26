---
title: "Event Loop Note"
pubDatetime: 2025-03-26T02:57:32.000Z
tags: ["Event Loop"," JavaScript","Interview Preparation","JavaScript"]
description: " title: Event Loop Note tags: Event Loop, JavaScript descrip..."
---

---
title: Event Loop Note
tags: Event Loop, JavaScript
description: A brief introduction to event loop in Javascript.
---



## Table of contents

## [MDN文件](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)中對Event loop的定義

JavaScript中，並行模型（concurrency model）建立於事件迴圈(event loop)的基礎上。Event loop的功能在於執行程式碼、蒐集與處理事件、以及執行等待中的次任務(sub-tasks).

## JavaScript is primarily single threaded.

**JavaScript是一種單線執行(single threaded)的程式語言**，亦即一次只有一條線執行所有的事。例如，有一個`for`迴圈需要花一段時間執行，則必須**等待這個`for`迴圈執行完畢，才能繼續執行後面的程式碼，這就會造成阻塞(blocking)**。
+ **如果是同步（Synchronous）執行：**
  想像一間餐廳裡只有一位服務生(單線)，假如第一桌的客人舉手找服務生點餐，服務生移動到第一桌，待第一桌點餐完畢，第二桌的客人舉手，服務生才能移動到第二桌；當服務生在第一桌點餐時，第一桌的客人問他的朋友想點甚麼，服務生並不能在此時離開去服務其他桌的客人，必須等第一桌的客人與朋友都決定好了，點餐才算完成。執行程式碼若遇到這種情況，需要等待該片段的code都執行完畢才能往下執行，可能會遇到畫面「卡住」的情況，稱為阻塞(blocking)。
<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

 舉例來說，執行以下程式碼：
```javascript
console.log("How long will it take")
for (let idx=0; idx < 999999999; idx++) {console.log(`This is the ${idx+1} loop.`)}
console.log("to print the result?")
```  
1. Console會先印出'How long will it take'
2. 接著`for`迴圈跑了999999999個迴圈，造成阻塞，因此有一段時間停滯。並且因為`console.log()` 是 I/O 操作，會佔用大量記憶體，甚至可能會拖垮開發工具。
3. 999999999迴圈都跑完後，才印出"to print the result?"

</blockquote>

+ **如果是非同步（Asynchronous）執行：**
  在餐廳的例子中，假設第一桌的客人點餐需要和朋友討論，服務生可以先去執行其他任務，例如先去送其他桌的餐、或告訴廚房要準備的餐等，待第一桌客人討論完再完成點餐。像Ajax（Asynchronous JavaScript and XML）這種以非同步（Asynchronous）執行的方法，就可以避免在等待回應的過程，無法繼續執行其他動作或導致瀏覽器停滯的情況。 
>:arrow_forward:關於JavaScript如何執行非同步事件，這篇參考文章有詳細的說明：[談談JavaScript中的asynchronous和event queue](https://pjchender.blogspot.com/2016/01/javascriptasynchronousevent-queue.html)
<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

重複一下上面的範例，這次使用`setTimeout()`這個方法:
```javascript
console.log("How long will it take");
setTimeout(() => { console.log("the result?")}, 2000);
console.log("to print");
```
1. Console會先印出'How long will it take'
2. 接著執行`setTimeout()`，開始計時2秒
3. 再執行第三行，印出"to print"
4. 最後，`setTimeout()`計時2秒結束，印出"the result?"

</blockquote>
![](https://i.imgur.com/wdY6217.png)
###### 圖片來源:https://tw.alphacamp.co/blog/ajax-asynchronous-request

 ---
## 事件循環的主要觀念 Main Concepts in Event Loop
![](https://i.imgur.com/tkMoZBW.png)
<p style="text-align: center;">JavaScript中的事件循環(Event Loop)</p>

###### 圖片來源:https://dev.to/rahulsaha28/javascript-4j1m
    
簡單來說，**事件循環是一個用於管理code執行的系統**，我們**想用JS非同步（asynchronously）**執行不阻塞的程式碼，這就是事件循環發揮功能的地方了。事件循環的主要元素包含：

### **1. Memory Heap**
以無順序的方式儲存物件的記憶體。當前使用的JavaScript變數和物件等會儲存在heap中。
### **2. Call Stack**

前面提到，JavaScript是以單線的方式執行，所有待執行的程式碼片段會被放在**堆疊(call stack)中執行**。當調用函式時，一個幀(frame)會被加入堆疊中，幀連結了該函數的參數(argument)與heap中的變數。Frame是以<span style="color:red">後進先出(Last in first out，LIFO)</span>的順序進入堆疊中，可以想成`array.push()`和`array.pop()`:
   + `array.push()`將元素加在array的最後
   + `array.pop()`將最後一個元素移除，並回傳被移除的元素
   (參考資料：[【在廚房想30天的演算法】Day 08 資料結構：堆疊 Stack](https://ithelp.ithome.com.tw/articles/10270257))
 <blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

   例如以下程式碼：
  ```javascript=1
   function one() {
     return function two() {
       return function three() {
         return 'Done!'
       }
    }
   }
console.log(one()()())
```

- **分析程式碼:**
`console.log(one()()())` 其實是：
```javascript
const result = one();  // result 現在是 function two
const result2 = result();  // result2 現在是 function three
const finalResult = result2();  // finalResult = 'Done!'
console.log(finalResult);
```
所以，函式執行的順序是： 1️⃣ `one()` 2️⃣ `two()` 3️⃣ `three()`

- **Call Stack 進入先後順序：**
當執行 `console.log(one()()())`; 時，Call Stack 變化如下：

1️⃣ `console.log()` 進入 Call Stack（但要等 `one()()()`執行完才輸出）
2️⃣ `one()` 進入 Call Stack
3️⃣ `one()` 回傳 two，從 Call Stack 彈出
4️⃣ `two()` 進入 Call Stack
5️⃣ `two()` 回傳 three，從 Call Stack 彈出
6️⃣ `three()` 進入 Call Stack
7️⃣ `three()` 回傳 'Done!'，從 Call Stack 彈出
8️⃣ `console.log('Done!')` 執行並輸出 'Done!'，然後從 Call Stack 彈出

- **Call Stack 變化（LIFO - 後進先出）**
```javascript
// 進入時
[console.log]  ⬅️ 1️⃣  進入
[one]          ⬅️ 2️⃣  進入
[two]          ⬅️ 3️⃣  進入
[three]        ⬅️ 4️⃣  進入

// 彈出時
[three]        ⬅️ 5️⃣  彈出（回傳 'Done!'）
[two]          ⬅️ 6️⃣  彈出
[one]          ⬅️ 7️⃣  彈出
[console.log]  ⬅️ 8️⃣  彈出（輸出 'Done!'）
```

- **最終執行順序**
函式執行順序（進入 Call Stack 順序）： 1️⃣ `one()` 2️⃣ `two()` 3️⃣ `three()`
函式結束順序（從 Call Stack 彈出順序）： 1️⃣ `three()` 2️⃣ `two()` 3️⃣ `one()` 4️⃣ `console.log()`（最後輸出 'Done!'）


- **總結**：
進入Call Stack的先後順序：
（底部-->頂部)
`global()` :arrow_right: `console.log(function)` :arrow_right: `one()` :arrow_right: `two()` :arrow_right: `three()`

:dart: 第一次啟動程式時，全域執行環境(Global Execution Context) 會被加到call stack中，其中包含全域變數(global variable)和詞彙環境(Lexical Environment)。詞彙環境是指程式碼在程式中的位置，可以參考[這篇文章](https://javascript.plainenglish.io/scope-chain-and-lexical-environment-in-javascript-eb1f6e60997e)。

</blockquote>
在JavaScript執行程式碼時，會由上而下、優先從全域(global)的程式碼開始執行，若全域的程式碼需要進入某個函式，再執行該函式，並把此函式加入stack的最上方，接著執行到函式中的return，函式便會移出堆疊(pop off)，以下程式碼舉例:
```javascript
function multiply(a, b) {    -multiply函式放入stack最上方
  return a * b         
}

function square(n) {       ， -square函式放入stack最上方，呼叫multiply函式
  return multiply(n, n)
}

function printSquare(n) {     -printSquare函式放入stack最上方，呼叫square函式
  let squared = square(n)
  console.log(squared)
}

printSquare(4)                -全域，呼叫printSquare函式        
```
   - **以非同步處理避免阻塞**
   以下程式碼為例：
```javascript
console.log('hi')

setTimeout(function () {
  console.log('there')
}, 5000)

console.log('JSConfEU')
```
由上而下、先執行全域的`console.log('hi')`（印出hi），接著執行`setTimeout`函式(開始計時五秒)，接著執行`console.log('JSConfEU')` (印出JSConfEU)，接著五秒倒數歸零，印出there。
:arrow_right:  <span style="background-color: gray;"> 所以console印出的順序會是：hi --> JSConfEU --> （5秒後）there </span>

### **3. Event Queue** (或稱Callback Queue)
**非同步程式碼（如 setTimeout, Promise 等）在執行時會將回調函式（callback function）放入 Callback Queue**。

當非同步操作完成後，回調函式並不會立即執行，而是進入 Callback Queue 等待執行。

前文中的例子顯示，在瀏覽器中可以同時處理多個事情，因為瀏覽器不是只有一個JavaScript Runtime(執行環境)，如前文中的`setTimeout`，是**瀏覽器提供的一個API(Web API)**，而非JavaScript engine本身的功能。

>:arrow_forward:關於執行環境(runtime)，這篇參考文章中有更詳細的說明：[從「為什麼不能用這個函式」談執行環境（runtime）](https://blog.huli.tw/2022/02/09/javascript-runtime/)
:arrow_forward: Web API: 如DOM、ajax、setTimeout、HTTP Request等等，可以幫助單線處理的JavaScript在瀏覽器中完成更多事。
更多WebAPIs可參考[MDN文件](https://developer.mozilla.org/en-US/docs/Web/API)。

`setTimeout` 中的 callback function（簡稱 cb）會被放到 WebAPIs 中，這時候，`setTimeout`已經執行結束，並從call stack中脫離。當計時時間到，就會將cb放入佇列(queue)中。**queue是以<span style="color:red">先進先出(First in first out，FIFO)</span>的方式執行，等待call stack中的任務清空，由事件循環監控，將queue中的任務傳入stack中依序執行**。

因此，簡單的說，**queue就像此單線的to-do-list，存放等待被執行的事件**，<span style="background-color: gray">因此，重複檢查queue中是否有任何需要被執行的事件，若有，則待call stack清空後，將佇列中的事件傳入call stack，直到queue為空；這樣**重複監控call stack與queue的循環，稱為事件循環(Event Loop)**。</span>
#### **範例**
```javascript
console.log('Start');  // 進入 Call Stack -> 輸出 'Start'

setTimeout(function() {
  console.log('Delayed');  // 進入 Callback Queue，等待 2 秒
}, 2000);

console.log('End');  // 進入 Call Stack -> 輸出 'End'

// 2 秒後，'Delayed' 會進入 Callback Queue，等待 Call Stack 清空後執行
```
輸出順序:
```javascript
Start
End
Delayed
```

#### **setTimeout 0 的意義**
```javascript
console.log('hi')

setTimeout(function () {
  console.log('there')
}, 0)

console.log('JSConfEU')
```
回到前文中的範例程式碼，假如把程式碼中setTimeout等候時間改為0，瀏覽器同樣先執行`console.log('hi')`，遇到setTimeout時間為0，將setTimeout的cb放入WebAPI的計時器中，等時間到再把此cb放入佇列(queue)，然而堆疊中任務尚未執行完畢，因此要等`console.log('JSConfEU')`也執行完畢，最後才會印出there。
:arrow_right:  <span style="background-color: gray;"> 所以console印出的順序會是：hi --> JSConfEU --> there </span>

setTimeout 的等待時間非執行的保證時間，而是要求執行環境處理所需的最少等待時間 <span style="color: crimson;">**(因此，不管設定時間是0秒或5秒，都必須等待call stack清空才可以將queue中的cb放入call stack執行)**。</span>

如MDN文件中的例子:
```javascript
(function() {

  console.log('this is the start')

  setTimeout(function cb() {
    console.log('this is a msg from call back')
  })

  console.log('this is just a message')

  setTimeout(function cb1() {
    console.log('this is a msg from call back1')
  }, 0)

  console.log('this is the end')

})()

// Console印出的順序如下:
// "this is the start"
// "this is just a message"
// "this is the end"
// "this is a msg from call back"
// "this is a msg from call back1"

```
## 小結
:pencil:　**最後，再用一個例子複習事件循環的過程：**
```javascript
console.log("First line")
function usingsetTimeout() {
    console.log("queue")
}
setTimeout(usingsetTimeout, 3000)
console.log("Last line")
```
1. 首先，`console.log("First line")`被加入<span style="color:Crimson" >**call stack**</span>並執行，印出'First line'，接著被移出<span style="color:Crimson" >**call stack**</span>

2. `setTimeout()`被加入<span style="color:Crimson" >**call stack**</span>

3. `setTimeout()`是非同步的，執行後，它會設定計時器，當計時器倒數至 0 後，`setTimeout` 函式本身會從<span style="color:Crimson" >**call stack**</span>中彈出。
> Note：計時期間，`usingsetTimeout` 並未進入<span style="color:ForestGreen" >**Event Queue**</span>，而是計時器歸零後，`usingsetTimeout` 才會被放入<span style="color:ForestGreen" >**Event Queue**</span>等待執行。

4. 同時，事件循環也不斷檢查<span style="color:Crimson" >**call stack**</span>是否都被執行完畢，當 <span style="color:Crimson" >**call stack**</span> 為空時，它才會從 <span style="color:ForestGreen" >**Event Queue**</span> 取出callback函式並執行。

5. `console.log("Last line")`被加入<span style="color:Crimson" >**call stack**</span>並執行，印出'Last line'，接著被移出<span style="color:Crimson" >**call stack**</span>
6. 等待 3000 毫秒：
在等待期間，`setTimeout`的callback函式，也就是`usingsetTimeout`並未進入<span style="color:ForestGreen" >**Event Queue**</span>中，它只是在等待計時器倒數結束。

7. 3000 毫秒後，計時器歸零，`usingsetTimeout` 被放入<span style="color:ForestGreen" >**Event Queue**</span>，等待執行。

8. 事件循環發現 <span style="color:Crimson" >**call stack**</span>現在空了，因此把<span style="color:ForestGreen" >**Event Queue**</span>中排在最前面的`usingsetTimeout`推進<span style="color:Crimson" >**call stack**</span>。 <span style="color:ForestGreen" >**Event Queue**</span> 進入 <span style="color:Crimson" >**call stack**</span> 的函式，仍然按照「先進先出（FIFO）」的順序執行。

9. `usingsetTimeout` 執行後，`console.log("queue")`被加入<span style="color:Crimson" >**call stack**</span>並執行，印出'queue'，然後被移出<span style="color:Crimson" >**call stack**</span>(遵守call stack後進先出LIFO的順序)。

10. 最後，`usingsetTimeout`被移出<span style="color:Crimson" >**call stack**</span>。
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:bulb: **In a nutshell：**
JavaScript 的 Event Loop（事件迴圈）是其非同步運作的核心機制，負責處理同步與非同步程式碼的執行順序，確保 JavaScript 仍然是單執行緒（single-threaded）但可以處理非同步操作（如 I/O、計時器、DOM 事件等）。

</blockquote>




---

## 參考資料
* [並行模型和事件循環(MDN Web Docs)](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/EventLoop#%E5%9F%B7%E8%A1%8C%E7%92%B0%E5%A2%83%E6%A6%82%E5%BF%B5%EF%BC%88runtime_concepts%EF%BC%89)
* [JavaScript - Event Loop](https://ithelp.ithome.com.tw/articles/10230871)
* [【JavaScript筆記】所以事件循環Event Loop到底是什麼？setTimeout 0 的藝術 ─ 我OK、你先請？](https://emilywalkdone.blogspot.com/2021/01/JavaScript-EVENT-LOOP.html)
* [理解JavaScript中的事件循環](https://pjchender.blogspot.com/2017/08/javascript-learn-event-loop-stack-queue.html)
* [What the heck is the event loop anyway? | Philip Roberts | JSConf EU](https://www.youtube.com/watch?v=8aGhZQkoFbQ&t=838s)
* [什麼是 Ajax？ 搞懂非同步請求 (Async request) 概念](https://tw.alphacamp.co/blog/ajax-asynchronous-request)
* [【筆記】搞懂 setTimeout 與 Event Loop 事件循環的關係](https://medium.com/@z88243310/%E7%AD%86%E8%A8%98-%E6%90%9E%E6%87%82-settimeout-%E8%88%87-event-loop-%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%92%B0%E7%9A%84%E9%97%9C%E4%BF%82-5f9fc5e5774)
* [Understanding JavaScript — Heap, Stack, Event-loops and Callback Queue](https://javascript.plainenglish.io/understanding-javascript-heap-stack-event-loops-and-callback-queue-6fdec3cfe32e)
* [設計看JS - 設計永遠搞不懂的同步、非同步 02](https://ithelp.ithome.com.tw/articles/10245127)
* [How dose JavaScript work?](https://june2.github.io/posts/Javasciprt/)
* Codecademy教材

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>