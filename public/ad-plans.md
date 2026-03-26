# Public API - 廣告方案瀏覽

> 允許訪客與會員瀏覽現有啟用的分類與廣告方案。

## [GET] `/api/ad-plans`
公開瀏覽所有啟用 (`active`) 的廣告方案。支援分類過濾。
- **Query Params:**
  - `category_id` (integer, optional) - 篩選特定廣告分類。

## [GET] `/api/ad-plans/{id}`
取得單一廣告方案的詳細內容。
- **Response:**
  - 包含方案層級設定，並會 Eager Load 其底下所有的計費選項 (Plan Options)。

## [GET] `/api/ad-plans/{planId}/options`
取得特定廣告方案的所有計費選項列表。

## [GET] `/api/plan-options/{id}`
取得單一計費選項詳細資料（例如天數限制、價格與有效期限）。
