---
title: "Docker Compose 基本觀念：網路、連接埠與環境設定"
date: 2026-02-12
categories: [Docker, DevOps]
tags: [Docker-Compose, .NET, Networking]
---

在使用ASP.NET實作一個包含 **Gateway (YARP)** 與兩個 **ASP.NET Core Web App** 的微服務架構時，我們深入了解了 Docker Compose 的運作機制，在這邊記錄下來。以下是本專案的核心配置檔案：

### 🛠️ 專案配置範本

#### **[docker-compose.yml](docker-compose.yml) (主設定檔)**
這是定義生產基準與服務依賴的核心：
```yaml
services:
  gateway:
    image: ${DOCKER_REGISTRY-}gateway
    build:
      context: ./src/Gateway
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    depends_on:
      - app1
      - app2
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ReverseProxy__Clusters__app1-cluster__Destinations__app1__Address=http://app1:5001
      - ReverseProxy__Clusters__app2-cluster__Destinations__app2__Address=http://app2:5001

  app1:
    image: ${DOCKER_REGISTRY-}app1
    build:
      context: ./src/App1.Web
      dockerfile: Dockerfile
    expose:
      - "5001"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_HTTP_PORTS=5001

  app2:
    image: ${DOCKER_REGISTRY-}app2
    build:
      context: ./src/App2.Web
      dockerfile: Dockerfile
    expose:
      - "5001"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ASPNETCORE_HTTP_PORTS=5001
```

#### **[docker-compose.override.yml](docker-compose.override.yml) (開發開發覆寫)**
針對開發環境進行連接埠映射與環境切換：
```yaml
services:
  gateway:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_HTTP_PORTS=8080
    ports:
      - "8080:8080"

  app1:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5001:5001"

  app2:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "5002:5001"
```

以下是針對上述配置所總結的關鍵知識點：

## 1. 設定檔的雙重奏：yml 與 override
*   **`docker-compose.yml`**：主說明書。定義了服務的基礎架構（Image 名稱、依賴關係 `depends_on`、生產環境變數）。
*   **`docker-compose.override.yml`**：開發補丁。專門處理「本機開發」的需求，例如將內部的 Port 映射到電腦的 `localhost`，或切換至 `Development` 模式。

## 2. 虛擬區域網路 (Virtual Bridge Network)
Docker Compose 啟動時會建立一個虛擬的區域網路。
*   **服務如同在交換機中 (Inside the Same LAN)**：
    *   在同一個 `docker-compose.yml` 裡的所有服務（Gateway, app1, app2），都像是插在同一台「虛擬交換機」上的成員。
*   **服務名稱即主機名稱**：容器之間不需要知道 IP，直接呼叫服務名稱（例如 `http://app1`）即可。
*   **內建 DNS**：Docker Engine 內部負責將「服務名稱」解析為該容器的虛擬 IP。
*   **隔離性**：容器內部是獨立系統，共享核心但環境隔離。

### 📌 範例：Gateway (Asp.Net YARP架構)如何呼叫 App1？
在 **Gateway 容器** 內部的程式設定（如 `appsettings.json` 或環境變數）：
```json
"Destinations": {
  "app1": { "Address": "http://app1:5001" } // 就像打分機一樣簡單
}
```
Docker 會在背後偷偷把 `http://app1:5001` 變換成 `http://172.18.0.3:5001`。

## 3. 連接埠 (Ports) 的層級觀念
這是最容易混淆的部分，可以分為三個層次：
*   **內部監聽 (Inside Container)**：程式真正跑的 Port（如 `5001`）。多個容器可以同時監聽相同的 Port 而不衝突。
    *   **💡 為什麼不衝突？**：因為每個容器在 Docker 網路中都有 **自己獨立的虛擬 IP**。這就像不同住戶（容器）都有「大門口 (5001)」，但因為住址（IP）不同，所以掛號信（請求）不會送錯。你可以想像 app1 是 `172.18.0.2:5001`，app2 是 `172.18.0.3:5001`，兩者完全隔離。
*   **對內宣告 (Expose)**：僅是標籤 (Metadata)，告訴 Docker 網路內的其他鄰居：「我準備好這個 Port 了」。
*   **對外對應 (Ports Mapping)**：將容器內部 Port 接到實體電腦 (Host) 的 Port（如 `8080:8080`）。**實體電腦的 Port 不能重複**。

