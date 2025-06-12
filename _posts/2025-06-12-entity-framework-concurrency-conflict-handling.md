---
layout: post
title: "Entity Framework 處理資料競爭衝突的完整解決方案"
date: 2025-06-12T11:26:00+08:00
categories: [Entity Framework, .NET Core, Database]
tags: [entity-framework, concurrency, timestamp, concurrency-check, race-condition, dbupdateconcurrencyexception]
---

使用 Entity Framework 在更新資料時，很多情況下都會是非同步作業，這時很有可能發生因為存取資料的時間差，造成資料寫入時使用到舊的資料，使得最終結果與預期不符的情況。而 Entity Framework 是 ORM 的工具，不太會直接使用 SQL 指令來更新資料，因此在處理資料衝突上，就需要另外特別處理。

## 處理作法

因為會發生問題的原因是 Update 資料時，在計算資料上因為非同步原因，造成使用到舊的資料做運算，因此計算完的結果不符預期。在解決這個問題的處理方法上，我們可以在 Update 資料時先驗證系統上的資料是否與資料庫的相同，如果相同在繼續作業，如果不同則表示資料有落差，再另外處理，在 Entity Framework 可以透過 Timestamp 或 ConcurrencyCheck 配合 DbUpdateConcurrencyException 來處理，透過 Timestamp 或 ConcurrencyCheck 驗證資料正確性，如果有問題則透過 DbUpdateConcurrencyException 例外處理來修改資料。

### 1. 透過 Timestamp 或 ConcurrencyCheck 來驗證資料是否與 DB 相同

#### Timestamp
- **Timestamp 是一種資料庫欄位型別**
- **Timestamp 需要配合 DB 使用，需要 DB 有支援才可以使用**
- **Timestamp 是用來追蹤整筆資料是否有被更新的一個 Id，如果有更新就會自動累加**
- **Entity Framework 本身就有支援 Timestamp**，因此不用額外處理，當 Update 資料時會自動比對系統上的 Timestamp 與 DB 的 Timestamp，如果不同就會發生 DbUpdateConcurrencyException

![Timestamp 競爭條件示例 1](/assets/images/entity-framework/timestamp-race-condition-1.png)

![Timestamp 競爭條件示例 2](/assets/images/entity-framework/timestamp-race-condition-2.png)

#### ConcurrencyCheck
- **ConcurrencyCheck 是一種屬性**，你可以使用他來標記某個欄位是需要比對資料是否有被變更
- **當你標記好某個欄位後**，當執行 Update 時 Entity Framework 會自動比對系統上的資料與 DB 上的資料，如果不同就會發生 DbUpdateConcurrencyException

![ConcurrencyCheck 範例](/assets/images/entity-framework/concurrency-check-example.png)

### 2. 透過 DbUpdateConcurrencyException 處理資料競爭衝突

- **DbUpdateConcurrencyException 會回傳 EntityEntry**，這裡面會有發生衝突的資料 List，接著再根據發生衝突的資料，修改要更新的資料內容即可
- **另外有一點要注意**，Entity Framework 用來與 DB 比對資料的數據是放在 OriginalValues 中，這是需要自行更新的，記得更新 OriginalValues 與 DB 同步否則會再次發生衝突

```csharp
//因為可能會多次發生資料衝突因此使用while 迴圈處理
var saved = false;
while (!saved)
{
    try
    {
        // Attempt to save changes to the database
        _testDbContext.SaveChanges();
        saved = true;
    }
    catch (DbUpdateConcurrencyException ex)
    {
        foreach (var entry in ex.Entries)
        {
            //可以辨識資料衝突的型態，Update與Deleted都會發生
            if (entry.State == EntityState.Deleted)
            {
                entry.Reload();
                continue;
            }

            if (entry.Entity is TableTest)
            {
                //系統目前取得的DB數據
                var originalValues = entry.OriginalValues;
                //在系統中計算處理過的數據
                var proposedValues = entry.CurrentValues;
                //再一次讀取DB的數據
                var databaseValues = entry.GetDatabaseValues();

                foreach (var property in proposedValues.Properties)
                {
                    var originalValue = originalValues[property];
                    var proposedValue = proposedValues[property];
                    var databaseValue = databaseValues[property];

                    // TODO: decide which value should be written to database
                    // proposedValues[property] = <value to be saved>;
                    //
                    if (property.Name.Equals(nameof(TableTest.Value1)))
                    {
                        var test = $"Database:{databaseValue} Update:{proposedValue} Original:{originalValue}";
                        proposedValues[property] = test;
                    }
                    else
                    {
                        proposedValues[property] = databaseValue;
                    }
                }

                // Refresh original values to bypass next concurrency check
                entry.OriginalValues.SetValues(databaseValues);
            }
            else
            {
                throw new NotSupportedException(
                    "Don't know how to handle concurrency conflicts for "
                    + entry.Metadata.Name);
            }
        }
    }
}
```

## 總結

Entity Framework 提供了完整的並行控制機制：

1. **使用 Timestamp 或 ConcurrencyCheck 屬性**來標記需要檢查的欄位
2. **透過 DbUpdateConcurrencyException 捕獲衝突**並進行適當處理
3. **更新 OriginalValues**以確保後續操作的正確性
4. **使用迴圈機制**處理可能的多次衝突

這樣的設計可以有效避免資料競爭問題，確保資料的一致性和正確性。

## 參考資料

- [Microsoft Docs - Entity Framework Core 並行衝突處理](https://docs.microsoft.com/zh-tw/ef/core/saving/concurrency)
- [Entity Framework Extensions - DbUpdateConcurrencyException](https://entityframework-extensions.net/dbupdateconcurrency-exception)
