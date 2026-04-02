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

## [POST] `/api/public/ads/{id}/impression`
記錄指定廣告的一次曝光 (Impression)。
- **權限要求:** 公開層級 (Guest/User 皆可)
- **防刷機制:** 系統會根據使用者的 `IP Address` 搭配 60 秒的防刷時間窗口進行去重判定。
- **Response:** 成功記錄（`201 Created`），或即使觸發防刷判定仍會回傳 `201` 成功但不計入統計。

## [GET] `/api/public/ads/{id}/click`
記錄指定廣告的一次點擊 (Click)，並將使用者跳轉至廣告主定義的 URL。
- **權限要求:** 公開層級 (Guest/User 皆可)
- **防刷機制:** 同 IP 60 秒內的點擊僅記錄一次，但**不論是否去重**皆會觸發 302 跳轉。
- **Response:** `302 Found` (跳轉至廣告設定的 `link_url`)。
