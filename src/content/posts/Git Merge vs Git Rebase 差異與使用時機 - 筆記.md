---
title: "Git Merge vs Git Rebase 差異與使用時機 - 筆記"
pubDatetime: 2026-09-29T08:29:58.867Z
tags: ["Interview Preparation","Git","Version Control","DevOps","Frontend"]
description: "Tags: Interview Preparation Git Version Control DevOps Fron..."
hackmd_id: "rJQTXgF5zl"
---

###### Tags: `Interview Preparation` `Git` `Version Control` `DevOps` `Frontend`


## Table of contents


## 核心觀念總覽

* **`git merge`（三方合併 / Non-Destructive）**：
  * **非破壞性**：保留所有分支的原始 Commit 紀錄與時間軸。
  * **額外提交**：合併時會產生一個新的 **Merge Commit** 來連接兩個分支。
  * **優點**：真實還原歷史脈絡，操作相對安全。
  * **缺點**：如果頻繁合併，歷史樹（Git Graph）會變得極度交錯混亂（「交錯的大腸包小腸」圖譜）。
* **`git rebase`（重新定義基準 / Linear History）**：
  * **重新打頂**：把當前分支的 Commit 「剪下」，重新貼在目標分支的最新 Commit 頂端。
  * **線性歷史**：不會產生新的 Merge Commit，讓 Commit 歷史呈現完美的**單一平行線**。
  * **優點**：歷史紀錄極度乾淨，易於追蹤與 `git log` 查閱。
  * **缺點**：會改寫 Commit 的 Hash ID 與歷史時間，使用不當會影響團隊其他人。



## 1. Merge vs Rebase 核心比較與速查

| 比較維度 | `git merge` | `git rebase` |  
| :--- | :--- | :--- |  
| **歷史紀錄 shape** | **分叉 / 分支樹狀圖**（分歧後再匯合） | **一條直線 (Linear History)** |  
| **產生 Merge Commit** | **會**（產生一個特殊的 2-parent Commit） | **不會** |  
| **Commit Hash** | 保持原本的 Commit Hash 不變 | **重新產生**（改寫變更的 Hash ID） |  
| **衝突處理 (Conflict)** | 一次性在 Merge Commit 中解決所有衝突 | 逐個 Commit 套用並解決衝突（需 `rebase --continue`） |  
| **安全性與還原** | 極高（原始紀錄完全保留，容易退回） | 中/低（改寫了歷史，還原需依賴 `git reflog`） |  
| **適用場景** | 將功能分支（Feature）合併進主幹（Main/Master） | 更新本地分支以同步主幹最新進度、清理提交訊息 |



## 2. 運作圖解與步驟說明

假設我們從 `main` 分支切出 `feature` 分支開發，期間 `main` 也發布了新的 Commit：

```text
       C3 (feature)
      /
C1---C2---C4 (main)
```

### 情境 A：使用 git merge main  
（將 main 的最新變更合併進 feature）

```bash
git checkout feature
git merge main
```

歷史結構演變：

```text
      C3-------M1 (feature)  <--- 產生全新的 Merge Commit (M1)
      /        /
C1---C2-------C4 (main)
```

- 特徵：C3 與 C4 的歷史都完整保留，最後多了一個 M1 來連接兩邊。

### 情境 B：使用 git rebase main  
（將 feature 的基底重新設定為 main 的最新位置 C4）

```bash
git checkout feature
git rebase main
```

歷史結構演變：

```text
                C3' (feature) <--- C3 被重新套用到 C4 上，Hash 變成 C3'
                /
C1---C2---C4 (main)
```

- 特徵：C3 被「移植」到 C4 後面，**整個歷史變成了 C1 ➔ C2 ➔ C4 ➔ C3' 的一條直線！**

## 3. Rebase 的黃金法則 (Golden Rule of Rebase)  
⚠️ **千萬不要對已經推送到公開/團隊共享遠端（Remote）的分支執行 Rebase！**

如果你對已經共享的分支（如 main 或團隊成員正在共同開發的分支）進行 Rebase，因為你的 Rebase 改寫了 Commit Hash，會導致其他成員的本地歷史與遠端完全對不起來，大家推拉程式碼時會引發極大規模的 Merge Conflict 與災難。

### 原則：

- 本地私有分支（Local Private Branch） ➔ 可以任意 Rebase（隨意清理歷史）。
- 公共共享分支（Public/Shared Branch） ➔ 絕對只用 Merge。

## 4. 最佳實務工作流 (Best Practice Workflow)  
許多團隊會採取 「Rebase + Merge (Squash)」 的組合拳：

```bash
# --------------------------------------------------
# 步驟 1: 在自己的 feature 分支開發完畢，準備發 PR 前
# --------------------------------------------------
git checkout main
git pull origin main                      # 拉取遠端最新 main

git checkout feature/login
git rebase main                           # 將自己的 feature 移植到最新的 main 頂端 (保持線型)

# --------------------------------------------------
# 步驟 2: 解決衝突後，推送至遠端 (需要 -f，因為 rebase 改寫了 Hash)
# --------------------------------------------------
git push origin feature/login --force-with-lease

# --------------------------------------------------
# 步驟 3: 在 GitHub/GitLab 上通過 Code Review 後
# --------------------------------------------------
# 選擇「Squash and Merge」或「Rebase and Merge」合併進 main 分支！
```

## 5. 面試深入回答策略 (Senior Response Strategy)  
建議分為三個層次回答，展現對工具的掌握度與團隊風險控管意識：

- 定義與歷史結構（What）：  
說明 Merge 會產生 Merge Commit 並保留分支樹狀圖；Rebase 則是透過重新對齊基底來維持「線性歷史（Linear History）」。

- Rebase 的優缺點（Pros & Cons）：  
優點是 git log 極度簡潔易讀；缺點是改寫歷史（Commit Hash 被替換），衝突時需要逐個 Commit 解決。

- 強調「黃金法則」與團隊實務（Senior Insight）：  
主動提出 **「只在本地/私有分支使用 Rebase 來拉取 main 最新進度」**，對於 **「公共共享分支（如 main/release）則堅持使用 Merge」**，並提及團隊通常會**搭配 `git rebase -i` (Interactive Rebase) 來整理過於零碎的 Commit 訊息（Squash）**，展現出軟體工程素養。

## FAQ
### Q1: 如果在 git rebase 的過程中遇到衝突 (Conflict) 該怎麼辦？  
Rebase 遇到衝突時不會像 Merge 一樣一次性解決，而是會停在發生的那一個 Commit 上：

1. 開啟衝突檔案手動修復。  
2. 將修復後的檔案加入暫存區：`git add .`（不需要執行 `git commit`）。  
3. 繼續執行 Rebase：`git rebase --continue`。  
（若想放棄 Rebase，可隨時輸入：`git rebase --abort` 回到原本狀態）。

### Q2: 強制推送時，為什麼建議使用 git push --force-with-lease 而不是 git push -f？
- `--force` (`-f`)：霸道地用本地的分支直接覆蓋遠端儲存庫，即使這期間隊友推送了新的程式碼也會被你直接蓋掉抹滅。

- `--force-with-lease`：相對安全的強制推送。**它會檢查遠端儲存庫是否有「你本地尚未下載的他人提交」，如果有，就會拒絕推送**，防止你意外覆蓋掉隊友寫好的程式碼。