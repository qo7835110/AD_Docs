# User API - 訂單與付款
**權限:** `auth:api` (需攜帶 Bearer Token)

## [GET] `/api/orders`
取得該會員名下的所有訂單清單（含付款狀態、金額與關聯之廣告方案）。

## [POST] `/api/orders`
替已建立之「現有廣告」與特定的「計費選項」產生訂單。
- **Payload Validation:**
  - `plan_option_id` (integer, required, exists:plan_options,id)
  - `ad_id` (integer, required, exists:ads,id) - 檢查擁有權與狀態

## [POST] `/api/orders/with-ads`
便捷 API：於結帳流程同步建立「廣告草稿」並直接產生「訂單」。
- **Payload Validation:**
  - `plan_option_id` (integer, required)
  - `category_id` (integer, required)
  - `ad_title` (string, required)
  - `ad_content` (string, required)
> 邏輯同建立廣告，會強制驗證使用者的 Category Permission。

## [GET] `/api/orders/{orderNumber}`
透過訂單獨立編號 (`order_number`) 取得訂單詳細資料與金流明細。

## [POST] `/api/orders/{orderNumber}/cancel`
主動取消尚未付款的訂單。

## [POST] `/api/orders/{orderNumber}/pay`
執行付款（觸發金流或內部虛擬付款）。
- **Payload Validation:**
  - `payment_method` (string, required, in:credit_card,transfer,balance)
> **業務邏輯關聯：** 付款成功後，關聯廣告若尚在草稿狀態，可自動被標記準備送審或直接送入審核排程。

## [POST] `/api/orders/{orderNumber}/refund`
申請退款（若業務允許）。

## [GET] `/api/orders/{orderNumber}/payments`
取得單一訂單的所有歷史付款、退款與失敗紀錄。
