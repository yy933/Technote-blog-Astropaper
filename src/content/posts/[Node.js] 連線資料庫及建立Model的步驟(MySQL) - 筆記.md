---
title: "[Node.js] 連線資料庫及建立Model的步驟(MySQL) - 筆記"
pubDatetime: 2026-05-25T11:17:36.342Z
tags: ["Express.js","Node.js","cheatsheet","MySQL","Sequelize"]
description: " Table of contents <mark 1. 新增資料庫</mark 在MySQL Workbench中的Qu..."
---

## Table of contents


 

### <mark>  1. 新增資料庫</mark>
* 在MySQL Workbench中的Query頁面輸入指令：
```
drop database if exists <database_name>;
create database <database_name>;
use <database_name>;
```
執行指令後在Action Output出現如下訊息，成功新增資料庫：
![](https://i.imgur.com/cxTkjqs.png)

### <mark> 2. Express資料庫設定 </mark>

 #### 1. 安裝套件
`npm install mysql2 sequelize sequelize-cli`
#### 2. 初始化設定
  
  + **執行初始化腳本**:　`npx sequelize init`
    執行指令後，可以從終端機看到sequelize CLI生成了幾個資料夾：
 -- config/config.json：資料庫設定檔，已經自動帶入內容
 -- models/index.js：model 的設定檔
 -- migrations：資料庫設定檔的存放位置，目前是一個空資料夾
 -- seeders：種子資料設定檔的存放位置，目前是一個空資料夾
  
  + **資料庫設定檔config.json**: 打開config/config.json，已經自動生成三種模式的資料庫設定:development、test、和production，三種模式用到的資料庫都不同。
  -- 修改development模式的database和password
  -- 刪除 `"operatorsAliases": false` 整行 (在 Sequelize v5 以後被棄用)
### <mark> 3. 設定Model </mark>
* **使用 Sequelize-CLI 自動生成 model 設定檔**，參考[官方文件](https://sequelize.org/docs/v6/other-topics/migrations/#creating-the-first-model--and-migration-)：
`npx sequelize model:generate --name Todo --attributes name:string,isDone:boolean`
生成兩個檔案：
-- models/todo.js：model 設定檔
-- migrations/XXXXXXXXXXXXXX-create-todo.js：資料庫遷徙紀錄，檔名前綴為時間戳記。可以根據資料的設計加上allowNull: false等之後，再進行migration。

* **執行資料庫遷徙**：若要把專案裡的 migration 設定檔同步到資料庫，需要使用 `db:migrate` 指令：
    `npx sequelize db:migrate`
![](https://i.imgur.com/so303qK.png)
1. 系統先載入 config.json 設定檔，並且使用了裡面的 development 相關設定來登入 MySQL 資料庫。
1. 接著它執行了 migration 檔案裡的內容，migrating 表示程序開始， migrated 表示動作完成。

:arrow_forward: 到MySQL Workbench中執行 `select * from todos`，成功建立的資料表會顯示在Result Grid中：(資料表名稱以小寫複數命名)
    ![](https://i.imgur.com/HdrbY4N.png)