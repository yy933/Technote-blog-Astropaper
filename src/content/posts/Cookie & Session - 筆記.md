---
title: "Cookie & Session - 筆記"
pubDatetime: 2026-05-26T03:29:26.366Z
tags: ["HTTP","cookie","session","authentication","web security"]
description: " Table of contents :memo: HTTP 的無狀態 (Stateless) 特性 HTTP通訊協定(..."
---

## Table of contents

## :memo: HTTP 的無狀態 (Stateless) 特性
HTTP通訊協定(HTTP Protocol)具無狀態(Stateless)的特性，這表示每個伺服器所接收的HTTP request是獨立的，並且與先前接收到的request不相關。伺服器不會保存client的狀態，因此伺服器給予的response是根據當下的狀態。

**無狀態協定具有特性如下:**
* 簡化了伺服器的設計
* 因為系統不需要追蹤多個連結通訊與session的細節，因此無狀態協定所需資源較少。
* 在無狀態協定中，各個封包(packet)各自運輸，與其他封包無涉
* 伺服器與客戶端之間的依賴程度低

因此，如果我們希望伺服器能認出 request 的狀態 (例如之前已經登入)，我們可以讓 request 本身攜帶充足的資訊，來幫助伺服器判斷這個 request 是否為會員系統裡的某位使用者，再進一步判斷這位使用者的狀態。而所謂「讓 request 本身攜帶充足的資訊」，類似會員卡的概念，我們會**讓 request 攜帶一串憑證，伺服器可以用這串憑證進一步查找資料庫裡的使用者資訊，cookie就是達成這種目的的一種憑證**。

## :memo: 憑證