### 📌 範例對照
```yaml
services:
  app1:
    environment:
      - ASPNETCORE_HTTP_PORTS=5001 # 1. 收件人(程式)決定監聽 5001
    expose:
      - "5001"                    # 2. 宣告內部鄰居可見 5001
    ports:
      - "8081:5001"               # 3. 外部路人請撥 8081，我轉接到內部 5001

  app2:
    environment:
      - ASPNETCORE_HTTP_PORTS=5001 # 1. 內部一樣可以用 5001，不衝突！
    ports:
      - "8082:5001"               # 3. 外部路人必須撥不同的 8082
```

## 4. 環境變數：連接埠設定
在設定檔中見到的 `ASPNETCORE_HTTP_PORTS` 是 .NET 框架專用的環境變數，用來啟動時設定站台要監聽哪個Port，不同的程式框架有不同的帶入方式，這邊以.Net為例。

*   **取得Port設定參數**：ASP.NET Core 程式在啟動時會主動檢查這個環境變數，取得設定參數。
*   **啟動行為**：一旦發現數值（如 `5001`），.NET 框架內部的伺服器 (Kestrel) 就會自動使用這個Port啟動。
*   **優勢**：這讓開發者能在不重新編譯程式的情況下，透過 Docker 靈活地更換服務的內部監聽埠號。

## 5. 最佳實踐：映像檔版本管理
在實際生產環境中，「預設的Image Tag是latest」。若不指定版本號，你永遠不知道生產環境跑的是哪一天的程式碼。關於這點Docker有支援可以透過 **環境變數注入**的方式變更，這樣我們就可以讓同一個 Compose 檔案支援不同的發布版本。

### 📌 設定檔寫法
在 `docker-compose.yml` 中使用變數佔位符：
```yaml
services:
  gateway:
    # ${變數名稱:-預設值}
    image: ${DOCKER_REGISTRY-}gateway:${TAG:-latest}
```

### 📌 執行指令範例

#### 方式 A：透過 PowerShell 環境變數（推薦 CI/CD 與手動測試使用）
最直覺的方式就是在執行指令的前面直接串聯變數設定，這會告訴 Docker Compose 暫時使用這個標籤：

```powershell
# 一行指令搞定：設定 TAG 並啟動 Build
$env:TAG="v1.2.3"; docker-compose build

# 或是直接啟動整套系統並指定版本
$env:TAG="v1.2.3"; docker-compose up -d
```
Docker 會在執行期間自動將 `${TAG}` 替換為 `v1.2.3`。

### 💡 運作邏輯
當 Docker 看到 `image: gateway:${TAG:-latest}` 時，它的解析邏輯如下：
1. 「我有看到 `$env:TAG` 這個變數嗎？」
2. **有** → 使用變數值（結果：`gateway:v1.2.3`）。
3. **沒有** → 使用冒號後面的預設值（結果：`gateway:latest`）。

這讓同一份 `docker-compose.yml` 可以同時兼顧「開發環境（沒設定時跑最新版）」與「正式發佈（指定特定版本）」。

#### 方式 B：透過 `.env` 檔案（推薦開發環境使用）
在專案根目錄建立一個 `.env` 檔案：
```text
TAG=v1.0-dev
DOCKER_REGISTRY=my-local-registry.com/
```
執行 `docker-compose up` 時，Docker 會自動讀取並將 Image 標記為 `my-local-registry.com/gateway:v1.0-dev`。

### 為什麼這很重要？
1. **可追蹤性**：發生錯誤時，你能精確知道是哪個版本在出錯。
2. **快速回滾 (Rollback)**：若新版有問題，只需要把 `TAG` 改回舊版號並重啟，不需要重新下載或重新 Build。
3. **避免快取污染**：使用 `latest` 可能會導致某些機器抓到舊的快取，而指定版本號能確保所有機器都抓到同一份程式。

## 6. 參考資料
*   **Docker 官方文件**
    *   [Docker Compose Overview](https://docs.docker.com/compose/): 了解 Compose 的基本指令與生命週期。
    *   [Networking in Compose](https://docs.docker.com/compose/networking/): 深入了解容器間自動發現與 DNS 的運作原理。
*   **針對 .NET 開發者**
    *   [Containerize an ASP.NET Core app](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/visual-studio-tools-for-docker): 關於 Visual Studio 工具與 Docker 整合的詳細介紹。
    *   [ASP.NET Core Environment Variables](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/#environment-variables): 了解 .NET 如何解析多種來源的配置。

---