---
title: "[Node.js] 用 Nodemailer 寄信 - 筆記"
pubDatetime: 2023-09-25T01:25:54.000Z
tags: ["Node.js","Express.js","npm"]
description: " Table of contents :memo: Nodemailer安裝與設定 首先參閱[Nodemailer的文件..."
---

## Table of contents

## :memo: Nodemailer安裝與設定
首先參閱[Nodemailer的文件](https://nodemailer.com/about/)。
### 安裝
`npm install --save nodemailer`
安裝完成後到package.json確認，沒問題就來進行接下來的設定，看一下文件的說明：
![](https://hackmd.io/_uploads/By9ijyRV2.png)
其實就是3個重點：
1. 先做一個transporter，可以使用[SMTP](https://www.geeksforgeeks.org/simple-mail-transfer-protocol-smtp/)或[其他傳輸機制](https://nodemailer.com/transports/)
2. 設定寄信的內容，包含寄件人、收件人、信件內容、主旨、附件等
3. 使用Nodemailer的`sendMail()`方法，透過先前產生的transporter將信件寄出

### 引入nodemailer
`const nodemailer = require('nodemailer')`

### 製作transporter
官方文件範例:
`let transporter = nodemailer.createTransport(options[, defaults])`
因為預設使用SMTP，所以如果使用SMTP機制的話，以下兩種寫法皆可：
`let  transporter = nodemailer.createTransport("SMTP", {smtpOptions});`
`let  transporter = nodemailer.createTransport({smtpOptions});`
完整範例如下，**帳號密碼可以用環境變數的方式**引入：

```nodejs
  let transporter = nodemailer.createTransport({
    host: "smtp.ethereal.email", // 寄信的host，
    port: 587,
    secure: false, // true for 465, false for other ports
    auth: {
      user: testAccount.user, // 代為寄信的帳號
      pass: testAccount.pass, // 該帳戶的密碼
    },
  });
```
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

#### 如果使用Gmail做為transporter
nodemailer已經知道知名的信件服務提供者的SMTP連線資訊，所以若使用[這些信件服務](https://community.nodemailer.com/2-0-0-beta/setup-smtp/well-known-services/)做為transporter，不需提供host、port等資訊，以下以gmail為例：
```nodejs
let transporter = nodemailer.createTransport({
     service: 'gmail', // no need to set host or port etc.
     auth: {...}
})
```
但是由於gmail對安全性的限制，這裡無法直接使用gmail帳號密碼做為驗證，必須設定**App password**：
1. 首先進入Google account頁面，點選左側**Security**
![](https://hackmd.io/_uploads/Bk-Z9gRVh.png)
2. 找到 How you sign in to Google的區塊，確認 2-Step Verification已經開啟
 ![](https://hackmd.io/_uploads/BJMUcxANh.png)
3. 進入 2-Step Verification頁面，拉到最下面，找到App passwords
 ![](https://hackmd.io/_uploads/HyHjqgRN2.png)
4. 點進App passwords，選擇app：Mail、device:(你的裝置)，按Generate。
5. 這時會產生一組password，這組密碼可以做為transporter中auth的密碼，**記得複製下來貼到.env裡面，視窗關了就看不到了**，必須重新產生一組。
#### 如果不想使用密碼
如果不想使用App password，又想利用gmail做為transporter，可以使用**Gmail API + OAuth2**的方式驗證，詳細作法參考:
* [Node.js - SEND Emails Using Nodemailer | Gmail | OAuth2](https://youtu.be/18qA61bpfUs)
* [How to Use Nodemailer to Send Emails from Your Node.js Server](https://www.freecodecamp.org/news/use-nodemailer-to-send-emails-from-your-node-js-server/)

</blockquote>
### 設定mail options
接著設定寄件內容，直接看範例：
```nodejs
let mailOptions = {
  from: "sender@server.com", //寄件人email
  to: "receiver@sender.com", //收件人email
  subject: "Message title",  //主旨
  text: "Plaintext version of the message", //信件內容，純文字
  html: "<p>HTML version of the message</p>" //信件內容，html
};
```
除了以上基本內容，還有更多進階內容可以設定，例如：
```nodejs
let mailOptions = {
    ...,
    cc: 'user1@example.com', //cc的對象
    bcc: 'user2@example.com', //bcc的對象
    attachments: [{
        filename: 'image1.jpg',
        path: 'filepath/image1.jpg'
    }], //附件
    headers: {
        'My-Custom-Header': 'header value'
    }, // headers內容
    date: new Date('2000-01-01 00:00:00') //日期
};
```
更多進階設定，參考文件[Message Configuration](https://nodemailer.com/message/)。

### 寄出信件
transporter和mail options都設定完之後，就可以用nodemailer提供的`sendMail()`方法寄出信件：
`transporter.sendMail(mailOptions)`
也可以加上callback回傳error和result，更多選項詳見[文件](https://nodemailer.com/usage/)：
```
transport.sendMail(mailOptions, (error, info) => {
　　if (error) {
       return console.log(error);
   　}
   console.log("Message sent: %s", info.messageId); 
   // Message sent: <b658f8ca-6296-ccf4-8306-87d57a0b4321@example.com>
})
```
## :memo: 實際操作 - 聯絡表單 Contact form
### 想法
製作一個聯絡表單，類似客服信箱的概念，使用者留言後按送出，表單內容會寄到客服信箱。

### 表單HTML
```[HTML]
  <form class="form" method="POST" action="/contact" onsubmit="return validation();">
     <div class="row">
	<div class="form-group col-md-6 my-3">
	    <input type="text" name="name" class="form-control" placeholder="Name" required>
	</div>
	<div class="form-group col-md-6 my-3">
	    <input type="email" name="email" class="form-control" placeholder="Email" required>
	</div>
	<div class="form-group col-md-12 my-3">
	    <input type="text" name="subject" class="form-control" placeholder="Subject" required>
	</div>
	<div class="form-group col-md-12 my-3">
            <textarea rows="6" name="message" class="form-control" placeholder="Your Message" required></textarea>
	</div>
	<div class="col-md-12 text-center my-3">
	    <button type="submit" value="Send message" name="submit" class="btn btn-contact fs-5" title="Submit Your Message!">Send Your Message!</button>
	</div>
      </div>
  </form>
```
### 把寄信功能封裝成一個helper
```nodejs
# ./helpers/email-helpers.js
const nodemailer = require('nodemailer')
const contactFormSend = async (name, email, subject, message) => {
  const transporter = nodemailer.createTransport({
    service: 'gmail',
    auth: {
      user: process.env.EMAIL,
      pass: process.env.APP_PASSWORD
    }
  })
  const mailOptions = {
    from: process.env.EMAIL,
    to: process.env.EMAIL,
    subject,
    html: `<h3>Name: ${name}</h3><br/><h3>Email: ${email}</h3><br/><h3>Message:</h3><br/><p>${message}</p>`
  }
  const sendMail = await transporter.sendMail(mailOptions)
  return sendMail
}

module.exports = contactFormSend
```
```nodejs
# .env
EMAIL=XXXX@gmail.com
APP_PASSWORD=your_password
```
### Route - POST /contact
* 首先利用Express架一個伺服器，專案入口設定為`app.js`(詳細步驟[點我](https://hackmd.io/@yy933/BJR_v_K_j))，接著到`app.js`引入email-helpers
`const contactFormSend = require('./helpers/email-helpers')`
* 傳送表單資料的路由
```nodejs
# app.js

if (process.env.NODE_ENV !== 'production') {
  require('dotenv').config()
}
const express = require('express')
const app = express()
const port = 3000
const contactFormSend = require('./helpers/email-helpers')
app.use(express.urlencoded({ extended: true }))
app.use(express.json())
// Send email from contact form
app.post('/contact', (req, res) => {
  try {
    //從req.body取得表單資料
    const { name, email, subject, message } = req.body
    // 傳入表單資料，利用email-helper寄信
    contactFormSend(name, email, subject, message)
    return res.redirect('/contact')
  } catch (error) {
    console.log(error)
    res.redirect('/contact')
  }
})

app.listen(port, () => {
  console.log(`App is listening at http://localhost:${port}`)
})
```
### 寄信功能試用
都設定完畢後，來試用看看：
1. 填寫表單並送出

![](https://hackmd.io/_uploads/S1-vJQCV3.gif)

2. 到信箱收信

![](https://hackmd.io/_uploads/S1pxeXCN3.jpg)

成功寄信啦!! :tada::tada::tada:

---

## 參考資料
* [Nodemailer文件](https://nodemailer.com/about/)
* [Node.js + Nodemailer : How to send Emails via SMTP with Nodemailer](https://devdotcode.com/node-js-nodemailer-how-to-send-emails-via-smtp-with-nodemailer/)
* [Node.js 系列學習日誌 #21 - 使用 nodemailer 套件透過 gmail 發送電子信箱](https://ithelp.ithome.com.tw/articles/10160766)


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>