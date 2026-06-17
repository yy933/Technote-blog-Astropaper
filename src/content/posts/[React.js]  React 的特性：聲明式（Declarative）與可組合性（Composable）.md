---
title: "[React.js]  React 的特性：聲明式（Declarative）與可組合性（Composable）"
pubDatetime: 2026-06-11T02:37:30.322Z
tags: ["React.js","Frontend","Interview Preparation"]
description: "Table of contents :memo: 什麼是聲明式（Declarative）與可組合性（Composabl..."
hackmd_id: "Bk4GjqPbzg"
---

## Table of contents

## :memo: 什麼是聲明式（Declarative）與可組合性（Composable）？  
React 之所以能徹底改變前端開發的生態，很大程度是因為它將 **聲明式（Declarative）** 與 **可組合性（Composable）** 發揮到了極致：
* **聲明式（Declarative）**：關注於「結果長怎樣（What）」，而非「如何一步步操作 DOM（How）」。
* **可組合性（Composable）**：將複雜的 UI 拆解成獨立、高複用性的小積木（Component），再層層堆疊拼裝。


## :memo: Declarative 與 Composable 的優勢  
在傳統的命令式(Imperative)開發（如原生 JS 或 jQuery）中，隨著專案規模變大，UI 的維護成本會呈指數級成長。React 的這兩大特性帶來了以下核心優勢：
* **大幅減少 Bug**：開發者只需專注在資料狀態（State）的邏輯，不需要痛苦地手動操作網頁 DOM，避免狀態與 UI 不同步的問題。
* **極高的程式碼複用性**：組件化設計讓相同的程式碼可以「寫一次，到處用」，提升開發效率。
* **低耦合、好維護**：不論是出錯排查（Debug）還是未來功能擴充，都只需要針對單一組件處理，不會牽一髮而動全身。

## :memo: 用法(範例)

### 用法一：聲明式（Declarative）的開發思維  
傳統的 **命令式 (Imperative)** 需要手動操控每一個 DOM 的更新步驟；而 React 的 **聲明式** 寫法，只需要定義好狀態與 UI 的映射關係，React 就會自動處理底層的更新細節。

**對比範例：做一個「點擊按鈕，數字加一，且大於等於 10 時文字變紅」的功能**

* **傳統命令式寫法 (jQuery/JS)：**

```javascript
    let count = 0;
    $('#btn').click(function() {
      count++;
      $('#counter-text').text(`目前點擊了 ${count} 次`);
      if (count >= 10) {
        $('#counter-text').css('color', 'red');
      }
    });
```

* **React 聲明式寫法：**

    ```tsx=
    import { useState } from 'react';

    export default function Counter() {
      const [count, setCount] = useState(0);

      return (
        <div>
          <p style={{ color: count >= 10 ? 'red' : 'black' }}>
            目前點擊了 {count} 次
          </p>
          <button onClick={() => setCount(count + 1)}>點擊加一</button>
        </div>
      );
    }
    ```
<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

**觀念解析：** 在 React 中，我們不再使用 `appendChild` 或 `textContent`。我們只告訴 React：當 `count` 改變時，網頁畫面的結構「應該長怎樣」，其餘的由虛擬 DOM（Virtual DOM）代勞。

</blockquote>


### 元件可組合性（Component Composition）  
在 React 中「萬物皆為元件」。你可以將一個複雜的大介面，拆解成許多獨立、好管理的小元件，再透過巢狀結構拼裝在一起。

```tsx
// 1. 定義獨立的小積木
function Avatar() { 
  return <img src="user.png" alt="user_avatar" />; 
}

function UserInfo() { 
  return <h2>王小明</h2>; 
}

// 2. 把小積木組合在一起，構建出大元件
export default function UserProfile() {
  return (
    <div className="card">
      <Avatar />
      <UserInfo />
    </div>
  );
}
```

### 進階組合技巧（利用 props.children）  
除了簡單的結構嵌套，React 還允許我們利用 `props.children` 製作高彈性的「外殼組件」（例如：通用彈出視窗、卡片容器），裡面要塞入什麼內容完全由母組件自由決定。

```jsx
// 定義一個通用的卡片外殼
function CardWrapper({ children }) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '20px', borderRadius: '8px' }}>
      {children} {/* 這裡會渲染所有包在裡面的子組件 */}
    </div>
  );
}

// 在不同場景下自由組合不同內容
export default function App() {
  return (
    <div>
      {/* 場景 A：文字卡片 */}
      <CardWrapper>
        <h3>公告通知</h3>
        <p>今天下午兩點系統維護。</p>
      </CardWrapper>

      {/* 場景 B：圖文卡片 */}
      <CardWrapper>
        <img src="banner.jpg" alt="banner" />
        <button>立即購買</button>
      </CardWrapper>
    </div>
  );
}
```
<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

詳細說明與進階組件設計模式，可參考[React 官方文件](https://react.dev/learn/passing-props-to-a-component)

</blockquote>

## 注意事項 & 限制

| 項目 | 說明 |  
| :--- | :--- |  
| **資料流方向（單向資料流）** | 組件組合時，資料必須是由上而下（Parent to Child）透過 `props` 傳遞，不可逆向。 |  
| **避免過度拆分（Over-engineering）** | 雖然可組合性很棒，但不必將每一行 HTML 都拆成組件，當組件有「可複用性」或「邏輯複雜度」時再拆分即可。 |  
| **狀態提升（Lifting State Up）** | 當多個組合組件需要共用同一個狀態時，必須將該狀態宣告在它們共同的「最近公共父組件」中。 |  
| **副作用（Side Effects）的控制** | 聲明式專注於純粹的 UI 映射。若需要處理 API 請求或手動操作外部 DOM，必須使用 `useEffect` 等 Hook 來安全處理。 |


## 小結  
這兩個特性加在一起，就構成了 React 開發的精髓：

$$\text{UI} = f(\text{state})$$

透過 **聲明式** 的語法，我們能用最直觀、與純函數 $f$ 相似的方式，寫出一個個職責單一的「小積木」；再透過 **可組合性**，我們能把這些小積木層層堆疊組合，最終輕鬆構建出一個龐大、穩固且易於維護的現代化web app。
<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>