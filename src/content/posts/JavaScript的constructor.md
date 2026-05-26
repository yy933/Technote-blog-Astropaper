---
title: "JavaScript的constructor"
pubDatetime: 2026-05-26T03:29:26.410Z
tags: ["JavaScript","Interview Preparation"]
description: " Table of contents :memo: 什麼是建構式(Constructor) 建構式(Constructo..."
---

## Table of contents

## :memo: 什麼是建構式(Constructor)
建構式(Constructor)是一種在創造物件時，執行初始化的函式。使用`new`關鍵字時，參數(arguments)將被傳入Constructor函式。以下分別介紹傳統的constructor function，以及ES6引入的class中的`constructor`方法。
### 物件建構函式 (The Object Constructor Function)
要使用constructor創造物件，只需定義一個帶有任意數量參數(arguments)的 JavaScript 函式即可。在函式內部，關鍵字`this`指的是正在創造的物件，因此我們可以用`this.property`來寫入該物件的屬性。**constructor函式的第一個字母最好使用大寫字母，以方便溝通、及避免與一般函式混淆**。
例如：
```javascript
// Define a constructor function
function Food(name, cost, rating) {
  this.name = name;
  this.cost = cost;
  this.rating = rating;
  this.info = function(){
      console.log(`The ${this.name} costs ${this.cost} dollars and has a rating of ${this.rating} stars.`)
  }
}
const cheese = new Food('cheese', 15, 4.5)
console.log(cheese)
/* output: create a new instance
Food {
  name: 'cheese',
  cost: 15,
  rating: 4.5,
  info: [Function (anonymous)]
}
*/
cheese.info()
// output: The cheese costs 15 dollars and has a rating of 4.5 stars.
```

- 上述例子中，由constructor(也就是`Food`函式)所建立的物件稱為實例(Instance)，因此，`cheese`是一個實例。也可以用`constructor.name`的方法取得該物件的constructor function名稱，例如上述例子中：
`
console.log(cheese.constructor.name)　// output: Food
`

- `this`指向的是正在建立的物件，所以上述的constructor function如果寫成下面的程式碼，會得到一樣的結果：
```javascript
function Food(name, cost, rating) {
  this.name = name;
  this.cost = cost;
  this.rating = rating;
  this.info = function(){
      console.log(`The ${this.name} costs ${this.cost} dollars and has a rating of ${this.rating} stars.`)
  }
  return this
}
const cheese = new Food('cheese', 15, 4.5)

cheese.info()
// output: The cheese costs 15 dollars and has a rating of 4.5 stars.
```


- 如果沒有使用`new`關鍵字，則不會創造新的物件，而只是單純執行constructor function，因此`this`不會指向新物件，而是指向全域物件(global object)，也就是`window`，因此在上述的例子中，如果把程式碼改成：
```javascript
function Food(name, cost, rating) {
  this.name = name;
  this.cost = cost;
  this.rating = rating;
  this.info = function(){
      console.log(`The ${this.name} costs ${this.cost} dollars and has a rating of ${this.rating} stars.`)
  }
  
}
// 拿掉new關鍵字
const cheese = Food('cheese', 15, 4.5)
cheese.info()
// output: undefined
```
會得到`undefined`的結果以及以下error:
`TypeError: Cannot read properties of undefined (reading 'info')`
因為此時的`this`指向的是全域物件`window`，`window`中沒有`info`這個屬性(property)，因此結果是`undefined`

### 類別(Class)中的`constructor`方法 (The Class constructor Method)

Class是ECMAScript 6 中引入的模板，一樣可以用來創造物件。Class中的constructor和傳統的constructor function類似，用來建立和初始化一個類別的物件，只需定義一個帶有任意數量參數(arguments)的 JavaScript 函式即可。在函式內部，關鍵字`this`指的是正在創造的物件。和傳統建構式的差別除了**更容易閱讀、減少與一般函式混淆之外，也可以將原型(prototype)的方法直接寫在class裡面。**

例如[MDN文件](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Classes/constructor)中的例子：
```javascript
class Polygon {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
  // Getter
  get area() {
    return this.calcArea();
  }
  // Method
  calcArea() {
    return this.height * this.width;
  }
}
const square = new Polygon(10, 10);

console.log(square.area); //100
```

上述程式碼可以觀察到幾個重點：

*  `constructor`是class中一種特殊的方法(method)，用來建立和初始化一個類別的物件。一個類別只能有一個名為`constructor`的特別方法，如果一個 class 出現兩次以上的 `constructor`，就會發生 `SyntaxError`。
*  getter: 和物件實字 (Object Literals)一樣，可以用[`get`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get)語法將物件屬性和函式綁定，查找該屬性時會呼叫該函式 
* 方法(Method): 方法是定義在各類別實例的原型(prototype)上，並且所有的實例都擁有該方法，在console上觀察看看：
```javascript
// 同樣的constructor建立的instance擁有同樣的方法
const rectangle = new Polygon(20, 10)
console.log(rectangle.calcArea())
// output: 200
// 兩個instance擁有相同prototype
console.log(Object.getPrototypeOf(square) === Object.getPrototypeOf(rectangle))
// output: true
```










---

## 參考資料
* [MDN文件](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Classes/constructor)
* [W3Schools-JavaScript Object Constructors](https://www.w3schools.com/js/js_object_constructors.asp)
* [[ES6 JavaScript] 類別 (Class) 與 建構式 (Constructor)](https://zwh.zone/es6-javascript--e9-a1-9e-e5-88-a5-class--e8-88-87--e5-bb-ba-e6-a7-8b-e5-bc-8f-constructor/)
* [[筆記] 談談 JavaScript 中的 function constructor 和關鍵字 new](https://pjchender.blogspot.com/2016/06/javascriptfunction-constructornew.html)
* [鐵人賽：JavaScript 建構式](https://www.casper.tw/javascript/2017/12/18/javascript-constructor/)
* [建構物件範本：Constructor Function](https://javascript.alphacamp.co/constructor-function.html)
* [[JavaScript] new的用法與基本概念](https://dotblogs.com.tw/AceLee/2017/06/22/135553)
* [[JS] JavaScript 類別（Class）](https://pjchender.dev/javascript/js-class/#prototype-method%E5%8E%9F%E5%9E%8B%E6%96%B9%E6%B3%95)
* [鐵人賽：ES6 建構式語法糖](https://www.casper.tw/javascript/2017/12/31/javascript-constructor/)




























<blockquote class="my-6 p-4 bg-green-50 dark:bg-green-950/30 border-l-4 border-green-500 rounded-r-md text-green-900 dark:text-green-200 blocknoted-fix">

:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！

</blockquote>