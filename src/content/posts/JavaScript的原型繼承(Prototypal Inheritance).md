---
title: "JavaScript的原型繼承(Prototypal Inheritance)"
pubDatetime: 2026-05-26T03:01:46.873Z
tags: ["Interview Preparation","JavaScript"]
description: " Table of contents :memo: 前言 在[MDN文件](https://developer.mozi..."
---

## Table of contents

## :memo: 前言
在[MDN文件](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)中提到，JavaScript並非一個以class為基礎(class-based)的語言(例如Java、C++)，儘管在JavaScript中有`class`這個關鍵字，但那只是為了開發者撰寫更直觀易懂的語法糖；事實上，**JavaScript是以原型為基礎(prototype-based)的語言**。

## :memo: 繼承(Inheritance)
### 什麼是繼承
繼承(Inheritance)可以說是物件導向程式設計（object-oriented programming, OOP）最重要的原則之一，繼承可以讓子類別(child class/subclass)沿用父類別(parent class/ superclass)的屬性與功能，以[MDN文件](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object-oriented_programming)的例子而言:
```
class Professor
    properties
        name
        teaches
    constructor
        Professor(name, teaches)
    methods
        grade(paper)
        introduceSelf()
```
以上例子中，定義了一個`Professor`類別，而這個類別中有兩個屬性(properties): `name` 和 `teaches`，以及兩種方法(methods):`grade()` 和`introduceSelf()`
類別就像是一個模板，可以創造該類型的物件，每個被創造出來的物件稱為該類別的**實例(Instance)**，而創造實例的過程則是透過一種特殊的函式- **建構式(Constructor)** 達成。

至於什麼是繼承呢，再來看另一個類別`Student`:
```
class Student
    properties
        name
        year
    constructor
        Student(name, year)
    methods
        introduceSelf()
```
在`Student`這個類別中，可以發現和`Professor`類別擁有相同的屬性`name`以及相同的方法 `introduceSelf()`，因此我們可以定義一個`Person`類別作為這兩個類別的父類別，讓這兩個類別繼承`Person`的屬性或方法：
```
class Person
    properties
        name
    constructor
        Person(name)
    methods
        introduceSelf()

class Professor : extends Person
    properties
        teaches
    constructor
        Professor(name, teaches)
    methods
        grade(paper)
        introduceSelf()

class Student : extends Person
    properties
        year
    constructor
        Student(name, year)
    methods
        introduceSelf()
```
### 繼承的優點
- **提高程式碼的重複使用性**
假如我們有一個class A，並且想再建立一個包含class A部分程式碼的class B，可以透過繼承從class A衍生class B，重複使用class A的資料與方法。
- **避免程式碼重複**
繼承可以在多個子類別中共享程式碼，減少程式碼重複；如果兩個相關的類別擁有類似的程式碼，我們可以將這些程式碼放進父類別中。
- **提高程式碼靈活性及延展性**
如果需要更改，可以在父類別中更改並由子類別繼承，換言之，父類別的屬性和方法所做的更改都可以直接應用在子類別上，所有公共的屬性和方法都可以直接在父類別宣告。另一方面，子類別也可以加入新的屬性或方法。
- **提供更佳的程式碼結構與管理**
繼承使得子類別必須遵照標準的介面(interface)進行延伸，提供了方便理解的程式碼結構
- **保留父類別的完整性**
宣告子類別並不影響父類別的原始碼，因此可以保留父類別的完整性。這也是封裝性（Encapsulation）的特性展現。
- **隱藏數據**
父類別可以將某些數據設為私有，子類別無法更改或取用
- **幫助達成執行環境的多型(Polymorphism)**
透過繼承，子類別可以加入不同的執行方式(implementation)，覆寫(override)父類別的方法