### 用戶端如何保存憑證
保存憑證的方法有很多種，瀏覽器本身提供了 cookie 和 web storage 等機制，如果有特殊目的，也可以拉出去儲成獨立的外部檔案。
![](https://i.imgur.com/Md0uYhL.png)
<p style="text-align:center">兩種不同的web session管理機制</p>
<h6 style="text-align:center;">(Source: Alpha Camp material)</h6>

### 伺服器如何比對 request 攜帶的憑證？
真正的使用者資料存在資料庫裡，所以在應用程式伺服器上也需要建立一個儲存機制，才能把用戶端、應用程式伺服器、資料庫三個地方的資訊匹配起來。

在應用程式伺服器上，用來存放使用者狀態的機制，稱為 session。
### **Session** 

![](https://i.imgur.com/MpFovoF.png)

<p style="text-align:center">session與cookie儲存使用者資訊的方式</p>
<h6 style="text-align:center">(Source: Alpha Camp material)</h6>

Session 翻譯成「會話」，這是抽象的概念，代表 **「用戶端與伺服器之間一系列的交流」**，你可以把一個「登入-登出」的週期視為一次「會話」。
* 通常「伺服器的 session」和「用戶端的 cookie」是相對存在的解決方案。
* session 也是一個 key-value pair 的結構，可以先想像成一個龐大的物件，裡面的資訊都和使用者登入記錄有關
* session通常儲存在：
  * 記憶體(預設)
  * 資料庫，如MySQL、MongoDB
  * cache，如Redis、Memcached  
* 常見的session攻擊如[session hijacking](https://owasp.org/www-community/attacks/Session_hijacking_attack)，攻擊者竊取session ID並送至伺服器，若伺服器未檢查session ID的使用者身分，即可成功獲取該ID使用者的相關資訊。
* 常見預防session攻擊的措施包含：
  - 增強session ID的長度、隨機性，使攻擊者難以猜測
  - 設置session的有效期限(time-out)
  - 增強cookie的安全性
  - 設置HTTPS-Only

### **Cookie**
Cookie是使用者瀏覽網站時，由伺服器建立，傳送給瀏覽器，並透過加密的方式儲存在用戶端（Client Side）上的小片段文檔(text)。
* Cookies是key-value pair 的結構，儲存session ID，不儲存session data(session data存在資料庫裡，[如圖所示](###Session))，由HTTP response header設置，有效期為直到瀏覽器關閉或cookie過期：
`Set-Cookie: Key=Value`　
實際上看起來像是`Set-Cookie: sessionID=91laR31m`
* Cookies可能會儲存敏感的個人資訊，例如個人偏好或瀏覽紀錄，所以應該確保cookie的安全性，例如透過

  **1. 增加cookie到期時間：**
`Set-Cookie: Key=Value; expires=Monday, 13-March-2023 09:20:10 GMT`
  **2. 增加HttpOnly的特性：**
  透過增加[HttpOnly](https://owasp.org/www-community/HttpOnly)的特性，確保cookie的資料不會因XSS攻擊(Cross-Site Scripting)經由JavaScript直接存取session cookie。
  `Set-Cookie: Key=Value; expires=Monday, 13-March-2023 09:20:10 GMT; HTTPOnly`
  **3. 增加其他特性：**
[SameSite](https://owasp.org/www-community/SameSite)：避免CSRF(Cross-Site Request Forgery)攻擊
[Secure](https://owasp.org/www-community/controls/SecureCookieAttribute)：確保只有https的網站可以獲得cookie的資料 
更多詳情可參考[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#samesitesamesite-value)關於Set-Cookie的解釋

* 通常儲存Cookie的地方包含：**記憶體或硬體**內，記憶體是由瀏覽器來維護的，通常會在瀏覽器關閉後清除，不同瀏覽器間的Cookie無法相互使用；硬體的Cookie則有保存期限，除非過期或是手動刪除，否則儲存時間會較瀏覽器來的久。
### **Session-Cookie運作模式**
1. 使用者進入網站，伺服器產生session和session ID
2. 伺服器的response header中可以Set-Cookie，寫入session id及其他屬性，瀏覽器收到response，儲存一個存有session ID的cookie(但不應包含任何私人資訊)
3. 存放有session ID的cookie自動連結在之後的HTTP request，傳送給伺服器
4. 伺服器接收到含有session ID的cookie的request時，解析session ID並回傳該ID相關連的session資料
5. 只要session 處於active狀態，此過程重複且持續進行
6. 當使用者關掉瀏覽器/登出/超過預先設定的session時限(如:30分鐘)，session和對應的cookie即過期。
### **利用Cookie與Session進行使用者認證**
![](https://i.imgur.com/vsSA0k9.png)<h6 style="text-align:center">(Source: Alpha Camp material)</h6>

* 第一次登入：伺服器建立一個 session :arrow_right: 把 session id 交給用戶端:arrow_right:用戶端把 session id 保存到瀏覽器的 cookie 裡。

* 下一次登入：同一個瀏覽器發送請求:arrow_right:附上同一組 session id:arrow_right:伺服器會判斷「這個 request 來自一個登入過的使用者」:arrow_right:這個瀏覽器可以使用授權的服務內容。

* 登出：把這個 session id 消滅掉:arrow_right:結束這一回合的會話 (session)。下次再進入網站時，又需要重新登入、建立新的 session。

### 使用者認證的安全性

cookie 保存在用戶端，而用戶端可以隨意修改自己的本地資訊。如果認證機制如此單純，使用者就可以修改自己的session id假冒他人登入，因此伺服器當然不可以允許這種事發生，不但資訊本身要加密，session 裡還需要有其他的驗證機制，防止用戶端偷改資訊以及避免資訊外洩。



---

## 參考資料
* Alpha Camp教案
* [Day25 來人上菜! 給我來點Cookie and Session](https://ithelp.ithome.com.tw/articles/10278733)
* [[不是工程師] Cookie 是文檔還是餅乾？簡述HTTP網頁紀錄會員資訊的一大功臣。](https://progressbar.tw/posts/91)
* [深入理解無狀態協議、HTTP 與 web 會話](https://blog.csdn.net/antony1776/article/details/83474496)
* [白話 Session 與 Cookie：從經營雜貨店開始](https://hulitw.medium.com/session-and-cookie-15e47ed838bc)
* [Difference between Stateless and Stateful Protocol](https://www.geeksforgeeks.org/difference-between-stateless-and-stateful-protocol/)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>