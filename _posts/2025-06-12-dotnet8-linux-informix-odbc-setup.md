---
layout: post
title: "使用.NET 8 在 Linux 環境上透過 ODBC 連線 Informix 資料庫設置指南"
date: 2025-06-12T14:00:00+08:00
categories: [.NET, Linux, Database, Docker]
tags: [dotnet8, linux, informix, odbc, docker, database-connection, unixodbc, csdk]
---

這個文章說明 .NET 8 在 Linux 環境上執行，透過 ODBC 的方式連線至 Informix 資料庫。由於對 Linux 環境並不是這麼熟悉，因此這個專案的方案提供一個方向能夠達到基本的連線功能，文件內容會說明系統基本的配置與執行方法。

## 所需套件

### Linux 套件

- unixodbc
- libncurses5
- openjdk-17-jdk
- libelf1
- locales

### Informix SDK

- ibm.csdk.4.50.FC11W1.LNX (此版本為測試當下最新版本，可以到 IBM 官網下載)

## 安裝方式

我是使用 Docker 進行配置，配置的 Dockerfile 如下，另外一些設定檔如 `odbc.ini` 我會先寫好放到專案中在 COPY 到 container 中。另外我 Informix 是直接使用 GitHub 上的 Docker image 去配置，配置時我沒另外修改都用預設值，只有另外自己在建立一個 database: `testdev`。

### Dockerfile

```dockerfile
# See https://aka.ms/customizecontainer to learn how to customize your debug container and how Visual Studio uses this Dockerfile to build your images for faster debugging.

# This stage is used when running from VS in fast mode (Default for Debug configuration)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base

ENV LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:/usr/local/ibm/gsk8_64/lib64
ENV INFORMIXDIR=/opt/IBM/Informix_Client-SDK
ENV LD_LIBRARY_PATH=$INFORMIXDIR/lib/esql:${LD_LIBRARY_PATH:-}
ENV ODBCINI=/etc/odbc.ini

COPY WebODBCTest/InformixLib/ibm.csdk.4.50.FC11W1.LNX.tar /tmp/in4file/

# 安裝INFORMIX DRIVER
RUN apt-get update \
&& apt-get install -y unixodbc-dev libncurses5 libelf1 locales openjdk-17-jdk \
&& tar xvf /tmp/in4file/ibm.csdk.4.50.FC11W1.LNX.tar -C /tmp/in4file \
&& tar xvf /tmp/in4file/ibm.csdk.4.50.FC11W1.LNX.tar -C /tmp/in4file \
&& /tmp/in4file/installclientsdk -i silent \
	-DLICENSE_ACCEPTED=TRUE \
	-DUSER_INSTALL_DIR=/opt/IBM/Informix_Client-SDK \
	-DCHOSEN_FEATURE_LIST=SDK,SDK-ODBC,DBA-DBA,SDK-NETCORE,GLS \
	-DCHOSEN_INSTALL_FEATURE_LIST=SDK,SDK-ODBC,DBA-DBA,SDK-NETCORE,GLS \
	-DCHOSEN_INSTALL_SET=Custom \
&& rm -rf /tmp/in4file \
&& apt autoremove -y openjdk-17-jdk \
&& rm -rf /var/lib/apt/lists/*

# 測試ISQL時需要設定  
RUN echo "informix onsoctcp ifx 9088" > $INFORMIXDIR/etc/sqlhosts

# 配置odbc設定
COPY WebODBCTest/OdbcFile/odbc.ini /tmp/odbc.ini
COPY WebODBCTest/OdbcFile/odbcinst.ini /tmp/odbcinst.ini
RUN cat /tmp/odbc.ini >> /etc/odbc.ini && rm -f /tmp/odbc.ini
RUN cat /tmp/odbcinst.ini >> /etc/odbcinst.ini && rm -f /tmp/odbcinst.ini

# 配置編碼包，如果寫在安裝locales前可以不用執行locale-gen
COPY WebODBCTest/OdbcFile/locale.gen /etc/locale.gen
RUN locale-gen

USER app
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

# This stage is used to build the service project
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src
COPY ["WebODBCTest/WebODBCTest.csproj", "WebODBCTest/"]
RUN dotnet restore "./WebODBCTest/WebODBCTest.csproj"
COPY . .
WORKDIR "/src/WebODBCTest"
RUN dotnet build "./WebODBCTest.csproj" -c $BUILD_CONFIGURATION -o /app/build

# This stage is used to publish the service project to be copied to the final stage
FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./WebODBCTest.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false

# This stage is used in production or when running from VS in regular mode (Default when not using the Debug configuration)
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WebODBCTest.dll"]
```

### 設定檔內容

#### odbc.ini

```ini
[ODBC]
UNICODE=UCS-2LE

[INFORMIXTESTDB]
Description=Informix ODBC connection
Driver=IBM INFORMIX ODBC DRIVER
Database=testdev
ServerName=informix
Host=ifx
Service=9088
Protocol=onsoctcp
CLIENT_LOCALE=en_US.utf8
DB_LOCALE=en_US.819
```

#### odbcinst.ini

```ini
[IBM INFORMIX ODBC DRIVER]
Description=IBM Informix ODBC Driver
Driver=/opt/IBM/Informix_Client-SDK/lib/cli/iclit09b.so
```

#### locale.gen

```text
en_US.UTF-8 UTF-8
```

## 問題排除

在設置時主要會遇到的問題是套件安裝相關套件執行失敗與 unixODBC 配置設定缺少，以及語系套件問題而無法正確連線資料庫。

### 1. ibm.csdk.4.50.FC11W1.LNX 安裝

#### libncurses5
首先要注意的是不可以使用 libncurses6 的，libncurses6 的內容不相容所以安裝時會有項目安裝失敗因此須注意是否套件是使用 libncurses5。

