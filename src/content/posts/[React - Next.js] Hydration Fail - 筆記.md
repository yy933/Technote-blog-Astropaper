---
title: "[React - Next.js] Hydration Fail - 筆記"
pubDatetime: 2026-05-26T03:29:26.491Z
tags: ["React.js","Node.js","Next.js","project","Debuggung"]
description: " Table of contents 問題 利用Next.js做Memory Game專案時，有翻牌配對的環節，相關的程..."
---

## Table of contents

## 問題
利用Next.js做Memory Game專案時，有翻牌配對的環節，相關的程式碼如下：
`// MemoryCardList.tsx`
```ts
"use client";
import MemoryCardItem from "./MemoryCardItem";
import { MemoryCardListProps } from "@/types";

export default function MemoryCard({
  handleClickAction,
  data,
  selectedCards,
  matchedCards
}: MemoryCardListProps) {
  
  return (
    <ul className="card-container">
      {data.map((emojiData, index) =>{ 
        const selectedCardEntry = selectedCards.find(
          (card) => card.index === index
        );
        const matchedCardEntry = matchedCards.find(
          (card) => card.index === index
        );
        return(
        <MemoryCardItem
          key={index}
          name={emojiData.name ?? "unknown"}
          index={index}
          htmlCode={emojiData?.htmlCode ?? []}
          handleClickAction={handleClickAction}
          selectedCardEntry={selectedCardEntry}
          matchedCardEntry={matchedCardEntry}
        />
      )})}
    </ul>
  );
}
```

`// MemoryCardItem.tsx`
```ts
import { decode } from "html-entities";
import EmojiButton from "./EmojiButton";
import { MemoryCardItemProps } from "@/types";
export default function MemoryCardItem({
  name,
  index,
  htmlCode,
  handleClickAction,
  selectedCardEntry,
  matchedCardEntry,
}: MemoryCardItemProps) {
  const content = htmlCode[0] ? decode(htmlCode[0]) : "?";

  const cardStyle = matchedCardEntry
    ? "card-item--matched"
    : selectedCardEntry
    ? "card-item--selected"
    : "";
  return (
    <li className="card-item">
      <EmojiButton
        name={name}
        index={index}
        content={content}
        className={`btn btn--emoji ${cardStyle}`}
        handleClickAction={handleClickAction}
        selectedCardEntry={selectedCardEntry}
        matchedCardEntry={matchedCardEntry}
      />
    </li>
  );
}
```

`// EmojiButton.tsx`
```ts
import { EmojiButtonProps } from "@/types";
export default function EmojiButton({
  content,
  className,
  handleClickAction,
  name,
  index,
  selectedCardEntry,
  matchedCardEntry
}: EmojiButtonProps) {
  const btnContent = selectedCardEntry || matchedCardEntry ? content : "?"
  const btnExtraStyle = matchedCardEntry
    ? "btn--emoji--matched"
    : selectedCardEntry
    ? "btn--emoji--selected"
    : "";
  return (
    <button
      className={`${className} ${btnExtraStyle}`}
      onClick={(e) => {
        e.preventDefault();
        handleClickAction(name, index);
      }}
    >
      {btnContent}
    </button>
  );
}
```

一切看似正常，在瀏覽器上試玩好像也沒問題，但是翻了幾張牌之後出現了這個error：
```
Hydration failed because the server rendered HTML didn't match the client. As a result this tree will be regenerated on the client. This can happen if a SSR-ed Client Component used:

- A server/client branch `if (typeof window !== 'undefined')`.
- Variable input such as `Date.now()` or `Math.random()` which changes each time it's called.
- Date formatting in a user's locale which doesn't match the server.
- External changing data without sending a snapshot of it along with the HTML.
- Invalid HTML tag nesting.

It can also happen if the client has a browser extension installed which messes with the HTML before React loaded.
```

## 為什麼發生Hydration fail?
### Next.js 是 SSR-first framework（Server Side Rendering）：

* Next.js 預設 server 端先產生一份 HTML → 傳給 browser。
* React 在 client 端接管這份 server HTML → 做 hydration。
* Hydration = React 必須確認「server HTML 跟 client 渲染後的 Virtual DOM 完全一致」。
* 如果 server 渲染的 HTML 跟 client 渲染的內容有差 → hydration fail。

