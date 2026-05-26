---
title: "動態路由 (Route Params) 和 查詢字串 (Query String) - 筆記"
pubDatetime: 2026-05-25T11:17:37.224Z
tags: ["Express.js","Node.js","cheatsheet"]
description: " Table of contents 動態路由 (Route Params) params 是在 URL 中設定的「變數..."
---

## Table of contents


## 動態路由 (Route Params) 
params 是在 URL 中設定的「變數」，它們會根據使用者訪問的路徑而改變。

範例：
```javascript
const express = require('express');
const app = express();
const movieList = require('./movies.json')

// 動態路由設定：冒號後面是變數名稱
app.get('/movies/:movie_id', (req, res) => {
  const movie = movieList.results.find(
    movie => movie.id.toString() === req.params.movie_id // 這裡對應到URL中的movie_id
  )
  res.render('show', { movie: movie })
})

app.listen(3000, () => {
  console.log('伺服器開在 http://localhost:3000');
});

```


### 對應到HTML
(這裡以Handlebars操作)
```html
<!-- show.handlebars -->
  <div class="row" id="data-panel">
    {{#each movies}}
    <div class="col-sm-3">
      <a href="/movies/{{this.id}}" class="text-secondary">
        <div class="card mb-2">
          <img class="card-img-top" src="https://movie-list.alphacamp.io/posters/{{this.image}}" alt="Card image cap">
          <div class="card-body movie-item-body">
            <h6 class="card-title">{{this.title}}</h6>
          </div>
        </div>
      </a>
    </div>
    {{/each}}
  </div>
```



## 查詢字串 (Query String)
用來「搜尋、篩選、控制顯示方式」等附加條件。

範例：
```javascript
app.get('/search', (req, res) => {
  const keyword = req.query.keyword
  const movies = movieList.results.filter(movie => {
    return movie.title.toLowerCase().includes(keyword.toLowerCase())
  })
  res.render('index', { movies: movies, keyword: keyword })
})
```
### 對應到HTML
(這裡以Handlebars操作)
```html
  <!-- search bar -->
  <div class="row" id="search-bar">
    <div class="col-12">
      <form action="/search">
        <div class="input-group mb-3">
          <input type="text" name="keyword" class="form-control" value="{{keyword}}" placeholder="Enter movie name to search..."
            aria-label="Movie Name..." aria-describedby="search-button">
          <div class="input-group-append">
            <button class="btn btn-outline-secondary" type="submit" id="search-button">Search</button>
          </div>
        </div>
      </form>
    </div>
  </div>
```
`<input>` 中的 `name="keyword"` 就是對應到 Express 中的 `req.query.keyword`，它會隨著網址中的 `?keyword=xxx` 一起被送到伺服器，因此就能用 `req.query.keyword` 取得使用者輸入的搜尋關鍵字。

### 優化使用者體驗
為了讓使用者搜尋後欄位不會清空，在 `<input>` 中這樣寫：
```html
<input type="text" name="keyword" class="form-control" value="{{keyword}}" placeholder="Enter movie name to search..."
            aria-label="Movie Name..." aria-describedby="search-button">
```
透過 `value="{{keyword}}"`，把伺服器傳回的 `keyword` 值帶入畫面中。

並記得在後端 Express 的路由中處理:
```javascript
app.get('/search', (req, res) => {
  const keyword = req.query.keyword
  const movies = movieList.results.filter(movie => {
    return movie.title.toLowerCase().includes(keyword.toLowerCase())
  })
  // 將 keyword 傳回模板引擎，讓前端 input 欄位可以保留使用者輸入的值
  res.render('index', { movies: movies, keyword: keyword })
})
```

## 小結
| 類型 | 動態路由（Route Params） | 查詢字串（Query String） |
|------|---------------------------|----------------------------|
| **語法範例** | `/users/:id` 👉 `/users/123` | `/search?keyword=book&page=2` |
| **用途** | 表示「路由的一部分」→ 像資料 ID、分類等 | 表示「附加條件」→ 篩選、排序、分頁等 |
| **取得方式（Express）** | `req.params.` | `req.query.` |
| **在網址中的位置** | 是 URL 路徑的一部分 | 是 URL 的「?」之後的部分 |
| **可否省略？** | 通常不能省略（除非用 optional param） | 可以省略 |