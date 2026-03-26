# Admin API - 廣告分類管理 (Categories)
**身份驗證:** `auth:admin`   
**所需模組權限:** `categories`

> 用來建立與維護最高層級的維度—「廣告分類 (Category)」。後續所有方案與邏輯都依附於 Categories 下。

## [GET] `/api/admin/categories`
管理員檢視所有分類列隊。
- **權限要求:** `admin.permission:categories,view`
- **Response:** 會包含 `trashed=true` 的參數支援，得以看見被 Soft Delete (軟刪除) 的分類歷史。

## [POST] `/api/admin/categories`
建立全新的分類類目。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `name` | string | required, unique:categories | 是 | 全新分類名稱 |
| `description` | string | nullable | 否 | 分類詳細簡介說明 |

### Payload 範例 (JSON)
```json
{
  "name": "首頁黃金橫幅特區",
  "description": "只能用於網站首頁頂部的橫幅專區，具備最高曝光率。"
}
```

## [GET] `/api/admin/categories/{id}`
取得分類詳細內容。
- **權限要求:** `admin.permission:categories,view`

## [PUT] `/api/admin/categories/{id}`
更新分類名稱與描述內容。

### Payload 說明
與建立欄位相同，唯 `name` 驗證會排除當下的 ID (`unique:categories,name,{id}`) 以防重複阻擋。

### Payload 範例 (JSON)
```json
{
  "name": "首頁黃金橫幅特區(最新)",
  "description": "更新後的規則說明與涵蓋範圍"
}
```

## [DELETE] `/api/admin/categories/{id}`
軟刪除該分類。無 Payload。
- **權限要求:** `admin.permission:categories,delete`
> **邏輯警告:** 分類被刪除後，旗下依賴的 `CategoryPermission` 或 `AdPlans` 有可能進入失效或無法查詢的狀態，前端應提供警示對話框。

## [POST] `/api/admin/categories/{id}/restore`
將誤刪或暫停的分類恢復為活耀狀態。無 Payload。
- **權限要求:** `admin.permission:categories,update`
