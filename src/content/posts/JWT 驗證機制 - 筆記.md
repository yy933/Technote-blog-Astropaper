---
title: "JWT 驗證機制 - 筆記"
pubDatetime: 2023-09-25T01:24:40.000Z
tags: ["authentication","HTTP","web security","JWT","token"]
description: " Table of contents :memo: HTTP 的無狀態 (Stateless) 特性 HTTP通訊協定(..."
---

## Table of contents

## :memo: HTTP 的無狀態 (Stateless) 特性
HTTP通訊協定(HTTP Protocol)具無狀態(Stateless)的特性，這表示每個伺服器所接收的HTTP request是獨立的，並且與先前接收到的request不相關。伺服器不會保存client的狀態，因此伺服器給予的response是根據當下的狀態；也就是說，client端發送的第一個request進行登入，在第二個request發送時，伺服器無法得知第二個request是否來自同一個使用者。

因此，如果我們希望伺服器能認出 request 的狀態 (例如之前已經登入)，我們可以讓 request 本身攜帶充足的資訊，來幫助伺服器判斷這個 request 是否為會員系統裡的某位使用者，再進一步判斷這位使用者的狀態，這是一種「交換憑證」的概念，[先前](https://hackmd.io/@Fb1mLcumT_GI60TovN10dQ/ryWRxZCns)提到的cookie與session機制為達成交換憑證的策略之一，然而，cookie的值限定在特定網域中可以被存取，在前後端分離的開發模式中，如果後端API站和前端網站部署在不同網域，則無法跨網域使用瀏覽器提供的cookie機制，因此，可以採用token-based的機制來進行交換憑證。

## :memo: 憑證
token-based authentication的概念和介紹[cookie-session的文章](https://hackmd.io/@Fb1mLcumT_GI60TovN10dQ/ryWRxZCns)中提到的cookie-based authentication概念類似，都是**辨認憑證中的資訊**來進行管理，達成用戶端與伺服器端的資訊傳遞，只是此處的憑證是token。

![](https://i.imgur.com/Md0uYhL.png)
<p style="text-align:center">兩種不同的web session管理機制</p>
<h6 style="text-align:center;">(Source: Alpha Camp material)</h6>

## :memo: JSON Web Token (JWT) 簡介

### Token的組成 
JWT屬於JSON物件，其中token的組成包含三個部分：
* **Header** - header由兩個部分組成，包含token**類型**(也就是JWT，media type [application/JWT](https://www.iana.org/assignments/media-types/application/jwt))以及產生signature所使用的**演算法**，如HMAC SHA256或RSA等。
```
{
  'alg': 'HS256',  
  'typ': 'JWT'
}
```
* **Payload** - **token要攜帶的資訊**，如 user_id 與時間戳記，也可以指定 token 的過期時間。預設不加密，因此不建議把敏感資訊放在payload。
```
{
  'sub': '12345678',
  'user_id': '1',  
  'name': 'Winnie',
  'admin': false
}
```
* **Signature** - 根據 Header中的演算法，將header、 Payload，加上密鑰 (secret) 進行雜湊，產生一組不可反解的亂數，當成簽章，用來驗證 JWT 是否經過篡改。secret只有伺服器知道，不可洩漏給用戶端。
```
HMACSHA256(
      base64UrlEncode(header) + "." +
      base64UrlEncode(payload),
      secret)
```

### 如何用JWT與伺服器溝通
1. 使用者登入，使用者資訊傳送到伺服器
1. 伺服器產生一組JWT，並將JWT回傳給瀏覽器(secret由伺服器端設置，經和payload、header雜湊後產生一組亂數再回傳給瀏覽器，用戶端不會知道secret)
1. 使用者傳送另一個request，接著瀏覽器將JWT傳回伺服器
1. 伺服器透過secret比對驗證JWT signature，並獲得JWT中的使用者資訊
1. 伺服器回傳Response。如果JWT有效，瀏覽器會得到他所request的內容(使用者資訊)；如果JWT無效，瀏覽器會得到錯誤訊息。

### 儲存JWT
關於JWT應該存在哪裡至今仍然眾說紛紜，以下簡單說明：
* 避免儲存在LocalStorage/SessionStorage，因為可以輕易地透過XSS攻擊取得資料。
* 儲存在cookie看起來是個可行的方式，不過也必須注意透過CSRF攻擊竊取資料的風險，有些框架具有防堵的機制；另也有人主張存在記憶體。詳細可參考以下討論：[1](https://stackoverflow.com/questions/27067251/where-to-store-jwt-in-browser-how-to-protect-against-csrf)、[2](https://stackoverflow.com/questions/48712923/where-to-store-a-jwt-token-properly-and-safely-in-a-web-based-application)、[3](https://medium.com/a-layman/where-should-we-store-the-jwt-for-spa-memory-cookie-or-localstorage-2491912d8e79)

### 為何要使用JWT?
* JSON物件的形式多種程式語言都支援
* 體積小、方便儲存資訊、在手機app上運作方便
* JWT本身包含驗證的資訊，不像session資訊需存在資料庫伺服器，減輕伺服器的負擔

### 利用JWT實作登入機制
在[JWT官網](https://jwt.io/libraries)列出了各種語言適用的套件，此處使用Node.js搭配Passport建立登入機制，因此安裝jsonwebtoken以及[passport-jwt](https://www.passportjs.org/packages/passport-jwt/):
`npm install jsonwebtoken passport-jwt`
#### **流程：**
1. 在user-controller中檢查信箱/密碼 --> 找到對應的user --> 簽發JWT，回傳給client端
2. user拿到JWT後使用服務，此時以JWT進行身分驗證
![](https://i.imgur.com/1ZBQib3.png)
<h6 style="text-align:center;">(Source: Alpha Camp material)</h6>



---

## 參考資料
* Alpha Camp教案
* [登入機制驗證，淺談JSON Web Tokens(JWT)](https://ithelp.ithome.com.tw/m/articles/10289553)
* [JWT(JSON Web Token) — 原理介紹](https://medium.com/%E4%BC%81%E9%B5%9D%E4%B9%9F%E6%87%82%E7%A8%8B%E5%BC%8F%E8%A8%AD%E8%A8%88/jwt-json-web-token-%E5%8E%9F%E7%90%86%E4%BB%8B%E7%B4%B9-74abfafad7ba)
* [一文帶你搞懂JWT 常見概念& 優缺點](https://tw511.com/a/01/45019.html)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>