*Ref: [MDN doc](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object-oriented_programming), [Inheritance in OOPS: An Idea of Code Reusability](https://www.enjoyalgorithms.com/blog/inheritance-in-java)*
### JavaScript的原型繼承
以下這兩段來自MDN文件的描述，快速說明了JS原型繼承的特性：
> **某些人認為 JavaScript 並非真正的物件導向 (Object-oriented, OO) 語言。** 在「典型 OO」中，你必須定義特定的類別物件，才能定義哪些類別所要繼承的類別。**JavaScript 則使用不同的系統 —「繼承」的物件並不會一併複製功能過來，而是透過原型鍊連接其所繼承的功能，亦即所謂的原型繼承 (Prototypal inheritance)。**
 From [MDN文件](https://developer.mozilla.org/zh-TW/docs/Learn/JavaScript/Objects/Classes_in_JavaScript)

:::warning
關於物件導向程式設計，可以參考[這篇文章](https://www.educative.io/blog/object-oriented-programming)
::: 
 
#### 原型鏈的頂端是物件
> JavaScript 就只有一個建構子：物件。每個物件都有一個連著其他原型（prototype）的私有屬性（private property）物件。**原型物件也有著自己的原型，於是原型物件就這樣鏈結，直到撞見 null 為止：null 在定義裡沒有原型、也是原型鏈（prototype chain）的最後一個鏈結。 幾乎所有 JavaScript 的物件，都是在原型鏈最頂端的物件實例。**
> From [MDN文件](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)

實際看看以下的程式碼:
```javascript
const milk = 1
const milkProto = Object.getPrototypeOf(milk)
const milkProtoProto = Object.getPrototypeOf(milkProto)
const milkProtoProtoProto = Object.getPrototypeOf(milkProtoProto)

console.log(milkProto)
// {}

console.log(milkProtoProto)
// [Object: null prototype] {}

console.log(milkProtoProtoProto)
// null
```

`console.log(milkProto)`這一行程式碼在console中可以看到:
![](https://hackmd.io/_uploads/rJWVBKrCh.png)
它的原型中包含建構式(constructor)`Number()`函式以及各種這個原型建構的實例可以使用的方法(method)，`[[Prototype]]` 則可以觀察到這個`Number`的原型是`Object`，也就是原型鏈的上一層。

`console.log(milkProtoProto)`這一行程式碼在console中則可以看到:
![](https://hackmd.io/_uploads/SJ-58tHCh.png)
`__proto__: (...)` 也就是再往原型鏈的上層找不到東西了，所以`console.log(milkProtoProtoProto)`印出的是`null`

## :memo: 範例
說了那麼多，直接用程式碼操作:
> Note: 以下例子使用[`class`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)語法糖撰寫，和傳統的建構函式會略有不同，關於兩者的比較也可以參考[另一篇文章](https://hackmd.io/@yy933/B1MGs4Tni)。
### `extends`和`super`
```javascript
class Drink {
  constructor(name, cost) {
    this._name = name
    this._cost = cost
  }
  // getter  
  get name() {
    return this._name
  }
  get cost() {
    return this._cost
  }
  get amount(){
    return this._amount
  }
  // method
  amountAdded(){
    this._amount++
  }
} 
```
先定義一個class`Drink`，接著定義一個class`Coffee`:
(關於屬性名稱為何要使用底線，請參考[這個問題](https://stackoverflow.com/questions/54562790/cannot-set-property-which-only-has-getter-javascript-es6))
```javascript
class Coffee {
  constructor(name, cost, origin) {
    this._name = name
    this._cost = cost
    this._origin = origin
  }
  get name() {
    return this._name
  }
  get cost() {
    return this._cost
  }
  get origin() {
    return this._origin
  }
} 
```
class`Coffee`的屬性和方法，繼承自class`Drink`，可以使用`extends`關鍵字進行繼承，所以改寫`Coffee`如下：
```javascript
class Coffee extends Drink {
  constructor(name, cost, origin) {
    super(name, cost);
    this._origin = origin;
  }
  get origin(){
    return this._origin
  }
  get coffeeInfo() {
    return `The ${this._name} from ${this._origin} costs ${this._cost} dollars.`
  }
}
```
:pushpin: Points：
- `extends`關鍵字讓`Coffee`可以使用父類別`Drink`中的**方法**。
- `super`關鍵字**呼叫父類別中的建構式，並可以取用父類別的屬性與方法**。在以上範例中，`super(name, cost)`將`name`和`cost`兩個argument傳入父類別`Drink`中的建構式，並執行產生新的`Coffee`物件實例。
- 值得注意的是，**當使用建構式時，`super`關鍵字必須在`this`關鍵字之前使用，確保新的物件已經在父類別的建構式中建立，此時`this`會指向這個新建立的物件**；如果沒有在`this`之前呼叫`super`，則會出現`reference error`，好的做法是**在子類別建構式的第一行使用`super`關鍵字**。
- `origin`是`Coffee`中的新屬性，所以在此處的建構式中定義它。
::: success
`class`語法糖中，我們可以直接把共用的方法(method)寫在class裡面；如果是用ES6以前的建構式，寫法相當於:
```javascript
// amountAdded function
Drink.prototype.amountAdded = function () {
 return this._amount++
}
```
或者是使用[`Object.assign`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)的語法:
```javascript
const amountAdded = {
  function () {
    return this._amount++
  }
}
Object.assign(Drink.prototype, amountAdded);
```
另外，在繼承父類別時，`class`語法糖使用`extends`關鍵字；ES6以前則可以使用[`call()`](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 函式，getter、setter則可以使用[`Object.defineProperty()`函式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。
Reference: [MDN docs - Object prototypes](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object_prototypes)
:::
接著，用子類別`Coffee`創造一個新的實例看看：
```javascript
const latte = new Coffee('latte', 5, 'Brazil')
console.log(latte.name)
// output: latte
```
上述程式碼中，我們創造了一個新的`Coffee`實例`latte`，因為`latte`可以取用父類別`Drink`中的getter`name`，所以會回傳儲存在`this._name`屬性中的值，也就是latte。

接著再執行以下程式碼:
```javascript
latte.amountAdded()
console.log(latte.amount)
// output: 1
```
以上程式碼做了什麼呢?
1. `Coffee`繼承了父類別`Drink`的`_amount`、amount getter、以及`amountAdded`函式
2. 當我們建立了實例`latte`，`Drink`的建構式把`_amount`屬性設為0
3. 因為繼承了`amountAdded`函式，所以新建立的實例`latte`可以呼叫這個方法並執行，使儲存在`_amount`屬性的值+1
4. 最後呼叫amount getter，所以會回傳儲存在`this._amount`屬性中的值，也就是1

如果我們再定義另一個子類別`Tea`，同樣繼承`Drink`的屬性和方法:
```javascript
class Tea extends Drink {
  constructor(name, origin) {
    super(name);
    this._origin = origin;
  }
  get origin(){
    return this._origin
  }
}
```
一樣試試看是否可以調用name getter:
```javascript
const blackTea =new Tea('Black Tea', 'India')
console.log(blackTea.name)
// output: Black Tea
```
成功調用name getter! 再來看看和`Tea`的原型和剛剛建立的`Coffee`是否相同:
```javascript
console.log(Object.getPrototypeOf(Tea) === Object.getPrototypeOf(Coffee))
// output: true
```


### 靜態方法(Static Methods)
有時，我們希望class中具有個別實例中不可調用的方法，但可以直接從該class中調用這些方法。這些方法稱為靜態方法(Static Methods)。這些方法可以用`static`關鍵字調用。

延續前面的範例:
```javascript
class Drink {
  constructor(name, cost) {
    this._name = name
    this._cost = cost
    this._amount = 0
  }
  get name() {
    return this._name
  }
  get cost() {
    return this._cost
  }
  get amount(){
    return this._amount
  }
  amountAdded(){
    this._amount++
  }
  static rating(){
     const randomNumber = Math.floor(Math.random()*5)
    return randomNumber
  }  
} 
```
直接從`Drink`調用`rating`方法:
```javascript
console.log(Drink.rating())
// output: <random rating>
```
繼承類別，也會繼承靜態方法:
```javascript
console.log(Coffee.rating())
// output: <random rating>
```

但是如果是該類別或繼承的子類別所建立的實例，調用該方法:
```javascript
const drink = new Drink('drink', 2)
console.log(drink.rating())
// TypeError: drink.rating is not a function

const latte = new Coffee('latte', 5, 'Brazil')
console.log(latte.rating())
// TypeError: latte.rating is not a function
```
則會出現錯誤，因為無法從實例上調用靜態方法。

最後再看一個例子:

`Math`是一個JavaScript的內建物件，它擁有多種靜態屬性與方法。比較特別的是，和大多數的全域物件(global object)不同，`Math`不是建構式(constructor)，也就是說，無法使用`new`運算子來建立一個實例(instance)，但是可以直接從`Math`調用靜態方法，例如調用[`Math.log()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/log)這個靜態方法：
```javascript
console.log(Math.log(1))
// output: 0
```

---

## 參考資料
* MDN文件
* [原型繼承與原型鏈](https://javascript.alphacamp.co/prototype-prototype-chain.html)
* [Day13-圖解原型繼承與原型鏈](https://ithelp.ithome.com.tw/articles/10289866)
* [Prototypal inheritance](https://javascript.info/prototype-inheritance)
* [Day 02: JavaScript 與 物件導向程式設計](https://ithelp.ithome.com.tw/articles/10265849#:~:text=%E5%B0%81%E8%A3%9D%E6%80%A7%EF%BC%88Encapsulation%EF%BC%89&text=%E4%B8%AD%E6%96%87Wiki%20%E5%AE%9A%E7%BE%A9%EF%BC%88%E9%80%A3%E7%B5%90%EF%BC%89%EF%BC%9A,%E8%83%BD%E4%B8%8D%E8%83%BD%E8%A2%AB%E9%9A%B1%E8%97%8F%E8%B5%B7%E4%BE%86%E3%80%82)
* [Inheritance in OOPS: An Idea of Code Reusability](https://www.enjoyalgorithms.com/blog/inheritance-in-java)
* [[教學] JavaScript ES6 Class：深入淺出類別概念與應用](https://www.shubo.io/javascript-class/#%E7%B9%BC%E6%89%BF%E9%9D%9C%E6%85%8B%E6%96%B9%E6%B3%95-static-method)
* Codecademy 教材

::: success
:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！
:::