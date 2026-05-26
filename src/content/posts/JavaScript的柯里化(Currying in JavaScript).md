---
title: "JavaScript的柯里化(Currying in JavaScript)"
pubDatetime: 2026-05-25T11:17:36.031Z
tags: ["JavaScript","Interview Preparation"]
description: " Table of contents <img src=\"https://images.unsplash.com/pho..."
---

## Table of contents

<img src="https://images.unsplash.com/photo-1631452180539-96aca7d48617?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=2070&q=80">

<div style="text-align:center;font-size: 10px">
(Photo by <a href="https://unsplash.com/@mekalluakella?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Kalyani Akella</a> on <a href="https://unsplash.com/photos/gml9g1kRQcM?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>)
</div>
<br/>

> 此Curry非彼Curry，不過Naan沾咖哩真的好好吃啊:yum:

## :memo: 前言
柯里化(Currying)是functional programming的一種技術 ，透過currying可以編寫模組化、易於測試和高度可重複使用的程式碼。
Functional programming(FP)是一種宣告式規範( declarative paradigm)，強調不變性(immutability)和純函式(Pure Functions)—代表該函式對於任何給定的輸入(input)，永遠會回傳相同的輸出(output)。這些特性可以使程式碼更易讀並且更容易維護，而Currying只是其中的一種技術。
:::info
更多關於Functional Programming，可以閱讀以下文章:
1. [為什麼要學 Functional Programming?](https://ithelp.ithome.com.tw/articles/10233399)
2. [JavaScript: Functional Programming 函式編程概念](https://totoroliu.medium.com/javascript-functional-programming-%E5%87%BD%E5%BC%8F%E7%B7%A8%E7%A8%8B%E6%A6%82%E5%BF%B5-e8f4e778fc08)
3. [Buzz Word 1 : Declarative vs. Imperative](https://ithelp.ithome.com.tw/articles/10233761)
:::

## :memo: 柯里化(Currying)
### 原理
Currying可以將**具有多個參數的函式轉換為一系列巢狀(nesting)函式**。也就是說，函式並非一次接受所有參數，而是接受第一個參數並返回一個新函式，此新函式接受第二個參數並返回另一個新函式，此新函式再接受第三個參數，依此類推，直到所有參數都被執行完畢。

### 範例
舉例來說，以下函式尚未進行currying：
```javascript
function multiply(a,b) {
    return a * b;
}
```
如果只傳入一個參數`a`，此函式仍會執行，而參數`b`則會以`undefined`去執行，也就是說如果執行`multiply(5)`，實際上執行的是`5 * undefined`，回傳`NaN`。
現在試著將函式currying，轉換成一系列巢狀函式:
```javascript
function curried_multiply(a) {
    return function nested(b) {
        return a * b;
   }
}
```
調用`curried_multiply()`函式:
1. 調用`curried_multiply()`函式，這個函式接受一個參數傳入，並且回傳另一個函式`nested()`。也就是**調用`curried_multiply(a)`，回傳的結果是`nested(b)`。**
2. 調用`nested()`函式，這個函式使用調用`curried_multiply()`以及`nested()`函式得到的參數`a`、`b`，並執行與回傳`a * b`的結果。**調用`nested(b)`，回傳的結果是`a * b`。**

如果進一步拆解這個函式:
```javascript
const one = curried_multiply(5)
console.log(one)
// output: [Function: nested]
```
回傳的值是`nested()`函式，接著呼叫`one()`函式，實際上是執行`nested()`函式:
```javascript
one(10)
// output: 50
```
回傳的是 `a * b`，也就是 `5 * 10`的值。

### 閉包(closure)與柯里化(currying)
呼叫`curried_multiply()`函式時調用的參數可用於巢狀函式，這是**閉包(closure)的特性**。函式中回傳函式，通常就是閉包。 當呼叫父函式時，會產生一個新的執行環境(context)，這個環境會保留所有區域變數(local variables)，這些區域變數可以透過和全域變數連結、或是從父函式的閉包，在全域環境中被取用。例如以下函式:
```javascript
function foo(x) {
  function bar(y) {
    console.log(x + y);
  }
  bar(2);
}

foo(2)
// output: 4
```
`x`被綁定在外部函式`foo()`中，當執行內部函式`bar()`時，`bar()`可以取用`x`，因為`bar()`是在`foo()`的作用域中建立的，父函式`foo()`執行完後，變數`x`被儲存於閉包中，根據JS的[Garbage Collection機制](https://www.geeksforgeeks.org/relation-of-garbage-collector-and-closure-in-javascript/)，執行`bar()`時找到其中有參照變數`a`，因此`a`不會被清除掉。另一方面，`bar()`可以取用其父函式及全域的變數，但是如果`bar()`中宣告其它函式、或`foo()`中其他函式的作用域(也就是平行於`bar()`函式)的變數，則`bar()`不可取用這些函式的變數。

:::info
Q: 函式中的函式一定代表閉包的存在?
A: 不一定。如果內部的函式(子函式)並沒有參照該函式作用域外(如父函式作用域)的變數，則閉包不存在。例如:
```javascript
function foo(x) {
  return function () {
      return true
  }
}

foo(5)() 
//output: true
```
以上例子中，不管傳入任何值到`foo()`，回傳的永遠是true，因為內部的函式並沒有參照到`foo()`作用域的變數`x`，這種狀況下閉包不存在。
:::

巢狀函式根據定義函式的位置，保留父函式的作用域；亦即，內層函式的區塊也可以取用外層函式的變數，例如前述[範例](###範例)中，`function nested(b) {return a * b}`，`nested()`函式可以取用父函式、也就是`curried_multiply()`函式的參數`a`，當我們執行`const one = curried_multiply(5)`時，`one()`保留了`curried_multiply()`的作用域，因此可以取用該作用域的變數`5`；也可以理解成在此巢狀函式中，第一個傳入的參數`a`，會成為閉包中的變數被記憶/儲存，並傳入巢狀函式鏈中的下一個函式執行。
### 箭頭函式
使用ES6的箭頭函式語法，重新撰寫前述[範例](###範例)中的`curried_multiply()`函式:
`let curried_multiply = a => b => a * b`

拆解一下上面這行程式碼:
1. 把最外層的箭頭函式`a => ...`賦值給變數`curried_multiply` 
(i.e. `curried_multiply`是一個接受`a`作為參數傳入的函式 )
3. 呼叫`curried_multiply`，傳入參數`a`，回傳另一個箭頭函式`b => a * b`(i.e. `curried_multiply`函式接受`a`作為參數傳入後，回傳的函式接受`b`作為參數傳入 )
4. 調用箭頭函式`b => a * b`，回傳相乘的結果(`a * b`) 

試著將參數傳入這個函式:
```javascript
let newNum = curried_multiply(2)(8)
console.log(newNum)
// output: 16
```
### 進階Currying範例

```javascript
const curry =(fn) =>{
    return curried = (...args) => {
        if (fn.length !== args.length){
            return curried.bind(null, ...args)
        }
    return fn(...args)
    }
}
```
以上code做了什麼?
1. `curry`作為外層函式，接受`fn`函式作為參數傳入，並回傳另一個函式`curried`
2. `curried`接受另一參數`args`傳入，並用[其餘參數（rest parameters）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters)將參數集合成陣列，並且比較`fn`和`args`的長度
:::warning
:bulb: 函式的長度(function length)是function的一種屬性(property)，**表示該 function 預期被傳入的參數數量**，這個數量並不包含其餘參數(rest parameter)且只包含第一個預設參數(Default Parameters)前的參數。
*Ref: [MDN doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/length)*
:::
3. `if`判斷式中的邏輯: 若`fn`和`args`的長度(也就是參數的數量)不同，則呼叫[`bind()`](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)方法，建立一個新的函式並傳入參數`...args`；若`fn`和`args`的長度(也就是預期傳入的參數數量)相同，則回傳傳入`args`參數的`fn`
:::warning
:bulb: `bind()`是函式的一種方法(method)，它的基本語法如下:
`fun.bind(thisArg[, arg1[, arg2[, ...]]])`
第一個參數是`this`要指向的物件，第二個與其後的參數則是要傳入該函式的參數。`bind()`會建立一個新的函式，必須調用該函式才會執行。
:::
實際應用這個函式:
```javascript
const totalNum=(x,y,z) => {
    return x+y+z 
} 
const curriedTotal = curry(totalNum)
console.log(curriedTotal(10)(20)(30))
// output: 60
```
讓我們更進一步的觀察這個函式做了什麼，在`curry`函式中加入一些code:
```javascript
const curry =(fn) =>{
    return curried = (...args) => {
          // 把args和fn相關資訊印出來
          console.log(args)
          console.log(fn.length)
          console.log(args.length)
        if (fn.length !== args.length){
            return curried.bind(null, ...args)
        }
    return fn(...args)
    }
}
```
再執行一次剛剛的`curriedTotal(10)(20)(30)`函式，在console上印出:
```javascript
// first round
[ 10 ]
3
1
// second round
[ 10, 20 ]
3
2
// third round
[ 10, 20, 30 ]
3
3
// sum output
60
```
印出的順序依序是<span style="background-color: tan">`args`</span>、<span style="background-color: lightcoral">`fn.length`</span>、<span style="background-color: gold">`args.length`</span>，因此console印出的東西可以分成三組:
1. 傳入`curry`函式的參數是`totalNum`函式，這個函式有三個參數，因此<span style="background-color: lightcoral">`fn.length`</span>是3
2. 第一次傳入一個參數，使用其餘參數轉換成陣列 `[10]`，也就是
<span style="background-color: tan">`args`</span>，此時的<span style="background-color: gold">`args.length`</span>是1，和<span style="background-color: lightcoral">`fn.length`</span>不同，因此執行`curried.bind(null, ...args)`，`curried`的參數數量變成1
3. 第二次也是傳入一個參數20，和剛剛的參數10使用其餘參數轉換成陣列 `[10, 20]`，也就是
<span style="background-color: tan">`args`</span>，此時的<span style="background-color: gold">`args.length`</span>是2，和<span style="background-color: lightcoral">`fn.length`</span>不同，因此執行`curried.bind(null, ...args)`，`curried`的參數數量變成2
4. 第三次傳入參數30，此時的<span style="background-color: tan">`args`</span>是 `[10, 20, 30]`，<span style="background-color: gold">`args.length`</span>是3，和<span style="background-color: lightcoral">`fn.length`</span>相同，因此執行`fn(...args)`，也就是把`[10, 20, 30]`使用[展開運算子(spread operator)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax)傳入`totalNum`函式，加總後得到結果60。

假如不使用currying的函式，一次傳入三個參數`curriedTotal(10, 20, 30)`，則會在console上印出:
```javascript
[ 10, 20, 30 ]
3
3
60
```
因為傳入的參數和`totalNum`參數數量相同，因此直接執行`fn(...args)`。

以上範例的`curry`函式也可以用[`apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)方法執行，結果是一樣的:
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(null, args);
    } else {
      return function(...args2) {
        return curried.apply(null, args.concat(args2));
      }
    }
  }
}
```
與`bind()`方法的不同之處在於，`apply()`方法接受的第二個參數是一個**陣列**，因此無須再展開。
:::warning
:bulb: [`apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)方法的基本語法:
`fun.apply(thisArg, [argsArray])`
第一個參數一樣是`this`指向的對象，第二個參數則是參數陣列或是array-like 物件；和`bind()`方法不一樣，`apply()`是直接執行該函式，而非建立新函式(或是說拷貝原函式物件)。
:::
:::info
:bookmark: 更多關於`apply()`、`bind()`、`call()`方法，可以閱讀以下文章:

- [使用 bind、call、apply 改變 this 指向的對象](https://b-l-u-e-b-e-r-r-y.github.io/post/BindCallApply/)
- [JavaScript 中 call()、apply()、bind() 的用法](https://www.runoob.com/w3cnote/js-call-apply-bind.html)
- [[JavaScript] 函數原型最實用的 3 個方法 — call、apply、bind](https://realdennis.medium.com/javascript-%E8%81%8A%E8%81%8Acall-apply-bind%E7%9A%84%E5%B7%AE%E7%95%B0%E8%88%87%E7%9B%B8%E4%BC%BC%E4%B9%8B%E8%99%95-2f82a4b4dd66)
:::
## :memo: Currying的優點
Currying可以**使函式具有單一用途，使程式碼更加模組化，從而更易於測試、debug、維護和閱讀。** 以下直接用範例說明:


假如我們有以下的資料，想要進行篩選與排序:
```javascript
const candidate = [
    { age: 20, location: "CA", skills:"JavaScript", dateApplied: new Date('2021-01-20') },
    { age: 26, location: "TX", skills:"Python",  dateApplied: new Date('2022-11-09') },
    { age: 45, location: "OR", skills:"PHP", dateApplied: new Date('2021-03-06') },
    { age: 26, location: "NJ", skills:"JavaScript",  dateApplied: new Date('2019-12-30') },
    { age: 33, location: "LA", skills:"Python", dateApplied: new Date('2020-05-10') },
    { age: 18, location: "CA", skills:"JavaScript",   dateApplied: new Date('2018-07-17') },
    { age: 31, location: "NJ", skills:"Java",   dateApplied: new Date('2022-05-18') }
]
```
針對`candidate`資料**根據地點進行篩選並排序**，因此寫了以下的函式:
```javascript
const sortByValueFromLocation = (candidateArr, location, sortKey) => {
    return candidateArr.filter(candidate => {
        return candidate.location === location
    }).sort((a,b) => {
        return a[sortKey] - b[sortKey]
    })
}
 
console.log(sortByValueFromLocation(candidate, "CA", "age"))
```
假設今天需要另一個兩個篩選條件的函式來處理這筆資料，於是我們又寫了另一個函式:
```javascript
const filterByValueFromLocation = (candidateArr, location, filterKey, filterValue) => {
 return candidateArr.filter(candidate => {
   return candidate.location === location
 }).filter(candidateFromCity => candidateFromCity[filterKey] === filterValue)
}
 
console.log(filterByValueFromLocation(candidate, "CA", "skills", "JavaScript"));
```
以上兩個函式可以觀察到:
1. 這兩個函式都根據地點篩選，並且每次都執行相同的篩選方式: 寫了許多重複的code
2. 這兩個函式都接受多個參數: 倘若發生錯誤，需要花比較多時間確認是哪個參數造成的錯誤

試著用Currying改寫以上函式，讓code變得更模組化、容易閱讀、可以重複使用:
```javascript
const setFilter = array => key => value => array.filter(x => x[key] === value);
const filterCandidates = setFilter(candidate);
const filterCandidatesByLocation = filterCandidates('location');
const filterCandidatesByCA = filterCandidatesByLocation('CA');
const filterCandidatesBySkills = filterCandidates('skills');
const filterCandidatesByJavaScript = filterCandidatesBySkills('JavaScript');
 
console.log(filterCandidatesByCA) 
// 回傳地點是CA的人選 
console.log(filterCandidatesByJavaScript); 
// 回傳技能是JavaScript的人選 
```
code模組化後，可以重用於其他功能，例如可以將上述技能是JavaScript的人選排序:
```javascript
const sortArrayByValue = sortArray => sortKey => {
    return sortArray.sort(function(a, b){
        if(a[sortKey] < b[sortKey]) { return -1; }
        if(a[sortKey] > b[sortKey]) { return 1; }
        return 0;
    });
}
 
const sortJS = sortArrayByValue(filterCandidatesByJavaScript);
const sortJSByDate = sortJS("dateApplied");
console.log(sortJSByDate)
/* output: 
[
  {
    age: 18,
    location: 'CA',
    skills: 'JavaScript',
    dateApplied: 2018-07-17T00:00:00.000Z
  },
  {
    age: 26,
    location: 'NJ',
    skills: 'JavaScript',
    dateApplied: 2019-12-30T00:00:00.000Z
  },
  {
    age: 20,
    location: 'CA',
    skills: 'JavaScript',
    dateApplied: 2021-01-20T00:00:00.000Z
  }
]
*/
```
上面的程式碼中，`filterCandidatesByJavaScript`作為`sortArrayByValue`的`sortArray`參數傳入，並回傳`sortKey => ...` 這個函式，接著再傳入`"dateApplied"`作為`sortKey`參數執行`sortArray`.[`sort()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)函式，最後回傳排序結果。

## :memo: Partial application vs. Currying

### 什麼是Partial Application

所謂的application，是指將函式應用(apply)在其參數(argument)上，以產出一個回傳值的過程；所以，partial application就是將該函式應用於其**部分參數**的過程，而該函式回傳留作它用。換句話說，partial application就是**一個函式接受多個參數、並回傳一個含有較原先參數數量少的函式的過程**。**和currying一樣，都會應用到閉包的概念，差別在於currying一次只傳入一個參數。** partial application將一或多個參數值固定在回傳的函式中，該回傳函式接受剩餘的參數，以便能完整執行該函式。聽起來好像很奇怪，不過這樣的做法可以減少參數的數量。請看以下範例：
```javascript
const promotionDetails = (product, discount, startDate) => {
  return `The ${product} price is ${discount}% off from ${startDate} `
}

const fruitPromotion = startDate => {
  return promotionDetails('fruits', 20, startDate)
}

const dairyPromotion = startDate => {
  return promotionDetails('dairy products', 20, startDate)
}
```
假設今天要做一個商品打折的活動，我們先設定了一個通用的`promotionDetails`函式，需要傳入三個參數`product`, `discount`, `startDate`，接著當我們需要分別對各種商品進行處理，又寫了幾個不同的函式，只是...怎麼每個看起來都好像，好多重複的code!試著改寫一下:
```javascript
const promotionDetails = (discount, product, startDate) => {
  return console.log(`The ${product} price is ${discount}% off from ${startDate} `)
}

const twentyOffPromotion = (product, startDate) => {
  return promotionDetails(20, product, startDate)
}

const fruitPromotion = startDate => {
  return twentyOffPromotion('fruits', startDate)
}

const dairyPromotion = startDate => {
  return twentyOffPromotion('dairy products', startDate)
}
```
看起來是不是比較簡潔?把共通的部分抽取出來，也就是折扣20%，另外寫成函式`twentyOffPromotion`，並且把它的參數`discount`固定為20這個值，便可以把這個函式應用在其他的函式中，`fruitPromotion`和`dairyPromotion`都只需要接收參數`startDate`即可。這就是partial application。

### 高階函式 (High Order Function)
高階函式意指一個函式可以接收另一函式作為參數、或者回傳一個函式。
:::info
Ref: [高階函式 (Higher Order Function) 是什麼？](https://www.explainthis.io/zh-hant/swe/what-is-hof)
:::
上述的範例如果應用高階函式改寫:
```javascript
const partial = (fn, ...argsToApply) => {
  return (...restArgsToApply) => {
    return fn(...argsToApply, ...restArgsToApply)
  }
}

const promotionDetails = ( discount, product, startDate) => {
  return console.log(`The ${product} price is ${discount}% off from ${startDate} `)
}

const twentyOffPromotion = partial(promotionDetails, 20)

const fruitPromotion = partial(twentyOffPromotion, 'fruits', '2020/02/20')
fruitPromotion() // The fruits price is 20% off from 2020/02/20
const dairyPromotion = partial(twentyOffPromotion, 'dairy products', '2020/01/23')
dauryPromotion() // The dairy products price is 20% off from 2020/01/23 
```
`partial`函式做了什麼?
1. 傳入兩個參數`fn`, `argsToApply`:`fn`是要被partially applied的函式，`argsToApply`則是收集傳入的參數，利用其餘參數轉換為陣列
2. 執行`partial`函式會回傳另一個函式，這個函式接收除了 `argsToApply`以外的參數，這裡命名為 `restArgsToApply`
3. 執行這個回傳的函式會再回傳另一個函式，這個函式接收兩個參數`argsToApply`及 `restArgsToApply`；這裡跟currying一樣，應用到閉包的概念，執行外層`partial`函式的參數`argsToApply`和`fn`儲存於閉包中，內部的函式因此可以取得外部函式(父函式)的參數。所以最內層回傳的函式，其實是執行最外層傳入的參數`fn`，也就是我們要進行partially applied的函式。


---

## 參考資料
* MDN文件
* [Closures & Currying in JavaScript](https://engineering.cerner.com/blog/closures-and-currying-in-javascript/)
* [[JS] Functional Programming and Currying](https://pjchender.dev/javascript/js-functional-programming-currying/#%E4%BB%80%E9%BA%BC%E6%98%AF-currying)
* [Understanding JavaScript currying](https://blog.logrocket.com/understanding-javascript-currying/)
* [Day 22 ：什麼是 Currying（4）？自己動手寫一個 Curry 吧！](https://ithelp.ithome.com.tw/articles/10297474)
* [進階 Javasctipt 概念 (4)](https://medium.com/@paulyang1234/%E9%80%B2%E9%9A%8E-javasctipt-%E6%A6%82%E5%BF%B5-4-703e69416118)
* [Closure 閉包](https://eyesofkids.gitbooks.io/javascript-start-from-es6/content/part4/closure.html)
* [JavaScript 深入淺出 Garbage Collection 垃圾回收機制](https://shawnlin0201.github.io/JavaScript/JavaScript-Garbage-Collection/)
* [Day02_JS的二三事](https://ithelp.ithome.com.tw/articles/10192743)
* [Beginners Guide To Higher Order Functions, Partial Functions and Currying](https://dev.to/ubahthebuilder/beginners-guide-to-higher-order-functions-partial-functions-and-currying-2dm5#:~:text=A%20higher%20order%20function%20is%20a%20function%20which%20returns%20another,until%20it%20is%20fully%20resolved.)
* [Functional JS #5: Partial Application, Currying](https://medium.com/dailyjs/functional-js-5-partial-application-currying-da30da4e0cc3)
* [Good Morning, Functional JS (Day 10, Partial Application 偏函數應用)](https://ithelp.ithome.com.tw/articles/10194837)
* Codecademy 教材

::: success
:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！
:::