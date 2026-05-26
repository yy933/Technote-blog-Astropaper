---
title: "關於 RESTful API - 筆記"
pubDatetime: 2026-05-26T03:01:47.033Z
tags: ["HTTP","headers","Interview Preparation","API"]
description: " Table of contents ![](https://i.imgur.com/kcWp75M.png) <div..."
---

## Table of contents

![](https://i.imgur.com/kcWp75M.png)
<div style="text-align:center;font-size: 15px">
API、伺服器、資料庫的關係 <br>
(圖片為作者自製，轉載請註明來源)
</div>
<br/>

## :memo: 什麼是RESTful API
REST為Representational State Transfer的簡寫，意思是「表現層狀態轉換」， 是一種軟體架構風格，使系統間溝通更容易；而 RESTful API  是符合REST原則的API架構，能讓兩個電腦系統用來安全地透過網際網路交換資訊的介面，特點是它們是無狀態的(stateless)，並且將客戶端和伺服器端的關注點分離。
 

## :memo: REST設計架構的限制

### 客戶端和伺服器端關注點分離 (Client–server)
在REST架構風格中，客戶端的實作和伺服器的實作可以獨立完成，也就是說，**客戶端或伺服器端的程式碼隨時可更改，不影響另一邊的運作**；只要雙方知道要向對方傳送什麼樣格式的訊息，它們就可以保持模組化和獨立。將使用者介面、發送請求等問題(客戶端)與資料儲存、處理請求等問題(伺服器端)分開，可以提高跨平台介面的靈活性，並透過簡化伺服器元件來提高擴展性(scalability)與獨立發展性。

透過使用 REST 接口/介面(interface)，不同的客戶端可以存取相同的 REST 端點，客戶端只需要把關注點放在實現接口，執行相同的操作並接收相同的回應。

### 無狀態 (Statelessness)
遵循 REST 規範的系統是無狀態的，客戶端不需要知道伺服器端所處的狀態，反之亦然。客戶端發送的每個請求都是獨立的，而伺服器端不會保留客戶端的狀態。這種無狀態的特性，可以透過使用資源(resource)來執行，針對 REST 服務，伺服器通常使用[統一資源定位器](https://zh.wikipedia.org/zh-tw/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E5%AE%9A%E4%BD%8D%E7%AC%A6)（Uniform Resource Locator，縮寫：URL，或稱統一資源定位器、定位位址、URL位址，**俗稱網址**） 來執行資源識別。

:::info
:bulb: 關於**資源**，[AWS](https://aws.amazon.com/tw/what-is/restful-api/)做了清楚的解釋:
> 資源是指不同應用程式向其用戶端提供的資訊。資源可以是影像、影片、文字、數字或任何類型的資料。將資源提供給用戶端的機器也稱為伺服器。組織使用 API 共用資源並提供 Web 服務，同時維護安全、控制和身分驗證。此外，API 協助他們確定哪些用戶端可以存取特定的內部資源。
:::

這種特性的優點是RESTful應用程式更為可靠、高效能和可擴展，因為元件可以被管理、更新和重複使用，而不會影響整個系統。

### 可快取（Cacheability）
RESTful Web 服務支援快取，也就是在用戶端或中介裝置上，存放伺服器的回應在快取伺伺服器(cache server)中(例如可儲存在CDN、或其他雲端服務)，減少伺服器回應時間、提高效能。

### 統一介面（Uniform Interface）
統一介面是設計Restful API的基礎，伺服器以標準格式傳輸資訊，這種格式可以與伺服器應內部資源的格式不同。例如，伺服器可以將資料存放為文字，但以 HTML 格式傳送。

### 分層系統（Layered System）
在分層系統架構中，可以把web service設計在多層伺服器上，用戶端不會知道(也不需要知道)是否連接到最終的伺服器，仍然可以接收來自伺服器的回應，伺服器將請求傳遞至其他伺服器，共同運作並回傳回應滿足用戶端請求。例如，在伺服器A部署業務邏輯相關的API、伺服器A連接伺服器B進行驗證、同時也連接儲存資料的伺服器C

### 隨需編碼（Code-On-Demand）
伺服器可將可執行的程式碼(executable code)傳輸至用戶端，提高擴展性或增加用戶端功能。例如用戶端可以呼叫 API 取得 UI 元件的渲染程式碼。

## :memo: 客戶端與伺服器之間的溝通
在 REST 架構中，客戶端發送請求(request)以檢視或修改資源，而伺服器發送對這些請求的回應(response)。

### <span style="color: crimson">發送請求 (Making request)</span>
請求通常包括：
* HTTP 動詞(HTTP verb)，定義要執行的動作類型
* Header ，允許客戶端傳遞有關請求的訊息
* 資源的路徑(Path)
* 包含資料的訊息正文(此為選擇性)

#### HTTP 動詞
RESTful API 使用了 HTTP Method 來當作「動詞」。以下是幾個常見的動詞：
* POST：新增
* GET：讀取
* PUT：修改（修改整份文件）
* DELETE：刪除
* PATCH：修改（修改其中幾個欄位）

#### Header
在request的header中，客戶端發送它能夠從伺服器接收的內容類型。這稱為`Accept`參數，它可確保伺服器不會發送客戶端無法理解或處理的資料。這些內容類型稱為MIME type，詳細介紹請參考[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

#### 資源的路徑(Path)
Request必須包含應操作的資源的路徑。在 RESTful API 中，路徑的設計可協助客戶端了解正在發生的情況。
按照慣例，路徑的第一部分應該是資源的複數形式，這可以使路徑易於閱讀和理解。例如:
`sportswear.com/customers/173/orders/16`
以上路徑可以很容易理解，用戶端要請求的資源是id第173號的顧客的16號訂單。

### <span style="color: crimson">發送回應(Sending Responses)</span>

#### 內容類型(Content Type)
伺服器的response header中，`Content-type`欄位提醒客戶端它在response body中發送的資料類型。這些內容類型是 MIME Type，和在request header中`accept`欄位一樣。伺服器response傳回的內容類型，應為客戶端在request header中`accept`欄位中指定的選項之一。例如發送一個GET request:

Request:
```
GET /notes/23 HTTP/1.1
Accept: text/html, application/xhtml
```
Response header:
```
HTTP/1.1 200 (OK)
Content-Type: text/html
```

#### 狀態碼(Status code)
伺服器的回應的狀態碼，提醒客戶端有關操作是否成功的相關訊息。狀態碼非常多，以下是幾種常見的：
* 1XX 表示某種消息 (informational message)
* 2XX 表示成功
* 3XX 表示轉導
* 4XX 表示客戶端(Client)出錯
* 5XX 表示伺服器端(Server)出錯

更多狀態碼詳見[MDN文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

對於不同的HTTP動詞，伺服器在**成功**時傳回預期的狀態碼：

GET —  `200(OK)`
POST —  `201(CREATED)`
PUT — `200(OK)`
DELETE — `204(NO CONTENT)` 

如果操作**失敗**，則傳回與遇到的問題最相關的狀態碼。


## :memo: REST設計架構的優點
### 可擴展性(Scalability)
由於 REST 架構將客戶端與伺服器端關注點分離，並且伺服器不必保留過去的用戶端請求資訊與狀態，因此降低了伺服器負載。快取則減少了一些用戶端-伺服器的互動，這些特點皆提高了可擴展性，提高效能。

### 靈活性
RESTful API可讓用戶端與伺服器完全分離。這些服務將各種伺服器元件簡化、模組化，使每個元件都能獨立演進。伺服器端的變動不會影響用戶端。另外，分層系統能夠進一步提高靈活性。

### 獨立性
伺服器端與客戶端可以使用任何程式語言編寫應用程式，而不影響 API 設計。

## :memo: RESTful API 設計參考資源
設計:
[Best practices for REST API design](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
[GitHub Actions Artifacts - Use the REST API to interact with artifacts in GitHub Actions.](https://docs.github.com/en/rest/actions/artifacts?apiVersion=2022-11-28)

至於API文件和相關的規範，那又可以寫另一篇文章了:stuck_out_tongue_winking_eye:這裡先推薦幾個好用的工具:
[Swagger](https://swagger.io/)
[Postman API documentation tool](https://www.postman.com/api-documentation-tool/)
[RapiDoc](https://rapidocweb.com/)


## 參考資料
* [REST Architectural Constraints](https://restfulapi.net/rest-architectural-constraints/)
* [HTTP Request Method 設計行為與分析](https://hackmd.io/@monkenWu/Sk9Q5VoV4/https%3A%2F%2Fhackmd.io%2F%40gen6UjQISdy0QDN62cYPYQ%2FH1yxwXyNN?type=book)
* [什麼是 RESTful API？](https://aws.amazon.com/tw/what-is/restful-api/)
* [API 實作(一)：規劃 RESTful API 要注意什麼](https://noob.tw/restful-api/)
* [一杯茶的時間，搞懂RESTful API](https://apifox.com/blog/a-cup-of-tea-time-to-understand-restful-api/?utm_source=google_search&utm_medium=g&utm_campaign=15676663585&utm_content=137784982731&utm_term=&gclid=Cj0KCQjw9rSoBhCiARIsAFOiplkBR0Cf6yrND-aM56e5qSzpNZNLqbY9PEgdEZw2evMY6nxk4bJQlLkaAnFHEALw_wcB)
* Codecademy教材
* Alpha Camp教案