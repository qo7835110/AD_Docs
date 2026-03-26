# User API - 廣告分類
**權限:** `auth:api` (需攜帶 Bearer Token)

## [GET] `/api/categories`
取得所有啟用狀態的廣告分類。
> **重點商務邏輯提示：** 
> 系統在回傳分類清單時，會自動比對當前使用者的 `CategoryPermission`。若該分類對該使用者設有限制，且使用者並無白名單權限，前端可藉此標示「鎖定/無權購買」。

## [GET] `/api/categories/{id}`
取得單一分類的詳細資料。
