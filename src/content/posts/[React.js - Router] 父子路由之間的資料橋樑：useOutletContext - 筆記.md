---
title: "[React.js - Router] 父子路由之間的資料橋樑：useOutletContext - 筆記"
pubDatetime: 2026-07-21T07:05:10.631Z
tags: ["React.js","React Router","Nested Route","Advanced React","Frontend"]
description: "Table of contents 為什麼需要 useOutletContext？ 在 React Router (v..."
hackmd_id: "HyEs8ch4Ml"
---

## Table of contents

## 為什麼需要 `useOutletContext`？  
在 React Router (v6+) 中，當我們使用 Nested Routes（巢狀路由） 時，子頁面元件（Child Component）並不是直接手動寫在父頁面的 JSX 裡面，而是透過 `<Outlet/>` 這個占位符（placeholder）動態渲染出來的。

這會導致傳統的 `props` 傳參數方式失效：

```javascript
/* ❌ 無法這樣寫：無法直接傳送特定 Props給 <Outlet /> 裡面的子元件 */
<Parent>
  <Child currentVan={currentVan} />
</Parent>
```

為了解決「父路由（Layout）如何傳遞資料給子路由」的問題，React Router 提供了 `useOutletContext`。可以把它想像成「專屬於特定路由樹區域的輕量版 Context API」。

## 語法拆解與基礎範例
### 父路由（Parent Route）：使用 `<Outlet context="{...}">` 分派資料  
在父元件取得 API 資料或宣告 State 後，透過 `<Outlet/>` 的 `context` 屬性將資料打包傳出：

```javascript
// pages/Host/HostVanDetails.jsx
import React from "react"
import { useParams, Link, NavLink, Outlet } from "react-router-dom"

export default function HostVanDetail() {
    const { id } = useParams()
    const [currentVan, setCurrentVan] = React.useState(null)

    const activeStyles = {
        fontWeight: "bold",
        textDecoration: "underline",
        color: "#161616"
    }

    React.useEffect(() => {
        fetch(`/api/host/vans/${id}`)
            .then(res => res.json())
            .then(data => setCurrentVan(data.vans))
    }, [id])

    if (!currentVan) {
        return <h1>Loading...</h1>
    }

    return (
        <section>
            <Link to=".." relative="path" className="back-button">
                &larr; <span>Back to all vans</span>
            </Link>

            <div className="host-van-detail-layout-container">
                <div className="host-van-detail">
                    <img src={currentVan.imageUrl} alt={currentVan.name} />
                    <div className="host-van-detail-info-text">
                        <i className={`van-type van-type-${currentVan.type}`}>{currentVan.type}</i>
                        <h3>{currentVan.name}</h3>
                        <h4>${currentVan.price}/day</h4>
                    </div>
                </div>

                <nav className="host-van-detail-nav">
                    <NavLink to="." end style={({ isActive }) => isActive ? activeStyles : null}>
                        Details
                    </NavLink>
                    <NavLink to="pricing" style={({ isActive }) => isActive ? activeStyles : null}>
                        Pricing
                    </NavLink>
                    <NavLink to="photos" style={({ isActive }) => isActive ? activeStyles : null}>
                        Photos
                    </NavLink>
                </nav>

                {/* 💡 核心重點：透過 context 屬性將 currentVan 共享給所有嵌套子路由 */}
                <Outlet context={{ currentVan }} />
            </div>
        </section>
    )
}
```

### 子路由（Child Route）：呼叫 useOutletContext() 接收資料  
任何渲染在該 `<Outlet/>` 位置的子頁面（例如 Details、Pricing、Photos），都可以透過 `useOutletContext()` 直接取得父元件共享的狀態，無需重複 Fetch 資料：

```javascript
// pages/Host/HostVanInfo.jsx
import React from "react"
import { useOutletContext } from "react-router-dom"

export default function HostVanInfo() {
    // 💡 核心重點：透過 Hook 直接解構出父層傳過來的 currentVan
    const { currentVan } = useOutletContext()

    return (
        <section className="host-van-detail-info">
            <h4>Name: <span>{currentVan.name}</span></h4>
            <h4>Category: <span>{currentVan.type}</span></h4>
            <h4>Description: <span>{currentVan.description}</span></h4>
            <h4>Visibility: <span>Public</span></h4>
        </section>
    )
}
```

## 常見問題

### Q1：context={{ currentVan }} 裡面的雙大括號是什麼意思？

* 外層 `{ }`：JSX 語法，代表裡面要放入 JavaScript 的表達式（Expression）。
* 內層 `{ }`：建立一個新的 JavaScript 物件。

其中的 `{ currentVan }` 是 ES6 物件屬性縮寫（Object Property Shorthand）：

```javascript
// 以下兩者完全等價：
context={{ currentVan: currentVan }} // 完整寫法
context={{ currentVan }}             // ES6 縮寫寫法
```

### Q2：可以把 State 的 Setter Function 傳給子路由嗎？  
可以，完全沒問題！

**`context` 可以傳遞任何 JavaScript 的資料類型，包含物件、陣列、狀態值以及函式。** 如果子路由需要修改父路由的狀態（例如編輯資料）：

```javascript
// 父路由：傳遞狀態與修改函式
<Outlet context={{ currentVan, setCurrentVan }} />

// 子路由：接收並呼叫 Setter 更新父層狀態
function EditVanDetails() {
    const { currentVan, setCurrentVan } = useOutletContext()
    
    const handleUpdate = (newName) => {
        setCurrentVan(prev => ({ ...prev, name: newName }))
    }
    return <button onClick={() => handleUpdate("New Name")}>Update</button>
}
```

### Q3：context 還能放些什麼？  
只要是該路由分枝（Route Branch）底下子頁面共享的資料都可以放，例如：

* 多重狀態與 API 狀態：`{ data, isLoading, error, refetch }`
* 事件處理函式（Event Handlers）：`{ handleDelete, handleSave }`
* 權限與頁面設定：`{ isEditable: true, theme: "dark" }`
* 陣列形式（若只需傳遞少數值）：`<Outlet context="{[count," setCount]}/>`

## 3. 對比：useOutletContext vs. useContext

| 比較維度 | React Router `useOutletContext` | React 原生 `useContext` |  
| :--- | :--- | :--- |  
| **作用範圍 (Scope)** | **限定** 於特定 Route 與其直接/間接的 `<Outlet/>` 子路由樹 | **全域/區域** 包覆於 `<Context.Provider>` 之下的所有元件樹 |  
| **設定繁瑣度** | **極低**：直接在 `<Outlet context="{...}">` 帶值即可使用 | **中等**：需先 `createContext()` 建立物件，再寫 `<Provider>` 包覆 |  
| **主要用途** | 處理「路由階層之間」的 Layout 與子分頁資料共享 | 處理「跨元件層級」的全域狀態（如 Theme、User Auth、UI 語法） |


## 小結  
`useOutletContext` 就是 React Router 幫開發者封裝好的「路由專用版 Context」，大幅簡化了巢狀路由間的資料傳遞流程。

<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>