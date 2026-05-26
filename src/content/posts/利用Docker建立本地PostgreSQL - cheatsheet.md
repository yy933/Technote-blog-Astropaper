---
title: "利用Docker建立本地PostgreSQL - cheatsheet"
pubDatetime: 2026-05-26T03:01:47.013Z
tags: ["database","cheatsheet","PostgreSQL","Docker"]
description: " Table of contents Prerequisite 安裝Docker: [Docker Desktop](h..."
---

## Table of contents


## Prerequisite
* 安裝Docker: [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* 視需求安裝[VSCode Docker Extension](https://code.visualstudio.com/docs/containers/overview)(Docker的GUI)、[pgAdmin](https://www.pgadmin.org/)(PostgreSQL的GUI)
> 小提醒：Docker 在 Windows 有 WSL2 需求，確認 BIOS 內已開啟虛擬化 (Virtualization)，確定 Docker Desktop 正常啟動。
 
## 1. 確認 Docker 可用
開啟專案資料夾，在terminal輸入:
```bash
docker --version
```
會顯示版本，例如：
```bash
Docker version 24.0.5, build xxx
```
## 2. 建立 Docker network（Optional）
```bash
docker network create nextauth-net
```
* 這個步驟是未來如果有多個 container（例如 Redis、PostgreSQL）可以互相通訊。
* 如果目前只有一個 PostgreSQL，可以先跳過這步。
確認 network：
```bash
docker network ls
```
可以看到所有network詳細資料:
```bash
NETWORK ID     NAME           DRIVER    SCOPE
72a4317bd580   bridge         bridge    local
abeef44cf44d   host           host      local
f33149638d94   nextauth-net   bridge    local
a34b46160e86   none           null      local
```

## 3. 啟動 PostgreSQL container
```bash
docker run -d \
  --name nextauth-postgres \
  --network nextauth-net \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=admin \
  -e POSTGRES_DB=nextauthdb \
  -p 5432:5432 \
  -v nextauth-data:/var/lib/postgresql/data \
  postgres:15
```
解釋：

* `-d` → 後台執行
* `--name nextauth-postgres` → container 名稱
* `--network nextauth-net` → 所屬 network
* `-e POSTGRES_USER=admin` → DB 使用者
* `-e POSTGRES_PASSWORD=admin` → DB 密碼
* `-e POSTGRES_DB=nextauthdb` → 預設資料庫
* `-p 5432:5432` → 映射本地 5432 port 到 container 5432（PostgreSQL 預設 port）
* `-v nextauth-data:/var/lib/postgresql/data`→ 加上 volume，避免 container 刪掉後資料消失
* `postgres:15` → 使用 PostgreSQL 15 映像檔

確認 container 正在跑：
```bash
docker ps
```
會看到名稱 nextauth-postgres 和對應的 port，例如:
```bash
CONTAINER ID   IMAGE         COMMAND                   CREATED          STATUS          PORTS                                         NAMES
a6482392b756   postgres:15   "docker-entrypoint.s…"   37 seconds ago   Up 36 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   nextauth-postgres
```
## 4. 連線到 PostgreSQL
1. 用 VSCode extension 連線：
* 安裝 PostgreSQL extension（如 PostgreSQL 或 PgAdmin）
* Host: localhost
* Port: 5432
* User: admin
* Password: admin
* Database: nextauthdb

2. 或是用command:
```bash
docker exec -it nextauth-postgres psql -U admin -d nextauthdb
```
就可以看到 PostgreSQL 提示符` nextauthdb=>`

## 5. 停止 / 重啟 container
```bash
docker stop nextauth-postgres
docker start nextauth-postgres
```
刪除 container：
```bash
docker rm -f nextauth-postgres
```
刪除 network：
```
docker network rm nextauth-net
```

## pgAdmin連線到Docker PostgreSQL

### 1. 建立Server
開啟pgAdmin，左側的Servers(也可以Create新的Server Group)按右鍵 → Register → Server

### 2. 連線設定
接著會看到如下畫面:
![螢幕擷取畫面 2025-09-09 233710](https://hackmd.io/_uploads/HkH5u6Tcgx.png)
* General Tab:
  - Name：隨意，例如 nextauth-db
* Connection Tab:
  - Host name/address：localhost

  > 如果用 Docker network，並在同一台機器上執行 pgAdmin 就用 localhost
  > 如果在另一台機器，就要用 Docker host IP
  - Port：5432（ docker run 時設定的 port）
  - Maintenance database：nextauthdb（在 container 建的資料庫）
  - Username：admin（設定的 POSTGRES_USER）
  - Password：admin（設定的 POSTGRES_PASSWORD）
> 如果 pgAdmin 無法連線，先確認 Docker 是否允許外部連線，或 port 5432 是否被其他程式（例如本機 PostgreSQL）佔用。

### 3. 儲存並連線
* pgAdmin 會自動顯示資料庫內容，可以看到 tables、schemas 等。
* 在執行 Prisma migration 後（npx prisma migrate dev），表格也會顯示在 pgAdmin 上。
* 快速驗證 DB 是否建立成功:
```sql
\l   -- 列出所有資料庫
\dt  -- 列出目前資料庫的所有 tables
```

## 💡 其他小技巧
* 如果想看 container 裡 PostgreSQL log：
```bash
docker logs nextauth-postgres
```
* 如果想直接進入 psql：
```
docker exec -it nextauth-postgres psql -U admin -d nextauthdb
```

建立完資料庫之後，就可以利用Prisma在JS/TS中操作資料庫了!
[下一篇](https://hackmd.io/A4memsR6Tcy5KPAwlz053g)來詳細說明Prisma schema的設計與建立table，可以將使用者資料寫入，方便後續操作登入驗證。
::: success
:crescent_moon: 　本站內容僅為個人學習記錄，如有錯誤歡迎留言告知、交流討論！
:::