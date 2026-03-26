# User API - 會員廣告管理
**權限:** `auth:api` (需攜帶 Bearer Token)

## [GET] `/api/ads`
取得會員自己建立的所有廣告清單（包含草稿、審核中、上架中、下架等各種狀態）。

## [POST] `/api/ads`
建立新的「廣告草稿」。
- **Payload Validation:**
  - `ad_plan_id` (integer, required, exists:ad_plans,id)
  - `title` (string, required, max:255)
  - `content` (string, required)
> **重點商務邏輯：** 建立廣告前，Server 會利用 `CategoryPermission` 檢查該會員是否具備該廣告方案（所屬分類）的購買權限。若不具備，將拋出 `403 Forbidden`。新建立的廣告預設狀態為草稿。

## [GET] `/api/ads/{id}`
取得會員擁有的單一廣告詳細內容（含審核回饋留言）。

## [PUT] `/api/ads/{id}`
更新廣告內容。
> **業務邏輯限制：** 僅限狀態為「草稿 (`draft`)」或「被退回 (`rejected`)」的廣告可以進行更新。審核中或已上架的廣告不得隨意竄改。

## [DELETE] `/api/ads/{id}`
刪除尚未發布的廣告草稿。

## [POST] `/api/ads/{id}/image`
上傳廣告附屬圖片（如 Banner）。
- **Payload:** `image` (file, required, image/jpeg/png, max:2048)

## [POST] `/api/ads/{id}/submit`
將草稿提交送審。
- **邏輯：** 廣告的屬性將自 `draft` 切換至 `pending`，並鎖定更新操作，等待管理員審核。