#### 環境變數
這邊有一項特別的問題就是在安裝 CSDK 時，會需要參考部分安裝的項目，如果沒有在環境變數上事先設定會造成關聯不到必要的內容而執行失敗。

```bash
LD_LIBRARY_PATH=/usr/lib/x86_64-linux-gnu:/usr/local/ibm/gsk8_64/lib64
```

#### iclit09b.so
CSDK 內 ODBC 要執行的檔案 iclit09b.so，如果沒有這是環境變數參考的話，會有相依套件沒有關連到造成執行錯誤因此須注意設置環境變數。

```bash
INFORMIXDIR=/opt/IBM/Informix_Client-SDK
LD_LIBRARY_PATH=$INFORMIXDIR/lib/esql
```

#### 套件配置是否完整
可以透過 cmd 指令 ldd 查看套件的相依性：

```bash
ldd /opt/IBM/Informix_Client-SDK/lib/cli/iclit09b.so
```

### 2. ODBC 配置

ODBC 配置會發生的主要問題是 ODBC API 編碼問題，由於 .NET 8 中 ODBC 套件執行 ODBC API 會使用 unicode 的相關方法執行，但是 unixODBC 執行時沒有透過 unicode 的編碼執行，造成會有編碼錯誤，我這邊直接強制使用 UCS-2LE，關於 UCS-2LE 目前我查到 unixODBC 並沒有支援 UTF-8 與 16 所以使用 UCS-2LE。

```ini
# odbc.ini檔案內設置
[ODBC]
UNICODE=UCS-2LE
```

### 3. locale 設置

Linux 預設的語系為 C.utf8，但在執行 Informix 連線時發現 CLIENT_LOCALE 設為 C.utf8 是無法解析的，因此額外安裝 en_US.utf8 的套件，另外 DB_LOCALE 我設置為 en_US.819，這是因為我查看 Informix 的資料庫是設置為 en_US.819 所以這樣設置，如果你要連線的資料庫並不是 en_US.819 請自行更改，另外 en_US.819 本地不安裝也是可以的，這是 DB 環境需要使用的編碼格式。

## DEBUG 相關方法與設置

### 1. Docker network 設置
提供資源都建立在 Docker 上時使用。如果測試環境的所有資源都放在 Docker 上，記得配置 network 讓各個 container 可以互相連線：

```bash
docker network create mynetwork
docker network connect mynetwork fix
docker network connect mynetwork WebODBCTest
docker network connect mynetwork sql1
```

### 2. 使用 root 權限登入 container

```bash
docker exec -u 0 -it WebODBCTest sh
```

### 3. ping 站台 - 測試連線
可安裝 iputils-ping 套件，透過 ping 指令測試連線：

```bash
apt-get install -y iputils-ping
```

### 4. isql 與 iusql 在 cmd 上測試 SQL 連線

#### isql 測試
isql 配置需求較低，預設比較容易執行成功，因此可先透過 isql 測試連線，如果有成功表示 ODBC 套件運作正常。

```bash
# INFORMIXTESTDB為DSN配置參照odbc.ini
isql -v INFORMIXTESTDB informix in4mix
isql -v -k "DRIVER={IBM INFORMIX ODBC DRIVER};Server=informix;Database=testdev;Host=ifx;Service=9088;Protocol=onsoctcp;UID=informix;PWD=in4mix;CLIENT_LOCALE=en_US.utf8;DB_LOCALE=en_US.819;"
```

#### iusql 測試
iusql 為 isql 的 unicode 版本，因此 iusql 執行成功，那 .NET 8 內執行基本上也會成功，可透過這個指令確認是否有配置完成。

```bash
# INFORMIXTESTDB為DSN配置參照odbc.ini
iusql -v INFORMIXTESTDB informix in4mix
```

### 5. ODBC log
odbcinst.ini 可以設定 ODBC log 是否儲存，會記錄執行的檔案，可檢查執行那些檔案時發生錯誤。

```ini
[ODBC]
Trace = Yes
TraceFile = /tmp/odbc_trace.log
```

## 總結

在 Linux 環境下配置 .NET 8 連接 Informix 資料庫需要注意以下幾個關鍵點：

1. **套件相依性**：必須使用 libncurses5 而非 libncurses6
2. **環境變數設定**：正確設定 LD_LIBRARY_PATH 和 INFORMIXDIR
3. **編碼配置**：使用 UCS-2LE 解決 Unicode 編碼問題
4. **語系設定**：配置 en_US.utf8 語系包
5. **測試驗證**：使用 isql 和 iusql 驗證連線設定

透過 Docker 容器化部署可以更好地管理這些複雜的依賴關係和環境配置。

## 參考資料

- [IBM Community - CSDK installation on Docker Linux container (.NET 8)](https://community.ibm.com/community/user/datamanagement/discussion/csdk-installation-on-docker-linux-container-net-8#bmedc1d2f2-239b-42cd-a895-0190969cc607)
- [IBM Docs - Function calls for ODBC driver managers](https://www.ibm.com/docs/zh-tw/db2/11.1?topic=applications-function-calls-odbc-driver-managers)
- [IBM Docs - Locale file subdirectories](https://www.ibm.com/docs/en/informix-servers/14.10?topic=files-locale-file-subdirectories#ids_gug_257)
- [IBM Docs - Connecting to a database from Linux and UNIX systems by using Integration ODBC database extender](https://www.ibm.com/docs/zh-tw/app-connect/11.0.0?topic=eocd-connecting-database-from-linux-unix-systems-by-using-integration-odbc-database-extender)
- [GitHub - Informix Docker Hub README](https://github.com/informix/informix-dockerhub-readme/blob/master/14.10.FC1/informix-developer-database.md)
