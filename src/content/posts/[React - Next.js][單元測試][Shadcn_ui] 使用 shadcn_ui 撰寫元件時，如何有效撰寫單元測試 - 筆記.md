---
title: "[React - Next.js][單元測試][Shadcn/ui] 使用 shadcn/ui 撰寫元件時，如何有效撰寫單元測試 - 筆記"
pubDatetime: 2025-07-07T20:35:42.000Z
modDatetime: 2026-05-25T10:04:23.349Z
tags: ["React.js","Node.js","Next.js","Project","Debugging","Unit Test","Shadcn"]
description: "Table of contents 問題 使用shadcn/ui開發的過程中，很常遇到的問題是，由於shadcn是Ra..."
---

## Table of contents

## 問題  
使用shadcn/ui開發的過程中，很常遇到的問題是，由於shadcn是Radix-based 的高階抽象複合元件，不是原生HTML元件，因此在寫單元測試的過程中，無法用 React Testing Library 中常見的 query 抓到元件（例如 `getByRole("combobox")`），使得測試不通過。

## 解決方法
### 解法 1. 封裝 UI 元件 + 單獨測試核心邏輯  
比如說有一個 `PlansForm` 元件，可以將狀態與邏輯抽成 hook，與 shadcn UI 解耦。
```ts
// usePlansForm.ts
export function usePlansForm() {
  const [menu, setMenu] = useState("classic")
  const [preferences, setPreferences] = useState<string[]>([])
  const [servings, setServings] = useState("")
  const [meals, setMeals] = useState("")

  const submit = async () => {
    return await fetch("/api/submit", {
      method: "POST",
      body: JSON.stringify({ menu, preferences, servings, meals }),
    })
  }

  return {
    menu,
    setMenu,
    preferences,
    setPreferences,
    servings,
    setServings,
    meals,
    setMeals,
    submit,
  }
}
```
單元測試：
```tsx
import { renderHook, act } from "@testing-library/react"

it("submits correct payload", async () => {
  const { result } = renderHook(() => usePlansForm())

  act(() => {
    result.current.setMenu("vegetarian")
    result.current.setPreferences(["gluten-free"])
    result.current.setServings("2")
    result.current.setMeals("5")
  })

  await result.current.submit()

  expect(fetch).toHaveBeenCalledWith("/api/submit", expect.objectContaining({ ... }))
})
```
優點:
* 不會被 shadcn DOM 結構影響
* 可以獨立測試邏輯與 API 行為

### 解法 2. mock 掉 shadcn 元件（維持 UI 測試）  
如果要測試整個表單行為、畫面、互動（非邏輯），可以 mock 掉 shadcn 元件（像 `Select`），還原為原生元件，例如以下：
```tsx
vi.mock("@/components/ui/select", () => {
  return {
    Select: ({ children, ...props }: any) => <select {...props}>{children}</select>,
    SelectTrigger: ({ children }: any) => <>{children}</>,
    SelectValue: ({ children }: any) => <>{children}</>,
    SelectContent: ({ children }: any) => <>{children}</>,
    SelectItem: ({ children, value }: any) => (
      <option value={value}>{children}</option>
    ),
    // ...
  }
})
```
測試:
```tsx
await user.selectOptions(screen.getByRole("combobox", { name: /Servings/i }), "2")
```
優點：
* 保留互動流程，維持簡單的 DOM 結構

### 解法 3. 使用完整互動模擬（最貼近真實，但也最複雜、容易壞）  
如果希望保留 shadcn 的互動方式（測試 radix 的行為），就要：
* 使用 userEvent 模擬點擊下拉選單
* 等待 dropdown 出現
* 再選擇項目

```tsx
import userEvent from "@testing-library/user-event"

const user = userEvent.setup()

await user.click(screen.getByRole("combobox", { name: /Servings/i }))
await user.click(screen.getByRole("option", { name: "2" }))
```
有時需配合 `findByText`、`within`、`act()` 解決非同步與 portal 問題。

* 適合 e2e 或需要真實模擬的場景。
* 但容易受 shadcn 或 radix 更新影響而破壞。

## 實務建議流程

* 邏輯與 UI 拆開： 把狀態處理抽成 hook
* 寫邏輯測試：針對邏輯邏輯用 hook 測試為主更穩定
* 寫 UI 行為測試（選擇其一）：
  - mock 掉 shadcn 元件（中小型專案建議）
  - 或完整互動模擬（UI 完整性要求高時使用）
* 如果 UI 頻繁變動，推薦用 mock 模擬元件 + e2e 測關鍵流程（Playwright / Cypress）

## Recap  
| 方法           | 用途           | 是否穩定  | 難度   |  
| ------------ | ------------ | ----- | ---- |  
| 抽邏輯成 hook    | 測邏輯、API      |  最穩定 | ⭐    |  
| mock 掉 UI 元件 | 測整體行為但不測互動動畫 |  穩定  | ⭐⭐   |  
| 不 mock、直接互動  | 模擬真實操作       | 貼近實際情況，但容易壞 | ⭐⭐⭐⭐ |