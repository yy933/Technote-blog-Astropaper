---
title: "[Concept] Map 與 Set：JavaScript / TypeScript 的雜湊式資料結構 - 筆記"
pubDatetime: 2026-11-18T08:55:30.326Z
tags: ["Concepts","TypeScript","Interview Preparation","JavaScript","Data Structure","Big O","React.js","Issue","Leetcode","Map","Set"]
description: "Tags: JavaScript TypeScript Map Set Data Structure Big O Le..."
---

###### Tags: `JavaScript` `TypeScript` `Map` `Set` `Data Structure` `Big O` `LeetCode` `React.js` `Concepts` `Issue` `Interview Preparation`


## Table of contents

## 問題（Issue）：為什麼需要 Map 和 Set？

### 用 Array 查找，每次都是 O(n)

```typescript
const seen = [2, 7, 11, 15];

seen.includes(11); // 要從頭一個一個比對，最壞情況要看完整個陣列 → O(n)
```

當資料量變大、又需要「反覆查詢」時，陣列的查找成本會直接疊加。例如 [Two Sum](https://leetcode.com/problems/two-sum/) 的暴力解，就是為了每個元素再掃一次陣列找補數，整體變成 O(n²)。

### 用 Object 當字典，會有一堆隱藏的坑

很多人習慣用 `{}` 當作「鍵值對」的容器，但 Object 原本不是為此設計的：

```typescript
const obj: Record<string, number> = {};

// 1. key 一律被轉成字串（或 Symbol）
obj[1] = 100;
obj["1"]; // 100，數字 1 和字串 "1" 被當成同一個 key

// 2. 繼承自原型的屬性會干擾判斷
const o = {};
"toString" in o; // true，明明什麼都沒存

// 3. 想知道有幾筆資料，要先轉成陣列
Object.keys(obj).length;
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

Object 當字典的核心問題：**key 型別受限、有原型鏈干擾、需要自己處理很多邊界情況**。

</blockquote>

### Array 去除重複也很麻煩

```typescript
// 手寫去重：每個元素都要用 includes 檢查 → O(n²)
const unique: number[] = [];
for (const n of [1, 2, 2, 3, 3]) {
  if (!unique.includes(n)) unique.push(n);
}
```

**Map 和 Set 就是為了解決這些問題而生**：它們提供「接近 O(1) 的查詢」，而且語意清楚、行為可預測。

## Map 與 Set 的功能（為什麼要使用）

* **快速查詢**：`has`、`get`、`set`、`delete` 平均都是 O(1)，用「空間換時間」把 O(n) 的查找降下來。
* **Map = 鍵值對（key → value）**：適合「由 key 查出對應資料」，例如「值 → 索引」、「字元 → 出現次數」。
* **Set = 不重複的值的集合**：適合「判斷在不在」、「去除重複」、「記錄是否走訪過」。
* **key 可以是任何型別**：數字、字串、物件、函式都可以（Object 做不到）。
* **保證插入順序**：走訪時依「放進去的順序」，不用擔心順序亂掉。
* **內建 `size`**：不用像 Object 一樣先轉陣列才知道數量。
* **不會被原型鏈干擾**：`new Map()` 一開始是完全空的。
* **好用的走訪方式**：支援 `for...of`、`forEach`、`keys()`、`values()`、`entries()`。

## Map 與 Set 的特性

### Map vs Object

| | Map | Object |  
|---|---|---|  
| key 的型別 | **任何型別** | 只有字串與 Symbol（數字會被轉成字串） |  
| 插入順序 | 保證依插入順序 | 大致有序，但整數 key 會被排到前面 |  
| 取得數量 | `map.size` | `Object.keys(obj).length` |  
| 原型鏈干擾 | 無 | 有（如 `toString`、`__proto__`） |  
| 走訪 | 可直接 `for...of` | 要先 `Object.keys / entries` |  
| 頻繁新增、刪除 | 效能較好 | 較不適合 |  
| JSON 序列化 | **不支援**，需自行轉換 | 原生支援 |

### Set vs Array

| | Set | Array |  
|---|---|---|  
| 元素是否可重複 | **不可重複** | 可以 |  
| 判斷元素是否存在 | `has` → 平均 O(1) | `includes` → O(n) |  
| 用索引取值 | 不行 | `arr[i]` |  
| 是否有序 | 依插入順序，但沒有索引 | 有索引 |  
| 常見用途 | 去重、判斷是否存在 | 有順序、要用索引的資料 |

### Map 與 Set 的差異

| | Set | Map |  
|---|---|---|  
| 存什麼 | 只存「值」 | 存「鍵值對」 |  
| 主要用途 | 判斷某個東西**在不在** | 由 key **查出對應資料** |  
| 常用方法 | `add` / `has` / `delete` | `set` / `get` / `has` / `delete` |  
| 重複的處理 | 重複的值只保留一個 | 相同的 key 會被**覆蓋** |

### 時間複雜度

| 操作 | Map / Set | Array |  
|---|---|---|  
| 新增 | 平均 O(1) | `push` O(1) |  
| 查詢是否存在 | 平均 O(1) | `includes` O(n) |  
| 刪除指定元素 | 平均 O(1) | O(n) |  
| 走訪全部 | O(n) | O(n) |  
| 空間 | O(n) | O(n) |

### 相等判斷：SameValueZero

Map 的 key 和 Set 的值，是用 **SameValueZero** 規則比較：

* 基本型別比「值」：`1` 和 `1` 相同，`"1"` 和 `1` 不同。
* **物件比「參考（reference）」**：內容一樣但不是同一個物件，就視為不同。
* `NaN` 會被視為與 `NaN` 相同（雖然 `NaN === NaN` 是 false）。
* `+0` 與 `-0` 視為相同。

## 程式碼範例

### 1. Map 基本操作

```typescript
const map = new Map<string, number>();

map.set("apple", 3);
map.set("banana", 5).set("cherry", 7); // set 會回傳 map 本身，所以可以鏈式呼叫

map.get("apple");   // 3
map.get("durian");  // undefined（不存在時）
map.has("banana");  // true
map.size;           // 3

map.delete("banana");
map.clear();        // 清空全部

// 也可以用「鍵值對陣列」初始化
const m2 = new Map<string, number>([
  ["a", 1],
  ["b", 2],
]);
```

### 2. Set 基本操作

```typescript
const set = new Set<number>([1, 2, 2, 3]);
set.size; // 3，重複的 2 只保留一個

set.add(4).add(4); // 一樣可以鏈式呼叫，重複加入不會有效果
set.has(3);        // true
set.delete(1);
set.clear();
```

### 3. 走訪（Iteration）

```typescript
const map = new Map<string, number>([
  ["a", 1],
  ["b", 2],
]);

for (const [key, value] of map) {
  console.log(key, value);
}

map.forEach((value, key) => {
  // 注意：forEach 的參數順序是 (value, key)
});

[...map.keys()];    // ["a", "b"]
[...map.values()];  // [1, 2]
[...map.entries()]; // [["a", 1], ["b", 2]]

const set = new Set(["x", "y"]);
for (const item of set) {
  console.log(item);
}
```

### 4. 陣列去重

```typescript
const nums = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(nums)]; // [1, 2, 3]
// 或 Array.from(new Set(nums))
```

### 5. 計數器（Frequency Counter）

```typescript
function countChars(s: string): Map<string, number> {
  const count = new Map<string, number>();
  for (const ch of s) {
    count.set(ch, (count.get(ch) ?? 0) + 1); // 沒有就當作 0
  }
  return count;
}

