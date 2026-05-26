---
title: "Cross-Site Request Forgery (CSRF) - 筆記"
pubDatetime: 2026-05-26T03:29:26.371Z
tags: ["web security","web attacks"]
description: " Table of contents :memo: 什麼是CrossSite Request Forgery (CSRF..."
---

## Table of contents

## :memo: 什麼是Cross-Site Request Forgery (CSRF)
Cross-Site Request Forgery (CSRF)，翻譯成跨站請求偽造，是一種常見的攻擊手法，使用者在不知情的情況下，代表了攻擊者向伺服器發送請求。以下簡單說明：
### **角色**
- A網站：銀行網站
- B網站：惡意網站
- 使用者

### **攻擊流程**
* 使用者登入Ａ網站，獲得Ａ銀行網站的cookie或auth token，並儲存到瀏覽器
* 使用者並未登出A網站(session是active狀態)，此時使用者瀏覽B網站，攻擊者將惡意連結藏在B網站中，誘使使用者點擊
* 使用者在不知情的情況下對A網站發送請求，惡意連結中，加入A網站的domain，例如：
 `http://bank.com/transfer?recipient=attacker&amount=100000`
* 因為使用者已經獲得A網站的auth token或cookie，若伺服器沒有其他條件驗證，攻擊者就可以獲得請求的內容。

通常，狀態改變(state-changing)的請求容易成為CSRF的攻擊目標，狀態改變意指伺服器的狀態改變，例如POST、PUT、DELETE等方法。常見的請求包含修改密碼、購物付款、轉帳、修改安全設定等，攻擊者經常使用社交軟體傳送惡意連結，例如內嵌在email或即時聊天室中。
## :memo: 如何防止CSRF攻擊
* 使用CSRF tokens，增加難以預測的參數，產生一個一次性、具有時效的token，先跟伺服器取得token，放進request header (authorization) 中，伺服器檢查是有效token才接受request。基於安全理由，**CSRF token並不會存在用戶端瀏覽器的cookie中**。
* 多一道驗證機制：對所有含有敏感資訊的操作，增加驗證機制，例如要求使用者再次輸入密碼等。
* 在伺服器端檢查referrer header：伺服器檢查請求的來源網域
* Set-Cookie設定SameSite屬性：
```
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Strict
Set-Cookie: <cookie-name>=<cookie-value>; SameSite=Lax
```
設定Strict：cookie只和same site的請求一起送出
設定Lax：只有特定的cross-site請求可以帶著cookie一起送出，詳情可以參考[這篇文章](https://medium.com/%E7%A8%8B%E5%BC%8F%E7%8C%BF%E5%90%83%E9%A6%99%E8%95%89/%E5%86%8D%E6%8E%A2%E5%90%8C%E6%BA%90%E6%94%BF%E7%AD%96-%E8%AB%87-samesite-%E8%A8%AD%E5%AE%9A%E5%B0%8D-cookie-%E7%9A%84%E5%BD%B1%E9%9F%BF%E8%88%87%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A0%85-6195d10d4441)




---

## 參考資料
* [[資安系列] 防禦CSRF攻擊的五種方法](https://gcdeng.com/blog/five-ways-to-defend-against-CSRF-attacks)
* [零基礎資安系列（一）-認識 CSRF（Cross Site Request Forgery）](https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-request-forgery/)
* [Understanding CSRF Attacks and Locking Down CSRF Vulnerabilities](https://kinsta.com/blog/csrf-attack/#:~:text=Typically%2C%20a%20CSRF%20attack%20involves,occur%20without%20the%20user's%20knowledge.)
* [Set-Cookie MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
* [HTTP Request Method 設計行為與分析](https://hackmd.io/@monkenWu/Sk9Q5VoV4/https%3A%2F%2Fhackmd.io%2F%40gen6UjQISdy0QDN62cYPYQ%2FH1yxwXyNN?type=book)
* [網站安全🔒 再探同源政策，談 SameSite 設定對 Cookie 的影響與注意事項](https://medium.com/%E7%A8%8B%E5%BC%8F%E7%8C%BF%E5%90%83%E9%A6%99%E8%95%89/%E5%86%8D%E6%8E%A2%E5%90%8C%E6%BA%90%E6%94%BF%E7%AD%96-%E8%AB%87-samesite-%E8%A8%AD%E5%AE%9A%E5%B0%8D-cookie-%E7%9A%84%E5%BD%B1%E9%9F%BF%E8%88%87%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A0%85-6195d10d4441)

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>