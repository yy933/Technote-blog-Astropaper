---
title: "關聯式資料庫(RDBMS)與非關聯式資料庫(NoSQL)的比較 - 筆記"
pubDatetime: 2023-09-25T01:26:12.000Z
tags: ["Interview Preparation","database"]
description: " Table of contents :memo: 關聯式資料庫(RDBMS, Relational Database ..."
---

## Table of contents

## :memo: 關聯式資料庫(RDBMS, Relational Database Management System)

### RDBMS是什麼
資料以資料表(table)的形式存在資料庫中，資料表之間有事先定義的關係，資料表中的一欄(column)代表某項屬性、一列(row)代表一個實體相關屬性的數值，每個資料表都具有主鍵(primary key)方便查詢，並且資料表間的關係可以透過外鍵(foreign key)建立。
### RDBMS的優點
* 資料完整性(Data Integrity)：RDBMS具有ACID的特性，ACID代表（參考[AWS關於RDBMS的介紹](https://aws.amazon.com/tw/relational-database/)）：
  - 不可分割性 (Atomic):交易必須整體成功執行，若是交易有一部分操作失敗，整個交易都會失效
  - 一致性 (Consistent)：做為交易的一部分寫入資料庫的資料，必須遵守所有明定規則以及約束 
  -  獨立性 (Isolated)：達成並行控制的重要關鍵，可以確保每一個交易都是獨立的 
  - 耐用性 (Durable)：在一個交易成功完成後，對資料庫所做的變更都是永久性的
* 查詢複雜性：可以透過SQL語法，進行較複雜的查詢
* 語言標準化:不像NoSQL各種資料庫有各自的操作語法，RDBMS都可使用SQL(Structured Querying Language)語言進行資料查詢與管理。
* 資料庫正規化
* 安全性高

### RDBMS的缺點
* 橫向擴展能力低，透過垂直擴展比較能發揮優勢，但是成本較高。
  * 橫向擴展(Horizontal Scaling)可以想像成增加機器的數量，用不同的機器進行同一個服務，除了減緩單一機器的負擔，也可以避免當單一機器故障時，整個服務就無法使用的狀況。
  * 垂直擴展(Vertical Scaling)在現有的硬體上進行升級，例如升級CPU、增加RAM等。
* 儲存與維護成本高
* 速度較慢，尤其當資料量龐大或多人同時使用服務的時候


## :memo: 非關聯式資料庫(NoSQL, Not only SQL)


### NoSQL是什麼
不同於SQL系統，NoSQL中的資料儲存不需要定義schema、也沒有固定架構，不保證ACID的特性，常用於分散式雲端系統。
### NoSQL的優點
* 橫向擴展能力佳 (Scalability):不必增加伺服器來擴大規模，可以透過分散式架構提供服務，以橫向擴展(Horizontal Scaling)的方式增加效能。
* 彈性較高 (Flexibility)：NoSQL不像關聯式資料庫需要schema，可以隨意定義資料模型，因此NoSQL可以處理無特定結構或半結構式(semi-structured)的資料
* 速度優勢：因為NoSQL不包含資料關聯性，查詢速度相對較快。

### NoSQL的缺點
* 資料完整度:不同於關聯式資料庫通常遵循ACID原則 (atomicity, consistency, isolation, durability)以確保資料的完整度，NoSQL較難提供ACID的保證，而是遵循BASE(basic availability, soft state, and eventual consistency) 的原則，並且可能犧牲資料的完整度。
* 語言標準化:不像關聯式資料庫大多可以使用SQL語言操作，NoSQL不同的資料庫有各自獨特的語言來管理資料。
* 查詢複雜性：NoSQL針對單一表格的查詢效果佳，但當資料複雜度增加，使用RDBMS的效果會更好。

### 常見的NoSQL類型
1. **Key-Value** 
  * 以key-value pair的形式儲存資料，key和value可以是任何形式(數字、字串、物件...)
  * key必須是獨一無二的，也就是說這種類型的資料庫最適合存放有獨一無二的key的資料，例如ID。 
  * 每筆資料各自獨立
  * [Amazon DynamoDB](https://aws.amazon.com/tw/dynamodb/)和[Redis](https://redis.com/)都是這類型的資料庫
2. **文件資料庫 (Document)**
  * 文件資料庫的資料，將資料儲存在分層結構(hierarchical structures)的文件中
  * 適合儲存非結構化資料，如HTML
  * 支援的文件格式多，如JSON, BSON, XML, and YAML等
  * 文件資料庫的模型被認為是key-value資料庫的延伸，但資料查詢並非僅依賴單一的key
  * 文件資料庫中的資料結構[範例](https://aws.amazon.com/tw/nosql/document/)如下(JSON)：
 ```
 [
    {
        "year" : 2013,
        "title" : "Turn It Down, Or Else!",
        "info" : {
            "directors" : [ "Alice Smith", "Bob Jones"],
            "release_date" : "2013-01-18T00:00:00Z",
            "rating" : 6.2,
            "genres" : ["Comedy", "Drama"],
            "image_url" : "http://ia.media-imdb.com/images/N/O9ERWAU7FS797AJ7LU8HN09AMUP908RLlo5JF90EWR7LJKQ7@@._V1_SX400_.jpg",
            "plot" : "A rock band plays their music at high volumes, annoying the neighbors.",
            "actors" : ["David Matthewman", "Jonathan G. Neff"]
        }
    },
    {
        "year": 2015,
        "title": "The Big New Movie",
        "info": {
            "plot": "Nothing happens at all.",
            "rating": 0
        }
    }
]
```
* [MongoDB](https://www.mongodb.com/?utm_campaign=academia_partners&utm_source=codecademy&utm_medium=referral)是一種受歡迎的文件資料庫
4. **圖形資料庫 (Graph)**
* 顧名思義，運用圖形結構儲存資料，在圖形結構中，資料存在節點(node/verticle)中，並且透過邊(edge)或線(line/links)進行連結、建立關係。
* 相較於關聯式資料庫，圖形資料庫的好處是建立、管理、查詢上都相對簡便
* [Neo4j](https://neo4j.com/)是一種熱門的圖形資料庫
5. **單欄式資料庫 (Column Oriented)**
* 儲存資料的方式和關聯式資料庫類似，但是在單欄式資料庫中，資料是以欄(column)的方式儲存，如下表：

  **Row-Oriented(關聯式資料庫)**

| ID | Product | Amount | Price |
|----|---------|--------|-------|
| 1  | Coffee  | 20     | 50    |
| 2  | Coke    | 3      | 20    |
| 3  | Milk    | 11     | 35    |

  **Column-Oriented(單欄式資料庫)**

| ID | Product |
|----|---------|
| 1  | Coffee  |
| 2  | Coke    |
| 3  | Milk    |

| ID | Amount |
|----|--------|
| 1  | 20     |
| 2  | 3      |
| 3  | 11     |

| ID | Price |
|----|-------|
| 1  | 50    |
| 2  | 20    |
| 3  | 35    |

* 例如上表中我們要取得產品價格的資料，在關聯式資料庫中必須將資料從各列拉出來，而在單欄式資料庫中，只需要加總價格資料表即可。單欄式資料庫的好處是可以快速擷取資料，可以減少需要載入的資料量，進行橫向擴展(Horizontal Scaling)提高傳輸量。
* [Amazon Redshift](https://aws.amazon.com/tw/redshift/)是一種熱門的單欄式資料庫。


## 小結
RDBMS與NoSQL並沒有絕對的好壞，端看使用的時機與需求




---

## 參考資料
* [[Day 4] NoSQL Database 的 BASE 特性](https://ithelp.ithome.com.tw/articles/10287859)
* [RDBMS vs. NOSQL | 關聯式資料庫 vs. 非關聯式資料庫](https://medium.com/@eric248655665/rdbms-vs-nosql-%E9%97%9C%E8%81%AF%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB-vs-%E9%9D%9E%E9%97%9C%E8%81%AF%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB-1423c9fbb91a)
* [圖形資料庫簡介，讓你對資料的關係一目瞭然！](https://ithelp.ithome.com.tw/articles/10237534)
* [什麼是關聯式資料庫？](https://aws.amazon.com/tw/relational-database/)
* [初步認識分散式資料庫與 NoSQL CAP 理論](https://oldmo860617.medium.com/%E5%88%9D%E6%AD%A5%E8%AA%8D%E8%AD%98%E5%88%86%E6%95%A3%E5%BC%8F%E8%B3%87%E6%96%99%E5%BA%AB%E8%88%87-nosql-cap-%E7%90%86%E8%AB%96-a02d377938d1)
* [[淺談]-NoSQL資料庫怎麼選？](https://xiang753017.gitbook.io/zixiang-blog/database/qian-tan-nosql-zi-liao-ku-zen-me-xuan)
* [什麼是 NoSQL？](https://aws.amazon.com/tw/nosql/)
* [[筆記] RDBMS v.s. NoSQL](https://shininglionking.blogspot.com/2018/04/rdbms-vs-nosql.html)


<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>