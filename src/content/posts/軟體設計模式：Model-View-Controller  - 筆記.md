---
title: "軟體設計模式：Model-View-Controller  - 筆記"
pubDatetime: 2026-05-26T03:29:26.597Z
tags: ["Interview Preparation"]
description: " Table of contents 什麼是「關注點分離」？ 在應用程式開發的領域裡，相當重視 「關注點分離 (sepa..."
---

## Table of contents


## 什麼是「關注點分離」？
在應用程式開發的領域裡，相當重視 **「關注點分離 (separation of concerns, SoC) 」的設計原則**，從字面上的意思，就是把整個應用程式分拆成不同功能層 (layer) 或程式碼模組 (module)，每個區塊有各自的關注點，彼此分工合作，以**提高程式碼的可讀性、可維護性和擴展性**。

<div class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200">
關注點（Concern）指的是系統中負責處理某一個特定功能的部分。例如：

* UI 設計（顯示資訊）
* 資料存取（與資料庫互動）
* 業務邏輯（處理規則與計算）
* 日誌記錄與錯誤處理（監控系統運行）
* 權限驗證（確保安全性）

關注點分離的目標是 讓這些不同功能的部分彼此獨立，不要耦合在一起，以便開發和維護時可以更容易理解和修改。
</div>

## Model-View-Controller (MVC)

Model-View-Controller（MVC）設計模式是一個常見的關注點分離模式，在這個模式裡，會把軟體分成 Model、View、Controller 三大功能層。每一次 request/response 週期的背後，都由這三大功能層來合作完成。

### Model（模型）
負責數據（和資料庫溝通）和業務邏輯。
Model 管理的功能層被稱做「邏輯層」，更明確一點說，**是和「商業邏輯」有關的功能**，例如：

* 電商網站：
會員購物有九折、訂單超過一定的金額免運費
檢查登入帳號的類型，並依此開放不同權限
* 社交網站：判斷使用者彼此之間的友好程度
* To-do List：過了期但沒被執行的 to-dos 不能被刪除

### View（視圖）
負責 UI 顯示，View 所管理的功能層叫作「表現層 (presentation layer)」，顧名思義是負責管理畫面的呈現，也就是 HTML 樣板 (template)。

在開發框架裡，因為 HTML template 會有需要以動態顯示資料的情況 (也就是由 Model 取出的資料內容)，所以 View 會再進一步運用樣板引擎 (template engine) 將資料帶入 template。
### Controller（控制器）
負責處理請求，協調 Model 和 View。

Controller 常譯為「控制器」，它掌握使用者互動邏輯，也是應用程式收發 request/response 的核心。來自路由的 request 會先被送到 Controller，再由 Controller 通知 Model 調度資料，並且把資料傳遞給 View 來產生樣板 (template)，並將呈現資料的 HTML 頁面回傳給用戶端。

你可以把 Controller 想做是 MVC 架構的中間人，它決定了應用程式的工作流程 (workflow)，並且蒐集不同元件的工作結果，統一回傳給使用者。以下常見的設計問題，會由 Controller 來控制：

* 使用者是否需要先登入 (認證) 才可以看到網頁內容？
* 使用者是否只能閱讀資料，但不能修改或刪除？
* 使用者新增了資料之後，會重新導向至哪個頁面？