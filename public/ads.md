# Public API - 廣告前台瀏覽

> 提供給一般大眾（Guest）與會員瀏覽正式上架的廣告。

## [GET] `/api/public/ads`
瀏覽目前已上架且狀態為「審核通過 (`status: active`)」的廣告列表。
- **Query Params:**
  - `category_id` (integer, optional) - 以分類過濾廣告。
  - `keyword` (string, optional) - 針對標題或內容進行模糊搜尋。
  - `page` (integer, optional)
  - `per_page` (integer, optional) - 預設通常為 15 或 20。

## [GET] `/api/public/ads/{id}`
取得單一上架廣告的詳細展示內容。
- **Response:**
  - 包含廣告封面圖片 (Image URL)
  - 廣告的起訖時間與詳細內文。
  - 關聯之分類與方案名稱。
