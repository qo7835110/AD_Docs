# User API - 訂單與付款
**權限:** `auth:api` (需攜帶 Bearer Token)

## [GET] `/api/orders`
取得該會員名下的所有歷史訂單清單（含付款狀態）。

---

## [POST] `/api/orders`
替已建立之「預備用實體草稿」與特定的「計費選項」綁定為正式訂單。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `plan_option_id` | integer | exists:plan_options,id | 是 | 廣告方案內的確切計價子項 ID |
| `ad_id` | integer | exists:ads,id | 是 | 此訂單要綁定生效的草稿廣告 ID |

### Payload 範例 (JSON)
```json
{
  "plan_option_id": 12,
  "ad_id": 88
}
```

---

## [POST] `/api/orders/with-ads`
便捷 API：於結帳流程同步建立全新「廣告草稿」並直接產生「訂單」。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `plan_option_id` | integer | required | 是 | 計價子項 ID |
| `category_id` | integer | required | 是 | 指定該廣告欲投稿的分類（系統憑此攔截權限） |
| `ad_title` | string | required | 是 | 快速建立之廣告標題 |
| `ad_content` | string | required | 是 | 廣告詳細內文 |

### Payload 範例 (JSON)
```json
{
  "category_id": 3,
  "plan_option_id": 12,
  "ad_title": "快速曝光旗艦體驗專案",
  "ad_content": "首頁橫幅連續刊登三天！"
}
```

---

## [GET] `/api/orders/{orderNumber}`
透過訂單獨立編號 (`order_number`) 取得訂單詳細資料與金流明細。

---

## [POST] `/api/orders/{orderNumber}/cancel`
取消未付款的訂單。無 Payload 需求。

---

## [POST] `/api/orders/{orderNumber}/pay`
執行付款（觸發金流串接或內部虛擬付款放行）。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `payment_method` | string | in:credit_card,transfer,balance | 是 | 選擇要呼叫的付款管道 |

### Payload 範例 (JSON)
```json
{
  "payment_method": "credit_card"
}
```
> **業務邏輯：** 付款成功狀態改變後，關聯之草稿可被系統非同步佇列接管，推進上架。

---

## [POST] `/api/orders/{orderNumber}/refund`
申請退款作業，觸發營運端紀錄。

## [GET] `/api/orders/{orderNumber}/payments`
取得單一訂單的所有付款歷史（不論成功或失敗）。
