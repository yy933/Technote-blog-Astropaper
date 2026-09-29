---
title: "Git 版本控制與常用指令 - 筆記"
pubDatetime: 2026-09-29T08:14:19.731Z
tags: ["Interview Preparation","Git","Version Control","DevOps","Frontend"]
description: "Tags: Interview Preparation Git Version Control DevOps Fron..."
hackmd_id: "BJ7h-lt9Gx"
---

###### Tags: `Interview Preparation` `Git` `Version Control` `DevOps` `Frontend`


## Table of contents


## 核心觀念總覽

* **什麼是 Git？**：Git 是一個**分散式版本控制系統（Distributed Version Control System, DVCS）**，用於追蹤程式碼的歷史變更紀錄、支援多人平行開發，並能防止程式碼覆蓋與遺失。
* **為什麼要使用 Git？（核心價值）**：
  * **歷史紀錄追蹤 (History Tracking)**：精確紀錄「誰」在「什麼時間」修改了「哪些程式碼」。
  * **分支平行開發 (Branching Workflow)**：讓不同工程師在各自的分支（Branch）開發新功能或修復 Bug，不影響主線（Main/Master）穩定度。
  * **團隊協作與 Code Review**：透過 Pull Request (PR) 機制，在合併程式碼前進行審查，確保品質。
  * **安全復原 (Time Travel/Rollback)**：當線上系統出現嚴重問題時，能迅速退回到穩定版本。
* **核心開發工作流（Git Flow）**：
  `工作區 (Working Directory)` ➔ `Staging Area (暫存區)` ➔ `Local Repository (本地儲存庫)` ➔ `Remote Repository (遠端儲存庫)`



## 1. 核心開發流與日常必備指令速查

| 步驟 / 階段 | 常用 Git 指令 | 說明與情境 |  
| :--- | :--- | :--- |  
| **1. 取得與分支** | `git checkout -b <branch-name>`<br>*(或 `git switch -c`)* | 從當前分支建立並切換至新的功能分支 (Feature Branch)。 |  
| **2. 取得最新程式碼** | `git pull origin <branch-name>` | 從遠端儲存庫拉取最新程式碼並與本地分支合併。 |  
| **3. 暫存變更** | `git add .` *(或 `git add <file>`)* | 將修改過的檔案加入 Staging Area（暫存區），準備提交。 |  
| **4. 提交版本** | `git commit -m "feat: add user login"` | 將暫存區的變更快照寫入本地儲存庫，並附上清晰的說明訊息。 |  
| **5. 推送遠端** | `git push origin <branch-name>` | 將本地分支的 Commit 紀錄推送至遠端儲存庫（如 GitHub / GitLab）。 |  
| **狀態檢查** | `git status` | 檢查當前工作區有哪些檔案被修改、暫存或未被追蹤（Untracked）。 |



## 2. 標準 Git 開發工作流範例 (Daily Git Workflow)

```bash
# --------------------------------------------------
# 步驟 1: 切換並建立新功能分支 (從主分支 main 開始)
# --------------------------------------------------
git checkout main
git pull origin main                 # 確保本地 main 是最新版本
git checkout -b feature/login-page   # 建立並切換到 feature 分支

# --------------------------------------------------
# 步驟 2: 開發程式碼後，檢查狀態並加入暫存區
# --------------------------------------------------
git status                           # 查看被修改的檔案
git add .                            # 將所有修改加入暫存區

# --------------------------------------------------
# 步驟 3: 提交 Commit 歷史紀錄
# --------------------------------------------------
git commit -m "feat: implement user login validation logic"

# --------------------------------------------------
# 步驟 4: 推送至遠端並準備建立 Pull Request (PR)
# --------------------------------------------------
git push origin feature/login-page
```

## 3. 回答策略 (Senior Response Strategy)
- 定義與對比：  
簡短說明 Git 是分散式版本控制系統。可以舉出反例（如過去透過隨身碟、Email 傳遞檔案，或命名 `final_v2_final.zip` 的亂象），襯托出 Git 如何解決多人協作與衝突管理。

- 順暢講出日常指令鏈（Standard Workflow）：  
按順序講出：**branch ➔ add ➔ commit ➔ pull ➔ push**，展現你每天都在寫程式、使用命令行或 GUI 工具的熟練度。

- 主動提及規範與實務：
  - 提及 Commit Message 規範（如 Conventional Commits：feat:, fix:, refactor:），展現團隊協作素養。
  - 提及  Pull Request (PR) / Code Review 流程，說明程式碼不會直接改動 main 分支。

## FAQ
### Q1: git fetch 與 git pull 有什麼差別？
- `git fetch`：僅將遠端儲存庫的最新變更下載到本地，不會自動合併到你當前正在工作的分支。**非常安全，適合用來查看團隊其他人的進度**。

- `git pull`：等於 **git fetch + git merge**。它會下載遠端變更並立刻嘗試與你當前分支合併。**如果兩邊修改了同一個地方，就會觸發 Merge Conflict（衝突）。**

### Q2: 如果開發到一半需要切換分支修 Bug，但當前變更還沒完成不適合 Commit，該怎麼辦？  
可以使用 `git stash` 將目前工作區暫存起來：

```bash
# 1. 暫存當前未完成的修改
git stash

# 2. 切換至其他分支修復 Bug
git checkout hotfix/bug-123
# ...完成修復並 push...

# 3. 切回原本的分支並恢復先前暫存的修改
git checkout feature/my-feature
git stash pop
```