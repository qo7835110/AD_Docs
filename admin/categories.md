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
- **權限要求:** `admin.permission:categories,create`
- **Payload:**
  - `name` (required, string, unique:categories)
  - `description` (nullable, string)

## [GET] `/api/admin/categories/{id}`
取得分類詳細內容。
- **權限要求:** `admin.permission:categories,view`

## [PUT] `/api/admin/categories/{id}`
更新分類名稱與描述內容。
- **權限要求:** `admin.permission:categories,update`

## [DELETE] `/api/admin/categories/{id}`
軟刪除該分類。
- **權限要求:** `admin.permission:categories,delete`
> **邏輯警告:** 分類被刪除後，旗下依賴的 `CategoryPermission` 或 `AdPlans` 有可能進入失效或無法查詢的狀態，前端應提供警示對話框。

## [POST] `/api/admin/categories/{id}/restore`
將誤刪或暫停的分類恢復為活耀狀態。
- **權限要求:** `admin.permission:categories,update`
