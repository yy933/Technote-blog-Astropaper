---
title: "HTTP Headers - 筆記"
pubDatetime: 2026-05-26T03:29:26.402Z
tags: ["HTTP","session","authentication","web security","headers"]
description: " Table of contents :memo: 什麼是HTTP Headers 無論是瀏覽器發送的request或伺..."
---

## Table of contents

## :memo: 什麼是HTTP Headers
無論是瀏覽器發送的request或伺服器回傳的response中，都包含headers，其中記錄該request或response的相關的相關資訊。這邊要談的是**伺服器回傳response中的headers**，其中包含日期、檔案大小及類型等基本資訊，以及安全性資訊如該網站使用的政策、通訊政策、或限制瀏覽器是否可以在其他網站的iframe中渲染當前網頁等。這些安全性相關的headers同時保護應用程式與使用者的安全，避免XSS等攻擊，同時提升了該網站的可信度，[提升其在SEO中的排名](https://anchordigital.com.au/holistic-seo-the-importance-of-http-security-headers/)。
<div style="text-align: center">
    
  ![](https://i.imgur.com/7yTaSyP.png)  
</div>

<p style="text-align: center">
Google Translate的Response Headers    
</p>

## :memo: 常見的安全性Headers

### HTTP Strict-Transportation-Security (HSTS)
這個header讓伺服器告訴瀏覽器必須使用HTTPS協定進行連線，確保用戶端與伺服器端的傳輸安全。
範例如下：
`Strict-Transport-Security: max-age=31536000; includeSubDomains`
* max-age：單位是秒，瀏覽器在接下來的這個期間(31536000秒，也就是一年)內，都會使用HTTPS
* includeSubDomains：告訴瀏覽器這個網站以及他的子網域都是HTTPS-Only。

更多詳情參考[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)

### Content-Security-Policy (CSP)
用來限制瀏覽器可以從哪裡載入資源，也就是列出一個白名單，放進所有可以載入資源的網域，避免XSS攻擊。
範例如下：
`Content-Security-Policy: script-src 'self'`
* script-src：限制可以載入JavaScript資源的地方
* self：代表瀏覽器只能從當前的網域載入JavaScript

更多詳情參考[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

### X-Frame-Options (XFO)
這個header可以防止當前的頁面被嵌入另一個網站HTML的[iframe](https://www.w3schools.com/tags/tag_iframe.ASP)中，尤其是釣魚網站，攻擊者將被害網頁放入iframe，使用CSS隱藏iframe，並讓使用者在不知情的情況下點擊並發送request到被害網站，這種攻擊稱為[Clickjacking](https://owasp.org/www-community/attacks/Clickjacking)
範例如下：
* `X-Frame-Options: DENY` 網頁無法被放在任何地方的iframe
* `X-Frame-Options: SAMEORIGIN` 網頁僅能放在同一個網域的iframe中
* `X-Frame-Options: ALLOW-FROM https://example.com` 網頁內容僅能被放在https://example.com的iframe中

更多詳請參考[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)


## 小結
上面簡介了幾種常見的安全性相關HTTP headers，headers的種類還有很多，例如[X-Content-Type-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)、[Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)等，在[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#security)及[OWASP Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)中有更詳細的介紹，也可以透過[Security Headers](https://securityheaders.com/)查詢特定網站的安全性headers。



---

## 參考資料
* [MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers#security)
* [HTTP security headers: An easy way to harden your web applications](https://www.invicti.com/blog/web-security/http-security-headers/)
* [HSTS設定](https://ithelp.ithome.com.tw/articles/10248473)
* [資安議題 — Http Security Header](https://medium.com/%E7%A8%8B%E5%BC%8F%E6%84%9B%E5%A5%BD%E8%80%85/%E9%97%9C%E6%96%BC%E5%AE%89%E5%85%A8%E6%80%A7%E7%9A%84header-b3b7adcb0fca)
* [Day11-記得要戴安全帽（一）](https://ithelp.ithome.com.tw/m/articles/10272394)
* [Holistic SEO: The Importance of HTTP Security Headers](https://anchordigital.com.au/holistic-seo-the-importance-of-http-security-headers/)
* [How to Implement Security HTTP Headers to Prevent Vulnerabilities?](https://geekflare.com/http-header-implementation/)


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>