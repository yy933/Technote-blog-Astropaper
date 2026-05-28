---
title: "如何把專案部署到Railway上 - 筆記"
pubDatetime: 2023-09-25T01:25:38.000Z
modDatetime: 2026-05-25T10:04:23.062Z
tags: ["Node.js","database","MySQL","deployment"]
description: "Table of contents <blockquote class=\"my6 p4 bgorange50 dark..."
hackmd_id: "HyuwNxYXh"
---

## Table of contents

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

:bangbang: 2023/08 更新：Railway自2023年8月1日起[取消免費方案](https://blog.railway.app/p/pricing-and-plans-migration-guide-2023)，最低收費Hobby Plan每個月5美元，各方案詳細收費標準可參考[官網](https://railway.app/pricing)。

</blockquote>

## 專案簡介

用Node.js完成的互動式社交網站Simple Twitter，搭配關聯式資料庫MySQL(v8.0)，採前後端分離開發，這次要部署的是後端repo。

![](https://i.imgur.com/cbre3IM.gif)
<h4 style="text-align: center">專案前台登入操作示範</h4>


## Web Service部署
1. 先創一個[Railway](https://railway.app/)帳號
1. 完成後到**Dashboard**中按 **+ New Project**
1. 選擇要建立的project，這邊先介紹部署專案repo，所以先選 **Deploy from GitHub repo** --> **Configure GitHub App**，這邊會詢問GitHub的授權和要部署的repo，依個人需求選擇即可
1. GitHub授權完成後在 **New Project** 的區塊可以找到要部署的repo (如果沒看到可以重複3的步驟，就會看到剛剛選的repo)，點進去可以看到 **Deploy Now** 和 **Add variables** (設置環境變數)，這裡可以先把 **.env** 檔案中的環境變數填進去(這個步驟很重要，error常常是環境變數沒設定好發生的:fearful:)
![](https://i.imgur.com/dsW45oV.png)
3. 環境變數設定完，按**Deploy Now**就會開始部署
 ![](https://i.imgur.com/khW40Oo.png)
6. 部署完成點進 **View Logs** 看看是否有錯誤，如下圖Deploy Logs沒有error就成功啦！
![](https://i.imgur.com/McTwEQh.png)
7. 如果有錯誤可以檢查:
  - 環境變數是否正確：**Variables**中的環境變數和.env檔案中的環境變數是否一致
  -  **Settings** 中的設置，例如Start command是不是和啟動專案的command一樣，特別要注意檔案名稱，例如是node app.js 還是 node index.js 等等
  - 在**Settings** 中也可以設置Automatic Deployments，如果GitHub上的檔案有更新會自動重新部署；另外也可以自動產生Domain或設定客製化Domain
   ![](https://i.imgur.com/EvXsKHn.png)



## MySQL 資料庫部署
Railway提供了4種資料庫的部署，PostgreSQL、Redis、MongoDB、MySQL，這個專案使用的是MySQL：
- 點進Dashboard，找到剛剛部署好的專案點進去(注意是點進剛剛的專案再新增資料庫，不是直接在工作區新增一個資料庫，可以想成專案repo和資料庫是同捆包)，按右上角的 **+New** --> **Database** --> **Add MySQL**
- 很快的Railway幫我們新增了一個MySQL資料庫，點進資料庫找到**Connect**，有資料庫的連線資訊
![](https://i.imgur.com/bGGgaBJ.png)
- 打開MySQL WorkBench，新增一個連線，把這些資訊填進去，然後test connection，無誤的話可以順利進到資料庫工作區
- 這時候突然想到，還沒有把專案跟這個雲端資料庫連線，所以打開VScode來修改一下：
```node.js=17
  # config.json
  "production": {
    "use_env_variable": "DB_URL"
  },
```
```node.js=3
  # .env
  DB_URL=your_db_url
```
- 修改完再push到GitHub，注意Railway是否有自動重新部署，若沒有記得手動更新
- 資料庫現在是空的，我們要建立Schema並且把種子資料寫進去，這裡希望也可以用sequelize的指令操作，因此先下載[railway cli](https://docs.railway.app/develop/cli)，詳細安裝方式請看railway的文件，<span style="color: darkred; font-weight:bold">注意如果是以npm安裝要先升級Node.js的版本到16，不然無法安裝!!</span>
- 以下簡單寫一下CLI操作方式，**詳細都請參閱[官方文件](https://docs.railway.app/develop/cli)**：
  1. 安裝完Railway CLI，終端機輸入 `railway login` 或`railway login --browserless`依照指示登入Railway
  2. 和剛剛建立的專案連結，輸入後可以選擇專案和環境（預設是production）：
```
# Link to a project
railway link
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. Railway提供了可以在本地使用和Railway專案相同環境變數進行開發的指令：
```node.js=1
# Run <cmd> locally with the same environment variables as your Railway project
railway run <cmd>
# Run your Node.js project with your remote environment variables
railway run npm start
```
- 現在要來進行資料的migration，在終端機使用熟悉的sequelize指令(先確認已切換到production模式)：
`railway run npx sequelize db:migrate`
![](https://i.imgur.com/aGcanMe.jpg)
出現了錯誤，呼叫Google大神，找到了[這篇文章](https://stackoverflow.com/questions/52815608/er-not-supported-auth-mode-client-does-not-support-authentication-protocol-requ)
在MySQL Workbench或MySQL CLI執行以下SQL query：
`ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'password'`
<blockquote class="my-6 p-4 bg-red-50 dark:bg-red-950/30 border-l-4 border-red-500 rounded-r-md text-red-900 dark:text-red-200 blocknoted-fix">

:warning: 注意！這裡的`root`和`password`要改成Railway雲端資料庫的user和password，才有權限操作資料庫，不然會出現另一個錯誤`ERROR: Access denied for user '<wrong_username>'@'<your_ip>' (using password: YES)` (密碼或使用者名稱不對都無法操作資料庫，可以參考[這篇文](https://stackoverflow.com/questions/17975120/access-denied-for-user-rootlocalhost-using-password-yes-no-privileges))
![](https://i.imgur.com/FV08cSm.png)

</blockquote>
- 再執行一次資料migration，這次就成功了；同理寫入種子資料：`railway run npx sequelize db:seed:all`
- 回到Railway確認資料是否正確寫入：
![](https://i.imgur.com/xhCy1zf.png)
點進table看看：
![](https://i.imgur.com/tt30MfQ.png)
![](https://i.imgur.com/a4gFC4f.png)
都沒問題囉，到這裡就完成了！:tada::tada::tada:

## 其他功能
這裡補充幾個Railway很方便的功能：
- 在專案中可以直接用reference的方式引入環境變數(包含資料庫變數)，連複製貼上都不用，真貼心 :heart_eyes: 
![](https://i.imgur.com/WZYMAyN.gif)
- 設置shared variables：如果有多個專案要共用的環境變數，可以設置成shared variables，在其他專案也可以用reference的方式引入，相當方便!
![](https://i.imgur.com/cSNgIgA.gif)

- 可以邀請協作者編輯專案，但協作者無法看到環境變數，如果要分享讀取環境變數的權限可以使用token，協作者用CLI的指令讀取，詳細參考[文件](https://docs.railway.app/deploy/integrations#project-tokens) (此功能請小心使用):
`RAILWAY_TOKEN=XXXX railway run`
![](https://i.imgur.com/st5XArY.png)
 
## 小結
沒有了Heroku這個方便~~免費~~的服務只好尋找替代品，本來試著部署在[Render](https://render.com/)，雖然Render操作起來也非常之簡單易懂，無奈目前免費服務不提供MySQL資料庫部署(PostgreSQL是可以的，但90天到期後自動刪除資料庫)，所以選擇了Railway，雖然功能或許沒有GCP或AWS強大，但對於個人小專案已經足夠，目前免費方案提供每月500小時或5美元的額度。**(2023/08 更新：Railway自2023年8月1日起[取消免費方案](https://blog.railway.app/p/pricing-and-plans-migration-guide-2023)，最低收費Hobby Plan每個月5美元，各方案詳細收費標準可參考[官網](https://railway.app/pricing)。)**
Railway的服務相對新，部署的時候遇到問題參考資料也不是很多，只能靠官方文件和自己摸索，希望這篇文能幫助到需要使用Railway服務的人！

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>