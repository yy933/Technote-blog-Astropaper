---
title: "[React] 表單（form）細節回顧 - 筆記"
pubDatetime: 2026-05-26T03:29:26.541Z
tags: ["JavaScript","React.js","HTML"]
description: " Table of contents 本文雖然不完全屬於React的範圍，但在React中操作表單(form)相當常見，..."
---

## Table of contents

本文雖然不完全屬於React的範圍，但在React中操作表單(form)相當常見，因此先來回顧一些關於表單的細節。

示範表單如下：
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

function App() {
  return (
    <section>
      <h1>Signup form</h1>
      <form>
        <label htmlFor="email">Email:</label>
        <input id="email" type="email" name="email" placeholder="joe@schmoe.com" />
        <br />
        
        <label htmlFor="password">Password:</label>
        <input id="password" type="password" name="password" />
        <br />
          
        <button>Submit</button>
          
      </form>
    </section>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```
## `htmlFor`、`id`屬性
```jsx
<label htmlFor="email">Email:</label>
<input id="email" type="email" name="email" placeholder="joe@schmoe.com" />
```
`htmlFor`相當於HTML中的`for`屬性，必須和`input`元素中的`id`給定同一個值，才能在點選該`label`的時候focus到該輸入欄位。

## 提交表單 Form Submission
```jsx
function App() {
  
  function handleSubmit(event) {
    event.preventDefault() // 防止瀏覽器預設行為(refresh)
    const formEl = event.currentTarget // 這裡的target就是表單
    const formData = new FormData(formEl) // 取得表單輸入內容
    const email = formData.get("email") // 利用name屬性取得該欄位資料
    // ... 把表單資料傳到後端
    formEl.reset()  // 清空、重置表單
  }
  
  return (
    <section>
      <h1>Signup form</h1>
      <form action="phpfile.php" onSubmit={handleSubmit} method="post"> // onSubmit管理表單提交行為
        <label htmlFor="email">Email:</label>
        <input id="email" type="email" name="email" placeholder="joe@schmoe.com" />
        <br />
        
        <label htmlFor="password">Password:</label>
        <input id="password" type="password" name="password" />
        <br />
        
        <button>Submit</button>
        
      </form>
    </section>
  )
}
```

## Form action in React 19
在[React 19](https://react.dev/reference/react-dom/components/form)中，可以更簡單地透過 `<form>` 搭配 action function（類似 server action）來處理表單提交。
* `onSubmit` 可以是 async function。
* 支援自動處理 FormData。
* 可搭配 server action 使用（如：`action={submitForm}`）。

上述的例子可以改寫成以下：
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

function App() {
  function signUp(formData) {   // 更直觀的函式命名
    const email = formData.get("email")
    // ... 把表單資料傳到後端
  }
  
  return (
    <section>
      <h1>Signup form</h1>
      <form action={signUp}> // 在action屬性中傳入signUp
        <label htmlFor="email">Email:</label>
        <input id="email" type="email" name="email" placeholder="joe@schmoe.com" />
        <br />
        
        <label htmlFor="password">Password:</label>
        <input id="password" type="password" name="password" />
        <br />
        
        <button>Submit</button>
        
      </form>
    </section>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```
防止瀏覽器預設行為、取得表單資料、重置表單等都由React在幕後完成了，不需要再寫出來，程式碼變得更乾淨了!

## `textarea` & `defaultValue`
```jsx
<form action={signUp}>

        <label htmlFor="email">Email:</label>
        <input id="email" defaultValue="joe@schmoe.com" type="email" name="email" placeholder="joe@schmoe.com" />

        <label htmlFor="password">Password:</label>
        <input id="password" defaultValue="password123" type="password" name="password" />

        <label htmlFor="description">Description:</label>
        <textarea id="description" name="description"></textarea>

        <button>Submit</button>

      </form>
```

## input type="radio"
```jsx
<form action={signUp}>

        <fieldset>
          <legend>Employment Status:</legend>
          <label>
            <input type="radio" name="employmentStatus" value="unemployed" />
            Unemployed
        </label>
          <label>
            <input type="radio" name="employmentStatus" value="part-time" />
            Part-time
        </label>
          <label>
            <input type="radio" name="employmentStatus" defaultChecked={true} value="full-time" />
            Full-time
        </label>
        </fieldset>
        <button>Submit</button>

      </form>
```
:bulb: 重點：
* 同一個項目中不同選項給定相同的`name`
* 可以用`<fieldset>`把所有選項包起來(更語意化)
* `<legend>`用來標示項目名稱
* radio input必須給定`value`才能正確取得使用者選擇的選項
* 預設選項可以用`defaultChecked={true}`指定
    
## input type="checkbox"
```jsx
function App() {
    function signUp(formData) {
    const dietaryRestrictions = formData.getAll("dietaryRestrictions") // getAll獲得使用者選擇的所有選項的陣列
  }

  return (
    <section>
      <h1>Signup form</h1>
      <form action={signUp}>

        <fieldset>
          <legend>Dietary restrictions:</legend>
          <label>
            <input type="checkbox" name="dietaryRestrictions" value="kosher" />
            Kosher
        </label>
          <label>
            <input type="checkbox" name="dietaryRestrictions" value="vegan" />
            Vegan
        </label>
          <label>
            <input type="checkbox" name="dietaryRestrictions" defaultChecked={true} value="gluten-free" />
            Gluten-free
        </label>
        </fieldset>

        <button>Submit</button>

      </form>
    </section>
  )
 }    
}
```
和radio類似，不同的是可選擇多個選項，要得到使用者選擇的所有選項，使用：
`formData.getAll("dietaryRestrictions") // 回傳使用者選擇的所有選項的陣列 `

## select & option
```jsx
function App() {
    function signUp(formData) {
    const favColor = formData.get("favColor")
  }

  return (
    <section>
      <h1>Signup form</h1>
      <form action={signUp}>

        <label htmlFor="favColor">What is your favorite color?</label>
        <select id="favColor" name="favColor" defaultValue="" required>
          <option value="" disabled>-- Choose a color --</option>
          <option value="red">Red</option>
          <option value="orange">Orange</option>
          <option value="yellow">Yellow</option>
          <option value="green">Green</option>
          <option value="blue">Blue</option>
          <option value="indigo">Indigo</option>
          <option value="violet">Violet</option>
        </select>

        <button>Submit</button>

      </form>
    </section>
  )
 }    
}
```

## 一次取得所有表單資料：Object.fromEntries(formData)

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';

function App() {
  function signUp(formData) {
    const data = Object.fromEntries(formData)  // 取得所有輸入的內容
    const dietaryRestrictions = formData.getAll("dietaryRestrictions")
    const allData = {
      ...data,
      dietaryRestrictions  // 使用getAll獲得的資料仍維持陣列格式
    }
    console.log(allData)
  }

  return (
    <section>
      <h1>Signup form</h1>
      <form action={signUp}>

        <label htmlFor="email">Email:</label>
        <input id="email" defaultValue="joe@schmoe.com" type="email" name="email" placeholder="joe@schmoe.com" />

        <label htmlFor="password">Password:</label>
        <input id="password" defaultValue="password123" type="password" name="password" />

        <label htmlFor="description">Description:</label>
        <textarea id="description" name="description" defaultValue="This is a description"></textarea>

        <fieldset>
          <legend>Employment Status:</legend>
          <label>
            <input type="radio" name="employmentStatus" value="unemployed" />
            Unemployed
        </label>
          <label>
            <input type="radio" name="employmentStatus" value="part-time" />
            Part-time
        </label>
          <label>
            <input type="radio" name="employmentStatus" defaultChecked={true} value="full-time" />
            Full-time
        </label>
        </fieldset>

        <fieldset>
          <legend>Dietary restrictions:</legend>
          <label>
            <input type="checkbox" name="dietaryRestrictions" value="kosher" />
            Kosher
        </label>
          <label>
            <input type="checkbox" name="dietaryRestrictions" value="vegan" />
            Vegan
        </label>
          <label>
            <input type="checkbox" name="dietaryRestrictions" defaultChecked={true} value="gluten-free" />
            Gluten-free
        </label>
        </fieldset>

        <label htmlFor="favColor">What is your favorite color?</label>
        <select id="favColor" name="favColor" defaultValue="blue" required>
          <option value="" disabled>-- Choose a color --</option>
          <option value="red">Red</option>
          <option value="orange">Orange</option>
          <option value="yellow">Yellow</option>
          <option value="green">Green</option>
          <option value="blue">Blue</option>
          <option value="indigo">Indigo</option>
          <option value="violet">Violet</option>
        </select>

        <button>Submit</button>

      </form>
    </section>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```
:bulb: `Object.fromEntries`取得的資料是字串，若要讓使用`getAll`獲得的資料仍維持陣列格式，可存到另外的物件中，把其餘資料放入後，再把`getAll`獲得的資料覆寫進去：
```jsx
function signUp(formData) {
    const data = Object.fromEntries(formData)  // 取得所有輸入的內容
    const dietaryRestrictions = formData.getAll("dietaryRestrictions")
    const allData = {
      ...data,
      dietaryRestrictions  // 使用getAll獲得的資料仍維持陣列格式
    }
    console.log(allData)　// output: {email: 'joe@schmoe.com', password: 'password123', description: 'This is a description', employmentStatus: 'full-time', dietaryRestrictions: ['kosher', 'vegan', 'gluten-free'], favColor: 'blue'}
  }
```