### Memory Game的流程


* Next.js server render 時：
`selectedCards` 是空的 → `MemoryCardItem` 可能渲染 `?`。

* React client 接管後：
點了一張卡，`selectedCards` 更新 → `MemoryCardItem` 突然變成 emoji。

* 這時 React hydration 發現：
  - server 傳來 `<button>?</button>`。
  - client render 變成 `<button>🐶</button>`。
  - 不一致 → Hydration fail。

👉 所以 Next.js 只要有 UI 會根據「client state」改變，server 端又事先 render 過 → 很容易 hydration failed。

## 拆解問題
在`EmojiButton.tsx`中這段：
```ts
const btnContent = selectedCardEntry || matchedCardEntry ? content : "?";
```
這個 `btnContent` 是根據「狀態」變化的 → 
可是 Next.js 預設會先做 SSR，server render 的時候 `selectedCardEntry / matchedCardEntry` 可能是空的，結果畫面是 `?` → 
但 client render 的時候狀態更新了，畫面變成 emoji → 不一致 → hydration failed 。

## 常見解法
### 1. 用「延遲 client render」技巧
```tsx
import { useEffect, useState } from "react";

export default function MemoryCardItem(...) {
  const [isMounted, setIsMounted] = useState(false);

  useEffect(() => {
    setIsMounted(true);
  }, []);

  if (!isMounted) {
    // 只要第一次 server render，先 return null 或 loading skeleton
    return null;
  }

  // 這裡才 render 真正的卡片
  return (
    <li className="card-item">
      ...
    </li>
  );
}
```
這樣一來，第一次 server render 的時候不會 render 卡片，等 client mount 完再開始出現卡片（保證 hydration 不會 fail）。

### 2. 更好的方式 → 整個 MemoryCardList 直接 use client + 在 MemoryCardItem 裡用 isMounted 技巧。
目前 MemoryCardList 是：
```tsx
export default function MemoryCard({ handleClickAction, data }: MemoryCardListProps) {
```

但是 MemoryCardList 也會傳 `selectedCards / matchedCards` → 要加 use client：
```tsx
// MemoryCardList.tsx
"use client";
```
MemoryCardItem 本來就 use client → 可以在裡面加 isMounted 更保險。

### 3. EmojiButton 部分 
不要動態改 `btnContent`，用「front/back 兩層 div 翻轉」，這樣 class 不會改變 dom structure，可以避免 hydration fail。

👉 為什麼？ 
直接改 innerText 內容 React 會因為 server/client mismatch 出錯；用 CSS flip，html 結構固定，只是 transform，React hydration 不會 fail。

例如把MemoryCardItem改成以下結構，CSS控制翻牌動畫：
```tsx
// MemoryCardItem.tsx
...
export default function MemoryCardItem({
  name,
  index,
  htmlCode,
  handleClickAction,
  selectedCards,
  matchedCards
}: MemoryCardItemProps) {
  const content = htmlCode[0] ? decode(htmlCode[0]) : "?";
  const selectedCardEntry = selectedCards.find((card) => card.index === index);
  const matchedCardEntry = matchedCards.find((card) => card.index === index);

  const isFlipped = selectedCardEntry || matchedCardEntry;
  const isMatched = !!matchedCardEntry;

  return (
    <li className="card-item">
      <div className="card">
        <div className={`card__inner ${isFlipped ? "is-flipped" : ""}`}>
          <div
            className="card__front"
            onClick={() => {
              if (!isMatched && !selectedCardEntry && selectedCards.length < 2) {
                handleClickAction(name, index);
              }
            }}
          >
            ?
          </div>
          <div
            className={`card__back ${isMatched ? "matched" : ""}`}
          >
            {content}
          </div>
        </div>
      </div>
    </li>
  );
}
```

## 為什麼 Memory Game 特別容易出現hydration fail？
* 因為 Memory Game 有「狀態改變 → UI 瞬間變化」，而這些狀態是來自父層 component 的 useState。Server 端 render 不知道你按了什麼卡 → 跟 client 會 mismatch。
* Next.js容易出現這種問題，如果是純 React，因為是「client-only」，一開始就只有 React 控制 DOM，不存在 server-render 的 HTML，因此不存在 mismatch 的問題。