---
title: "[Node.js]利用Google Sheets API操作Google試算表"
pubDatetime: 2023-09-25T01:23:36.000Z
modDatetime: 2026-05-25T10:04:23.283Z
tags: ["Node.js","API","Express.js"]
description: "Table of contents :memo: 前言 當我們想要把資料放在Google Sheets上方便閱讀、計算..."
hackmd_id: "SkbnlqhT3"
---

## Table of contents

## :memo: 前言

當我們想要把資料放在Google Sheets上方便閱讀、計算等操作的時候，可以透過串接Google Sheets API製作、寫入資料。這裡介紹如何應用Node.js串接Google Sheets API，廢話不多說就開始實作吧！




## :memo: 取得Google憑證
首先，需要有Google憑證(credentials)才可以使用這個服務。
* **Creat New project:**
先進入[Google Cloud Console](https://console.cloud.google.com/getting-started)，登入後先建立一個新的專案：`Select a project` > `New Project`
* **Enable API:**
選擇側邊欄`APIs and services`> 接著點選 `Enabled APIs and Services`，再選擇畫面上方的 `+ ENABLE APIS AND SERVICES` > 在搜尋欄搜尋 google sheets API > 點進Google Sheets API，在Product details的畫面按 `Enable` > 稍後片刻等他開通 > 開通後跳轉到新的頁面，按側邊欄 `Credentials`
* **OAuth consent screen:** 
進入到Credentials頁面，選擇畫面上方的 `+ CREATE CREDENTIALS` > 選擇 `OAuth client ID`> 點選右邊的藍色按鈕 `configure consent screen` > 選擇User Type，此處我選External > 點選`CREATE` > 填入App information > `Save and Continue` > 接下來Scopes和Test Users都可以暫時不管他，一樣`Save and Continue` > 進入Summary頁面，按`Back to Dashboard`
* **Create OAuth client ID:**
這時總算可以來建立 OAuth client ID了，一樣按側邊欄 `Credentials` 進入到Credentials頁面，選擇畫面上方的 `+ CREATE CREDENTIALS` > 選擇 `OAuth client ID` > Application Type選`Web Application`、填寫Name > 填寫 Authorized redirect URIs（例如：`http://localhost:3000/oauth2callback`）> 點選`CREATE`  
* **Download credentials:**
建立成功後，會跳轉到`OAuth client created`的畫面，上面有`Clinet ID`和`Client Secret`的資訊，下方有`Download JSON`選項，按一下就會下載OAuth Client ID的憑證資訊，這個檔案要保存好，等一下會用到（先重新命名為`credentials.json`）。

## :memo: 專案實作
### 1. **前置作業(環境設置)**
   * Node.js
   * npm
    
### 2. **初始化專案並安裝相關套件**
```
mkdir sheetAPI
cd sheetAPI
npm init -y
npm i express @google-cloud/local-auth googleapis --save
```
專案架構如下：
```
sheetAPI
├─ index.js
├─ package-lock.json
└─ package.json
```
### 3. **建立Express伺服器**
建立一個基本的Express伺服器：
```javascript
//index.js
const express = require('express')
const app = express()
const port = 3000

app.listen(port, () => {
  console.log(`Express server is running on port ${port}`)
})
```
### 4. **進行驗證並串接Google Sheets API**
[Google官方文件](https://developers.google.com/sheets/api/quickstart/nodejs?hl=zh-tw)提供了一個快速設定的範例，稍微修改就可以：
```javascript
//index.js
...
const fs = require('fs').promises;
const path = require('path');
const process = require('process');
const {authenticate} = require('@google-cloud/local-auth');
const {google} = require('googleapis');

// If modifying these scopes, delete token.json.
const SCOPES = ['https://www.googleapis.com/auth/spreadsheets'];
// The file token.json stores the user's access and refresh tokens, and is
// created automatically when the authorization flow completes for the first
// time.
const TOKEN_PATH = path.join(process.cwd(), 'token.json');
const CREDENTIALS_PATH = path.join(process.cwd(), 'credentials.json');

// 檢查TOKEN_PATH中是否存放refresh token
async function loadSavedCredentialsIfExist() {
  try {
    const content = await fs.readFile(TOKEN_PATH);
    const credentials = JSON.parse(content);
    return google.auth.fromJSON(credentials);
  } catch (err) {
    return null;
  }
}

// 讀取CREDENTIALS_PATH中的Credentials進行驗證，產生refresh token並放進TOKEN_PATH中
async function saveCredentials(client) {
  const content = await fs.readFile(CREDENTIALS_PATH);
  const keys = JSON.parse(content);
  const key = keys.installed || keys.web;
  const payload = JSON.stringify({
    type: 'authorized_user',
    client_id: key.client_id,
    client_secret: key.client_secret,
    refresh_token: client.credentials.refresh_token,
  });
  await fs.writeFile(TOKEN_PATH, payload);
}

// 驗證：如果有refresh token，就用refresh token驗證製作access token；如果沒有就先產生refresh token
async function authorize() {
  let client = await loadSavedCredentialsIfExist();
  if (client) {
    return client;
  }
  client = await authenticate({
    scopes: SCOPES,
    keyfilePath: CREDENTIALS_PATH,
  });
  if (client.credentials) {
    await saveCredentials(client);
  }
  return client;
}
```
:pushpin: Points：
- `SCOPES`是根據我們所要使用的功能選擇授權的範圍，Google Sheets API提供的SCOPES可以看[這裡](https://developers.google.com/sheets/api/scopes?hl=zh-tw)。
- `CREDENTIALS_PATH`是憑證資料的路徑，也就是前面設定OAuth Clinet ID下載的那個JSON檔案，剛才已經重新命名為`credentials.json`，這裡我直接放在根目錄。
- `TOKEN_PATH`是用來放`token.json`的路徑，`token.json`每次進行驗證取得refresh token會自動產生；如果更改SCOPE，記得要把舊的`token.json`刪除。
- 目前專案架構如下：
```
sheetAPI
├─ credentials.json
├─ index.js
├─ package-lock.json
└─ package.json

```

### 5. **操作Sheet檔案**
驗證完成後，就可以根據選擇的SCOPE對Sheet進行操作，這裡簡單示範幾個功能：     

#### **建立新的試算表**
* [`spreadsheets.create`](https://developers.google.com/sheets/api/reference/rest/v4/spreadsheets/create)：官方提供的[範例](https://developers.google.com/sheets/api/guides/create)
在`index.js`中加入以下程式碼：
```javascript
// index.js
...
async function createSheet (auth, title) {
  try {
    const service = google.sheets({ version: 'v4', auth })
    const resource = {
      properties: {
        title
      }
    }

    const spreadsheet = await service.spreadsheets.create({
      resource,
      fields: 'spreadsheetId'
    })
    console.log(`Spreadsheet ID: ${spreadsheet.data.spreadsheetId}`)
    return spreadsheet.data.spreadsheetId
  } catch (error) {
    console.log(error)
  }
}
```
:pushpin: Points：
* `title`是sheet的標題，也可以加入其他內容，參考文件：https://developers.google.com/sheets/api/reference/rest/v4/spreadsheets#SpreadsheetProperties
* `spreadsheetId`是每一個sheet獨有的ID，觀察sheet的網址，粗體字的部分就是sheetId:
https://docs.google.com/spreadsheets/d/**<spreadsheet Id>**/edit?pli=1#gid=0

在`createSheet`函式下方再加入一個函式，測試看看：
```javascript
// index.js
...
async function run () {
  try {
    const auth = await authorize()
    const sheetId = await createSheet(auth, 'test')
  } catch (err) { console.error(err) }
}

run()
```
接著讓伺服器運作，成功的話，會在終端機上面看到：
`Spreadsheet ID: <Your spreadsheet Id>`
同時根目錄也新增了一個`token.json`檔案，也可以到web介面看看，網址:https://docs.google.com/spreadsheets/d/**<Your spreadsheet Id>**/edit?pli=1#gid=0
應該可以看到一個標題是test的試算表建立成功，也可以進入[個人的Google Sheets頁面](https://docs.google.com/spreadsheets/u/0/)，查看是否新增了一個新的試算表test：
:bell: <span style="color: crimson;">需用取得Google憑證時的Google account去看，目前其他人還沒有權限</span> <br>
![](/images/rJbMlkRa2.png)

    



#### **在試算表新增資料**
* [`spreadsheets.values.update`](https://developers.google.com/sheets/api/reference/rest/v4/spreadsheets.values/update?hl=zh-tw)：官方提供的[範例](https://developers.google.com/sheets/api/guides/values?hl=zh-tw)
在`index.js`中加入以下程式碼：
```javascript
// index.js
...
async function updateValues (auth, spreadsheetId) {
  const service = google.sheets({ version: 'v4', auth })
  const values = [
    ['Item', 'Cost', 'Stocked', 'Ship Date'],
    ['Wheel', '$20.50', '4', '3/1/2016'],
    ['Door', '$15', '2', '3/15/2016'],
    ['Engine', '$100', '1', '3/20/2016'],
    ['Totals', '=SUM(B2:B4)', '=SUM(C2:C4)', '=MAX(D2:D4)']
  ]
  const resource = {
    values
  }

  const result = await service.spreadsheets.values.update({
    spreadsheetId,
    range: 'Sheet1!A1:D',
    valueInputOption: 'USER_ENTERED',
    resource
  })
  console.log('%d cells updated.', result.data.updatedCells)
  return result
}
```
:pushpin: Points：
* `values`:試算表範圍內的資料，也就是要操作的資料，values是必填的參數，型態是陣列(array)。其他參數參考[文件](https://developers.google.com/sheets/api/reference/rest/v4/spreadsheets.values?hl=zh-tw#resource-valuerange)。    
* `valueInputOption`是指系統如何解讀輸入資料。例如這裡使用`USER_ENTERED`，則系統會解析`Total`那列的`'=SUM(B2:B4)'`，加總B2:B4的值，就和在Excel資料表裡面操作一樣的方法。其他選項參考[文件](https://developers.google.com/sheets/api/reference/rest/v4/ValueInputOption)。
* `range`是指資料的範圍，可以指定多個儲存格 (例如 A1:D5) 或單一儲存格 (例如 A1)。詳細用法參考[文件](https://developers.google.com/sheets/api/reference/rest/v4/spreadsheets.values?hl=zh-tw#resource-valuerange)。或是[範例](https://developers.google.com/sheets/api/guides/values?hl=zh-tw)中也有提到:
![](/images/SJhygyC6n.png)


在`run()`中加入：
```javascript
// index.js
...
async function run () {
  try {
    ...
    await updateValues(auth, sheetId)
  } catch (err) { console.error(err) }
}

run()
```
同樣讓伺服器運作，成功的話，會在終端機上面看到：
```
Spreadsheet ID: <Your spreadsheet Id>
20 cells updated.
```
到web介面看看，這時可以看到test試算表中有資料了!
    ![](/images/Sk4qS2663.png)
    
#### **讀取試算表的資料**
* [`spreadsheets.values.get`](https://developers.google.com/sheets/api/reference/rest/v4/spreadsheets.values/get?hl=zh-tw)：官方提供的[範例](https://developers.google.com/sheets/api/guides/values?hl=zh-tw)
一樣根據範例稍微改寫，在`index.js`中加入以下程式碼：
```javascript
// index.js
...
async function getValues (auth, spreadsheetId) {
  try {
    const service = google.sheets({ version: 'v4', auth })
    const result = await service.spreadsheets.values.get({
      spreadsheetId,
      range: 'Sheet1!A1:D2'
    })
    const numRows = result.data.values ? result.data.values.length : 0
    console.log(`${numRows} rows retrieved.`)
    return result.data.values
  } catch (error) {
    console.log(error)
  }
}
```
在`run()`中加入：
```javascript
// index.js
...
async function run () {
  try {
    ...
    const result = await getValues(auth, sheetId)
    return console.log(result)
  } catch (err) { console.error(err) }
}

run()
```
同樣讓伺服器運作，成功的話，會在終端機上面看到：
```
Spreadsheet ID: <Your spreadsheet Id>
20 cells updated.
2 rows retrieved.
[
  [ 'Item', 'Cost', 'Stocked', 'Ship Date' ],
  [ 'Wheel', '$20.50', '4', '3/1/2016' ]
]
```
成功讀取了A1:D2這個範圍的資料，也就是Web介面上：
![](/images/rkN8xkCT3.png)



## 小結
以上簡單介紹了Google Sheets API的一些用法，還有很多其他的功能都可以參考[API文件](https://developers.google.com/sheets/api/guides/concepts?hl=zh-tw)。前面介紹了使用OAuth Client ID進行驗證的方法，另外也可以使用`Service Account`進行驗證，驗證方式可以參考[這篇文章](https://www.section.io/engineering-education/google-sheets-api-in-nodejs/)；使用`Service Account`驗證須注意權限，如要用自己的gmail帳號登入需要開啟權限，除了前面這篇文章中使用的方式（先用Gmail帳號開新的Sheet再分享權限給`Service Account`）之外，另外也可以使用`Google Drive API`給予權限，詳情可以參考[這篇文章](https://www.daimto.com/google-drive-api-with-a-service-account/#Create_Google_Drive_API_V3_service_object)，以及`Google Drive API`文件[`permissions.create`](https://developers.google.com/drive/api/reference/rest/v3/permissions/create)或[`permissions.update`](https://developers.google.com/drive/api/reference/rest/v3/permissions/update)，當然直接[使用Google Drive API建立Sheet](https://developers.google.com/drive/api/guides/create-file)也是可以的。


---

## 參考資料
* [Google Sheets API文件](https://developers.google.com/sheets/api/guides/concepts?hl=zh-tw)
* [使用 Node.js 操作 Google Sheets API 讀寫試算表資料庫](https://www.wfublog.com/2023/04/nodejs-google-sheets-api-read-write.html)
* [How to use Google sheet as your database with Node.js](https://medium.com/@sakkeerhussainp/google-sheet-as-your-database-for-node-js-backend-a79fc5a6edd9)


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>