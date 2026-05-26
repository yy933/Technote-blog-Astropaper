---
title: "「在瀏覽器輸入一個網址，按下 ENTER，瀏覽器刷新、顯示出網頁」背後發生的事 - 筆記"
pubDatetime: 2023-02-16T23:56:28.000Z
tags: ["others"]
description: "1. 瀏覽器解析使用者輸入內容，解析網址URL( Uniform Resource Locator )， 依序處理 ： ..."
---

1. 瀏覽器解析使用者輸入內容，解析網址URL( Uniform Resource Locator )， 依序處理 ：
* 通訊協定（Protocol）：如http、https等
* 網域（Domain）：透過解析，對應到網站的IP位址 ，就像地址一樣
* 路徑（Path）：檔案的存放位置，分為絕對路徑與相對路徑
2. 建立連線後， 瀏覽器依照 IP 位置及埠號（port），將要送出的資訊打包成一個個的「封包」，經由 TCP( Transmission Control Protocol，傳輸控制協定 ) 連線開始傳輸， 瀏覽器向伺服器發送請求(request) 。
3. 瀏覽器接收到來自伺服器的回應(response)，可能關閉TCP連線或保留給另一個請求重複使用。
4. 瀏覽器確認回應的狀態： 包含 HTTP status code (狀態碼)以及一些其他訊息，例如 Content-Encoding, Cache-Control (瀏覽器如何快取頁面), Cookie 等 header。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**HTTP 狀態碼的意義：**
* 1XX 表示某種消息 (informational message)
* 2XX 表示成功
* 3XX 表示轉導
* 4XX 表示客戶端(Client)出錯
* 5XX 表示伺服器端(Server)出錯
5. 瀏覽器根據回應渲染(render)頁面：
* 回應HTML文件-->建造DOM tree，可以用JavaScript操作的資料結構
* 回應CSS-->建造CSSOM tree
* 將 DOM tree 和 CSSOM tree 合成一個 render tree
* 在 render tree 上執行 layout (reflow) 計算所有節點的幾何形狀
* 將所有節點繪製到畫面上