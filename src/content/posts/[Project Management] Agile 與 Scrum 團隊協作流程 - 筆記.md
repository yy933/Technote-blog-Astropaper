---
title: "[Project Management] Agile 與 Scrum 團隊協作流程 - 筆記"
pubDatetime: 2026-09-29T10:25:01.648Z
tags: ["others"]
description: "Tags: Interview Preparation Agile Scrum Project Management..."
hackmd_id: "ByPw1ftqfg"
---

###### Tags: `Interview Preparation` `Agile` `Scrum` `Project Management` `Frontend`


## Table of contents



## 核心觀念總覽

* **敏捷開發的本質**：Agile 是一種團隊運作流程與哲學，非單純的技術硬實力。
* **實務現狀（Industry Reality）**：沒有任何一家公司會 100% 照著教科書（Textbook Definition）執行 Agile/Scrum。**每個組織與團隊都會根據自身狀況彈性微調（Slightly Different）**，這是完全正常的現象。
* **沒有職場經驗時的回答策略**：若無實務經驗，不必硬捏造；可以轉為**講述敏捷的精神與核心原則（Tenets of Agile）**，並展現你對團隊協作流程的理解。
* **Scrum vs. Kanban 的核心差異**：
  * **Scrum**：有固定週期（Sprint）、估算估分（Story Points）與速度（Velocity）。
  * **Kanban**：通常不估分（Points），強調拉取式（Queue-based），優先處理佇列最頂端（Top Priority）的高優先級任務。



## 1. Scrum 核心關鍵字與實務精神速

| 關鍵概念 / 儀式 | 英文原名 | 實務運作說明與核心意圖 |  
| :--- | :--- | :--- |  
| **衝刺期** | **Sprint** | 通常為 **2 週**的固定開發週期，目標是持續交付有價值的軟體功能。 |  
| **開發速度 / 點數** | **Velocity / Story Points** | 根據歷史 Sprint 累積的點數（Velocity），評估當前 Sprint 能承接的合理工作量。 |  
| **每日站會** | **Daily Stand-up** | 目的**不是向主管報告進度以保住工作**，而是 **同步資訊、排除阻礙（Unblock）** 並保持工作流暢。 |  
| **準備就緒定義** | **Definition of Ready (DoR)** | 團隊共識：當 Ticket 的需求、邊界條件足夠清晰，可以開始被開發時的標準。 |  
| **完成定義** | **Definition of Done (Dod)** | 團隊共識：當程式碼通過 Code Review、測試並部署後，視為真正完成的標準。 |  
| **衝刺計畫會議** | **Sprint Planning** | Sprint 開頭舉辦，評估團隊 Velocity，決定本期要拉入哪些 Ticket。 |  
| **衝刺回顧會議** | **Retrospective (Retro)** | Sprint 結尾舉辦，檢討「什麼做得好、什麼不好、如何改進」，持續優化團隊流程。 |



## 2. 現代 Scrum 標準工作流程圖 (The Scrum Cycle)

```text
[ Sprint Planning ] ➔ 確定目標與票券 (Based on Velocity)
         │
         ▼
[ 2-Week Sprint Cycle ]
   ├── Daily Stand-up (每日 15 分鐘：同步進度、排除 Blocking)
   ├── Definition of Ready (DoR) ➔ 檢查 Ticket 是否可開工
   └── Definition of Done (DoD)  ➔ 檢查程式碼是否達到交付標準
         │
         ▼
[ Sprint Retrospective ] ➔ 討論 What went well / What to improve
```

## 3. 回答策略 (Senior Response Strategy)  
回答敏捷開發相關問題時，建議採用 **「流程理解 ➔ 實務精神 ➔ 彈性思維」** 的三層架構：

### 1. 說明核心流程（Workflow）：  
簡述 2 週一個 Sprint，開頭有 Sprint Planning 規劃任務，每天有 Daily Stand-up 同步進度，結尾透過 Retro 進行團隊反思與優化。

### 2. 點出站會與團隊約定的真正價值（Why）：  
強調 Daily Stand-up 的本質是「互相幫忙解除封鎖（Unblock colleagues）與資訊透明」，絕非冰冷的進度控管（Status Update）。

### 3. 提到 Definition of Ready (DoR) 與 Definition of Done (DoD) 是團隊間的契約（Team Agreements），能減少開發摩擦。  
展現成熟的軟體工程思維（Senior Insight）：  
主動提到：「**我了解不同公司在落實 Scrum 時都會根據文化調整（Not strictly textbook），但關鍵始終在於透明度、團隊溝通與持續改進（Continuous Improvement）。**」

## FAQ
### Q1: 如果在 Sprint 進行到一半時，突然有緊急需求（Hotfix 或老闆臨時塞任務）進來，Scrum 團隊該如何處理？  
在嚴格的 Scrum 規範中，Sprint Goal 一旦設定就不應輕易變更。但在真實世界中：

- **評估優先級**：由 Product Owner (PO) 評估該緊急需求是否高於當前 Sprint 的任務。
- **等價交換（Trade-off）**：如果必須插入新任務， PO 必須從當前 Sprint Backlog 中抽走相同點數（Story Points）的次要任務，確保團隊負荷量不會過載。
- **紀錄與檢討**：在 Retro 會議中討論為什麼會有臨時插入的需求，優化未來的需求評估流程。

<blockquote class="my-6 p-4 bg-sky-50 dark:bg-sky-950/30 border-l-4 border-sky-500 rounded-r-md text-sky-900 dark:text-sky-200 blocknoted-fix">

### 學習 Agile 與 Scrum 的資源
- Scrum Guide 官方指南：scrumguides.org（最權威、最精簡的原始定義文件）。
- Atlassian Agile Coach：Atlassian（Jira 開發商）提供的敏捷教學文章，非常貼近業界開發實務。

</blockquote>