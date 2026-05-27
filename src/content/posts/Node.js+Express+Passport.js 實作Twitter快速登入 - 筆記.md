---
title: "Node.js+Express+Passport.js 實作Twitter快速登入 - 筆記"
pubDatetime: 2023-05-28T05:49:50.000Z
tags: ["Node.js","Express.js","npm","Passport.js","Authentication"]
description: "Table of contents :memo: 串接Twitter API 基本設置與Token取得 首先先到[Tw..."
hackmd_id: "BJx366g8h"
---

## Table of contents

## :memo: 串接Twitter API
### 基本設置與Token取得
首先先到[Twitter Developer](https://developer.twitter.com/en/apply-for-access)建立一個帳戶，完成後到左邊選單Projects&Apps新增一個Project，並在Project裡面建立一個App(免費方案目前只有一個App的額度)，接著選擇剛剛建立的APP，找到頁面下方**User authentication settings**，點**Edit**，需要填寫以下欄位:
* App permissions: 選擇需要的權限，這裡只是要使用者的帳戶名稱，所以選Read；另外也可以要求使用者的email，但是必須提供privacy policy和terms of service的連結，詳細方式見[Twitter文件](https://developer.twitter.com/en/docs/twitter-api/v1/accounts-and-users/manage-account-settings/api-reference/get-account-verify_credentials)。
* Type of App：這裡選Web App, Automated App or Bot
* App info:
  * Callback URI / Redirect URL:在用戶端 OAuth 設定的重新導向 URI，注意如果是在localhost上執行，網域 http://localhost:XXXX ，這裡是[無法作為Callback](https://developer.twitter.com/en/docs/apps/callback-urls)的，可以使用如  http(s)://127.0.0.1  (localhost IP) 或是用[ngrok](https://ngrok.com/)等服務建立一個localhost和外網的tunnel，產生一個https的網址，作為這裡的callback URI/ Redirect URL。
  * Website URL: 和callback URI一樣，不要填localhost。

確定都填了就可以按save。

回到Projects&APP頁面，選擇 **Keys and tokens**，這裡有幾種不一樣的token，由於這次要利用passport協助驗證，先看看[passport文件](https://github.com/jaredhanson/passport-twitter)中要求的token：
```
passport.use(new TwitterStrategy({
    consumerKey: TWITTER_CONSUMER_KEY,
    consumerSecret: TWITTER_CONSUMER_SECRET,
    callbackURL: "http://127.0.0.1:3000/auth/twitter/callback"
  },
  function(token, tokenSecret, profile, cb) {
    User.findOrCreate({ twitterId: profile.id }, function (err, user) {
      return cb(err, user);
    });
  }
));
```

需要的token是consumerKey、consumerSecret，因此選擇**Consumer Keys**這欄產生一組API Key and Secret，記得先貼到.env裡面，關起來之後再打開就不會重複顯示了，必須重新產生一組。
```
# .env
TWITTER_CONSUMER_KEY=your_key
TWITTER_CONSUMER_SECRET=your_secret
```

### Passport.js 驗證策略


### 登入路由設置
需設定兩條路由：
* 使用者要求使用 Twitter 帳號登入的按鈕：/auth/twitter

* Twitter 獲得使用者同意之後，將使用者資料發送給 Express 的位址： /auth/twitter/callback
這條路由必須和在Twitter Developer console上填的callback URI一樣

### 發送登入請求


### 運用回傳資料進行資料庫操作


---

## 參考資料
* [passport-twitter文件](https://github.com/jaredhanson/passport-twitter)



<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>