countChars("hello"); // Map { 'h' => 1, 'e' => 1, 'l' => 2, 'o' => 1 }
```

* `count.get(ch)` 的型別是 `number | undefined`，所以用 `?? 0` 給預設值。

### 6. Two Sum：用 Map 把「找補數」降到 O(1)

```typescript
function twoSum(nums: number[], target: number): number[] {
  const map = new Map<number, number>(); // 值 → 索引
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    if (map.has(complement)) {
      return [map.get(complement)!, i];
    }
    map.set(nums[i], i); // 先查再存，自己不會被查到
  }
  return [];
}
```

* 時間 O(n)、空間 O(n)。
* `!` 是 TypeScript 的非空斷言，因為已經用 `has` 確認過存在。

### 7. Contains Duplicate：用 Set 判斷是否有重複

```typescript
function containsDuplicate(nums: number[]): boolean {
  return new Set(nums).size !== nums.length;
}
```

### 8. Group Anagrams：Map 的 value 是陣列

```typescript
function groupAnagrams(strs: string[]): string[][] {
  const groups = new Map<string, string[]>();
  for (const s of strs) {
    const key = [...s].sort().join(""); // 排序後的字串當作 key
    const list = groups.get(key) ?? [];
    list.push(s);
    groups.set(key, list);
  }
  return [...groups.values()];
}
```

### 9. Longest Substring Without Repeating Characters：Set 搭配滑動視窗

```typescript
function lengthOfLongestSubstring(s: string): number {
  const window = new Set<string>();
  let left = 0;
  let best = 0;

  for (let right = 0; right < s.length; right++) {
    while (window.has(s[right])) {
      window.delete(s[left]); // 有重複就從左邊縮小視窗
      left++;
    }
    window.add(s[right]);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

### 10. 走訪過的位置（visited）

```typescript
// 二維座標不能直接放進 Set（陣列是用參考比較），所以轉成字串當作 key
const visited = new Set<string>();

visited.add(`${row},${col}`);
visited.has(`${row},${col}`); // true
```

### 11. 集合運算（聯集、交集、差集）

```typescript
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

const union = new Set([...a, ...b]);                       // {1, 2, 3, 4}
const intersection = new Set([...a].filter((x) => b.has(x))); // {2, 3}
const difference = new Set([...a].filter((x) => !b.has(x)));  // {1}
```

* 較新的 JavaScript 環境也提供了內建方法，例如 `a.union(b)`、`a.intersection(b)`、`a.difference(b)`。
* 使用前要確認執行環境（瀏覽器、Node.js 版本）與 TypeScript 的 `lib` 設定是否支援。

### 12. Map、Object、Array 互相轉換

```typescript
const map = new Map<string, number>([["a", 1], ["b", 2]]);

// Map → Object
const obj = Object.fromEntries(map); // { a: 1, b: 2 }

// Object → Map
const map2 = new Map(Object.entries(obj));

// Map → Array
const arr = Array.from(map); // [["a", 1], ["b", 2]]

// Set → Array
const set = new Set([1, 2, 3]);
const arr2 = [...set];
```

### 13. React：state 裡使用 Set，必須建立新的 Set

```tsx
import { useState } from "react";

const SelectableList = ({ ids }: { ids: number[] }) => {
  const [selected, setSelected] = useState<Set<number>>(new Set());

  const toggle = (id: number) => {
    setSelected((prev) => {
      const next = new Set(prev); // 複製一份新的，不要直接修改 prev
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next; // 回傳新的參考，React 才會重新渲染
    });
  };

  return (
    <ul>
      {ids.map((id) => (
        <li key={id} onClick={() => toggle(id)}>
          {id} {selected.has(id) ? "✅" : ""}
        </li>
      ))}
    </ul>
  );
};
```

## 實作流程（怎麼選擇資料結構）

1. **先問：需要用「索引」取值或排序嗎？**
   * 需要 → 用 Array。  
2. **只需要判斷「有沒有出現過」、「去除重複」？**
   * 用 **Set**。  
3. **需要由某個 key 查出對應的值（索引、次數、分組結果）？**
   * 用 **Map**。  
4. **key 是固定的、已知的字串欄位（像一筆資料的屬性）？**
   * 用 Object（或 interface / type）。  
5. **key 是動態的、數量不固定、或不是字串？**
   * 用 **Map**。  
6. **需要序列化成 JSON（存 localStorage、傳給 API）？**
   * 先轉成 Object / Array，或直接用 Object / Array。  
7. **在 React / Redux 的 state 裡使用？**
   * 更新時一律建立新的 Map / Set，不要直接修改。

**LeetCode 對照速查**

| 題型 | 資料結構 | 代表題目 |  
|---|---|---|  
| 找補數、配對 | `Map<值, 索引>` | 1 Two Sum |  
| 判斷是否重複 | `Set` | 217 Contains Duplicate |  
| 計次數 | `Map<元素, 次數>` | 242 Valid Anagram、347 Top K Frequent Elements |  
| 分組 | `Map<key, 陣列>` | 49 Group Anagrams |  
| 滑動視窗去重 | `Set` | 3 Longest Substring Without Repeating Characters |  
| 走訪標記（visited） | `Set<string>` | 200 Number of Islands |

## 其他注意事項

### JSON.stringify 不支援 Map 與 Set

```typescript
JSON.stringify(new Map([["a", 1]])); // '{}'，資料全部消失！
JSON.stringify(new Set([1, 2]));     // '{}'

// 正確做法：先轉換
JSON.stringify(Object.fromEntries(map)); // '{"a":1}'
JSON.stringify([...set]);                // '[1,2]'
```

<blockquote class="my-6 p-4 bg-orange-50 dark:bg-orange-950/30 border-l-4 border-orange-500 rounded-r-md text-orange-900 dark:text-orange-200 blocknoted-fix">

存進 `localStorage` 或回傳給 API 之前，記得先轉換，不然資料會悄悄消失，而且不會報錯。

</blockquote>

### 物件當作 key：用的是「參考」比較

```typescript
const m = new Map();
m.set({ id: 1 }, "a");
m.get({ id: 1 }); // undefined！這是另一個新物件

const s = new Set([[1, 2], [1, 2]]);
s.size; // 2，兩個陣列的內容一樣，但參考不同
```

* 想用「內容」比較時，通常要自己轉成字串當 key，例如 `` `${r},${c}` `` 或 `JSON.stringify(...)`。
* 若 key 就是同一個物件參考（例如 DOM 元素、同一個物件實例），Map 反而很好用。

### Map.get 的回傳型別是 `T | undefined`

* TypeScript 會要求你處理「找不到」的情況。
* 常見處理方式：
  * `map.get(k) ?? 預設值`
  * 先用 `has` 確認，再用 `!` 斷言
  * 用 `if (value !== undefined)` 做型別縮小

### 走訪時修改 Map / Set

* 在走訪過程中 **刪除** 元素是安全的。
* 走訪過程中 **新增** 元素，新元素也可能被走訪到，容易造成無窮迴圈，要小心。

### 展開運算子與編譯目標

* `[...new Set(arr)]` 在 TypeScript 的 target 低於 ES2015 時，可能需要開啟 `downlevelIteration`，或改用 `Array.from(new Set(arr))`。
* 一般現代專案（Vite、Next.js）預設就沒有這個問題。

### 排序

* Map 與 Set 本身沒有 `sort` 方法，需要先轉成陣列：

```typescript
const sorted = [...map.entries()].sort((a, b) => b[1] - a[1]); // 依 value 由大到小
```

### 「平均 O(1)」的意思

* Map / Set 內部是用雜湊表（hash table）實作，所以查詢、新增、刪除**平均**是 O(1)。
* 雜湊碰撞很嚴重時，理論上可能退化，但實務上不需要特別擔心。


### WeakMap 與 WeakSet（簡單認識）

* key（或值）**只能是物件**，而且是「弱參考」：當物件沒有其他地方在使用時，會被垃圾回收，不會造成記憶體洩漏。
* **不能走訪**，也沒有 `size`。
* 常見用途：替物件附加額外資料、快取，又不想阻止物件被回收。
* 一般刷題與日常開發很少用到，知道有這個東西即可。

### 缺點

* 不能直接 `JSON.stringify`，序列化要額外轉換。
* 沒有像陣列那樣的索引存取與方法（`map`、`filter`、`reduce` 要先轉成陣列）。
* 在 React / Redux 的 state 裡使用時，要記得複製出新的物件，直接修改不會觸發重新渲染。

## 小結

* **Set = 不重複的集合**，擅長「判斷在不在」與「去重」；**Map = 鍵值對**，擅長「由 key 查出 value」。
* 兩者的查詢、新增、刪除平均都是 **O(1)**，是用**空間換時間**，把 O(n²) 的暴力解降成 O(n)。
* Map 比 Object 更適合當字典：key 可以是任何型別、保證插入順序、有 `size`、沒有原型鏈干擾。
* 比較規則是 SameValueZero：基本型別比值，**物件比參考**。
* 常見用法：
  * Map：配對（Two Sum）、計數、分組
  * Set：去重、是否出現過、滑動視窗、visited 標記
* 注意事項：`JSON.stringify` 不支援、`Map.get` 可能是 `undefined`、React state 要建立新的 Map / Set。
* 刷題時看到「找補數」、「出現過嗎」、「統計次數」、「分組」，第一個就該想到 Map / Set。

## 參考資料

* [MDN：Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
* [MDN：Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
* [MDN：Keyed collections（Map、Set、WeakMap、WeakSet 總覽）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections)
* [LeetCode：1. Two Sum](https://leetcode.com/problems/two-sum/)