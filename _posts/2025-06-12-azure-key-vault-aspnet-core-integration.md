---
layout: post
title: "Azure Key Vault 在 ASP.NET Core 中的完整使用指南"
date: 2025-06-12T11:31:00+08:00
categories: [Azure, ASP.NET Core, Security]
tags: [azure, key-vault, asp.net-core, security, secret-management, defaultazurecredential, azure-identity]
---

Azure Key Vault 是 Azure 提供的一個安全存放 Secret 的服務，由於是另外提供的服務，在 ASP.NET Core 上使用時需要做額外設定，在這邊簡單介紹一下如何能快速使用 Key Vault 的服務。

## 開發時使用 Key Vault

在開發時要使用 Key Vault，你需要有 Azure AD 帳號需透過帳號的權限去取得 Key Vault，使用前要先到 Azure Key Vault 後台的存取原則選項中設定，把帳號的存取權限設定好，接著在開發時就可以配合套件 Azure.Identity 提供的 `DefaultAzureCredential` Class 來取得使用權限，當然在開發環境是需要告知到底是使用哪個 Azure 帳號做存取的，這部分可以使用開發工具預設的方案，以 Visual Studio 2022 為例，可以至 Tools → Options → Azure Service Authentication 中設定 Azure 帳號，這樣 `DefaultAzureCredential` 就會使用 Azure Service Authentication 的權限，另外根據開發工具的不同你也可以使用 `VisualStudioCredential`、`VisualStudioCodeCredential` 這類憑證去做調整。除了使用開發工具提供的權限外也可以直接使用帳號密碼來取得權限，如 `UsernamePasswordCredential`，也可以達到同樣的效果。

### 設置 Azure Service Authentication

![Azure Service Authentication 設定](/assets/images/azure/key-vault/azure-service-authentication-setup.png)

### 執行範例

```csharp
//.net core 6 Program
builder.Configuration.AddAzureKeyVault(
    new Uri("保存庫 URI"),
    new DefaultAzureCredential());
```

![Key Vault URI 範例](/assets/images/azure/key-vault/key-vault-uri-example.png)

## 部署於 Azure 站台使用 Key Vault

如果專案是要部署在 Azure 上，使用上也非常方便，一樣只要在 Azure Key Vault 後台的存取原則選項上，把執行主體設定好這樣在存取時就可以透過 `DefaultAzureCredential` 取得存取權限，不需要額外設定。

## 其他進階用法

### 1. 使用 SecretClient 動態讀取

如果不想在初始設定時就將 Key Vault 讀取進來，你可以另外在你想執行的地方透過 `SecretClient` 讀取，執行方法如下：

```csharp
SecretClientOptions options = new SecretClientOptions()
{
    Retry = {
        Delay = TimeSpan.FromSeconds(2),
        MaxDelay = TimeSpan.FromSeconds(16),
        MaxRetries = 5,
        Mode = RetryMode.Exponential
    }
};
var client = new SecretClient(new Uri("保存庫 URI"), new DefaultAzureCredential(), options);
KeyVaultSecret secret = client.GetSecret("要取得的Secret");
```

### 2. 階層配置設定

如果資料要有階層的話在 Azure Key Vault 上設定需要改為使用 `--`，如果要儲存為陣列則使用數字來表示：

![階層配置範例](/assets/images/azure/key-vault/hierarchical-configuration-example.png)

### 3. 多組織帳號權限設定

另外補充，Azure 帳號權限需要額外注意一點，當你是存在於多個組織時，你必須指定你是使用哪個組織的權限來做存取：

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri("保存庫 URI"),
    new DefaultAzureCredential(new DefaultAzureCredentialOptions() { TenantId = "指定組織的TenantId" }));
```

## 總結

Azure Key Vault 與 ASP.NET Core 的整合非常完善：

1. **開發環境**：透過 Visual Studio 的 Azure Service Authentication 設定
2. **生產環境**：使用 Azure 管理身分識別自動授權
3. **彈性使用**：支援動態讀取和階層配置
4. **多租戶支援**：可指定特定組織的 TenantId

使用 `DefaultAzureCredential` 可以讓我們在不同環境下無縫切換認證方式，大大簡化了 Key Vault 的使用複雜度。

## 參考資料

- [Microsoft Learn - ASP.NET Core 中的 Azure Key Vault 配置提供者](https://learn.microsoft.com/zh-tw/aspnet/core/security/key-vault-configuration?view=aspnetcore-6.0)
- [Microsoft Learn - Azure Key Vault 節流指導方針](https://learn.microsoft.com/zh-tw/azure/key-vault/general/overview-throttling)
