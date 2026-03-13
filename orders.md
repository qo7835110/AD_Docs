# 🛒 Orders — 訂單管理 API

> **Prefix：** `/api/orders`
> **Guard：** `api`（`users` 表）
> **認證：** 所有端點均需要 🔒 `Authorization: Bearer {member_token}`
> **訂單編號格式：** `ORD{YYYYMMDD}{8位隨機英數}` 例：`ORD20260312AB12CD34`

---

## 端點索引

| 方法   | 路徑                                 | 說明                   |
| ------ | ------------------------------------ | ---------------------- |
| `POST` | `/api/orders`                        | 建立訂單               |
| `POST` | `/api/orders/with-ads`               | 建立訂單並建立廣告草稿 |
| `GET`  | `/api/orders`                        | 取得訂單列表           |
| `GET`  | `/api/orders/{orderNumber}`          | 取得訂單詳情           |
| `POST` | `/api/orders/{orderNumber}/cancel`   | 取消訂單               |
| `POST` | `/api/orders/{orderNumber}/pay`      | 支付訂單               |
| `POST` | `/api/orders/{orderNumber}/refund`   | 申請退款               |
| `GET`  | `/api/orders/{orderNumber}/payments` | 取得付款記錄           |

---

## 狀態值說明

### 訂單狀態（`order_status`）

| 值           | 說明             |
| ------------ | ---------------- |
| `pending`    | 待處理（剛建立） |
| `processing` | 處理中           |
| `active`     | 已啟用（付款後） |
| `completed`  | 已完成           |
| `cancelled`  | 已取消           |
| `expired`    | 已過期           |

### 付款狀態（`payment_status`）

| 值               | 說明     |
| ---------------- | -------- |
| `unpaid`         | 未付款   |
| `paid`           | 已付款   |
| `partial_refund` | 部分退款 |
| `refunded`       | 全額退款 |
| `failed`         | 付款失敗 |

---

## POST `/api/orders` — 建立訂單 🔒

**Request Body（JSON）**

| 欄位               | 類型      | 必填 | 說明                                                |
| ------------------ | --------- | ---- | --------------------------------------------------- |
| `items`            | `array`   | ✅   | 訂單項目（至少 1 筆）                               |
| `items[].type`     | `string`  | ✅   | 項目類型：`plan`（廣告方案）或 `option`（方案選項） |
| `items[].id`       | `integer` | ✅   | 方案 ID 或選項 ID                                   |
| `items[].quantity` | `integer` | ❌   | 數量（min:1，預設 1）                               |
| `discount`         | `number`  | ❌   | 折扣金額（min:0，上限不超過小計）                   |
| `notes`            | `string`  | ❌   | 訂單備註（max:1000）                                |

> **折扣上限：** `discount` 若超過訂單小計（`subtotal`），系統會自動將折扣設為小計金額，確保 `total_amount` 不為負數。

**請求範例**

```json
{
    "items": [
        { "type": "plan", "id": 1, "quantity": 1 },
        { "type": "option", "id": 2, "quantity": 1 }
    ],
    "discount": 100,
    "notes": "希望盡快審核"
}
```

**回應範例（201）**

```json
{
    "success": true,
    "message": "訂單建立成功",
    "data": {
        "order": {
            "id": 1,
            "order_number": "ORD20260312AB12CD34",
            "order_status": "pending",
            "payment_status": "unpaid",
            "subtotal": 3998.0,
            "discount": 100.0,
            "total_amount": 3898.0,
            "notes": "希望盡快審核",
            "orderable": {
                "type": "user",
                "id": 1,
                "name": "王小明"
            },
            "items": [
                {
                    "id": 1,
                    "item_type": "AdPlan",
                    "item_id": 1,
                    "item_name": "基礎方案",
                    "quantity": 1,
                    "unit_price": 999.0,
                    "subtotal": 999.0,
                    "validity": {
                        "duration_days": 30,
                        "valid_start_date": null,
                        "valid_end_date": null
                    }
                }
            ],
            "payment_logs": [],
            "created_at": "2026-03-13 11:00:00",
            "updated_at": "2026-03-13 11:00:00"
        }
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                     |
| ------ | ------------------------ |
| `400`  | 業務錯誤（如商品不存在） |
| `401`  | 未授權                   |
| `422`  | 驗證失敗                 |

---

## POST `/api/orders/with-ads` — 建立訂單並建立廣告草稿 🔒

一次完成訂單建立與廣告草稿建立。每個訂單項目可攜帶多筆廣告資料，系統會自動建立狀態為 `draft` 的廣告。

**Request Body（JSON）**

| 欄位                        | 類型     | 必填 | 說明                                 |
| --------------------------- | -------- | ---- | ------------------------------------ |
| `items`                     | `array`  | ✅   | 訂單項目（至少 1 筆）                |
| `items[].type`              | `string` | ✅   | `plan` 或 `option`                   |
| `items[].id`                | `integer`| ✅   | 方案/選項 ID                         |
| `items[].quantity`          | `integer`| ❌   | 數量（min:1，預設 1）                |
| `items[].ads`               | `array`  | ❌   | 該項目對應的廣告資料                 |
| `items[].ads[].title`       | `string` | ✅   | 廣告標題（max:255）                  |
| `items[].ads[].description` | `string` | ❌   | 廣告描述                             |
| `items[].ads[].link_url`    | `string` | ❌   | 連結網址（需為合法 URL，max:2048）   |
| `discount`                  | `number` | ❌   | 折扣金額（min:0，上限不超過小計）    |
| `notes`                     | `string` | ❌   | 訂單備註（max:1000）                 |

**請求範例**

```json
{
    "items": [
        {
            "type": "plan",
            "id": 1,
            "quantity": 1,
            "ads": [
                {
                    "title": "春季促銷廣告",
                    "description": "限時優惠活動",
                    "link_url": "https://example.com/promo"
                }
            ]
        }
    ],
    "notes": "請盡快處理"
}
```

**回應範例（201）**

```json
{
    "success": true,
    "message": "訂單與廣告建立成功",
    "data": {
        "order": {
            "id": 1,
            "order_number": "ORD20260313XY56AB78",
            "order_status": "pending",
            "payment_status": "unpaid",
            "subtotal": 999.0,
            "discount": 0,
            "total_amount": 999.0,
            "items": [ ... ],
            "payment_logs": []
        },
        "ads": [
            {
                "id": 1,
                "title": "春季促銷廣告",
                "description": "限時優惠活動",
                "status": "draft",
                "user_id": 1,
                "order_item_id": 1
            }
        ]
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                              |
| ------ | --------------------------------- |
| `400`  | 業務錯誤（商品不存在、建立失敗等）|
| `401`  | 未授權                            |
| `422`  | 驗證失敗                          |

---

## GET `/api/orders` — 訂單列表 🔒

僅回傳**目前登入會員**的訂單。

**Query Parameters**

| 參數             | 類型     | 說明                             |
| ---------------- | -------- | -------------------------------- |
| `order_status`   | `string` | 篩選訂單狀態（見上方狀態值說明） |
| `payment_status` | `string` | 篩選付款狀態（見上方狀態值說明） |
| `date_from`      | `date`   | 建立日期起（`Y-m-d`）           |
| `date_to`        | `date`   | 建立日期迄（`Y-m-d`）           |

**請求範例**

```
GET /api/orders?order_status=active&date_from=2026-01-01&date_to=2026-12-31
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得訂單列表",
    "data": {
        "orders": [
            {
                "order_number": "ORD20260312AB12CD34",
                "order_status": "active",
                "payment_status": "paid",
                "total_amount": 3898.0,
                "created_at": "2026-03-13 11:00:00"
            }
        ],
        "summary": {
            "total": 1,
            "total_amount": 3898.0
        }
    }
}
```

---

## GET `/api/orders/{orderNumber}` — 訂單詳情 🔒

**Path Parameters**

| 參數          | 類型     | 說明                                 |
| ------------- | -------- | ------------------------------------ |
| `orderNumber` | `string` | 訂單編號（如 `ORD20260312AB12CD34`） |

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得訂單詳情",
    "data": {
        "order": {
            "id": 1,
            "order_number": "ORD20260312AB12CD34",
            "order_status": "active",
            "payment_status": "paid",
            "subtotal": 3998.00,
            "discount": 100.00,
            "total_amount": 3898.00,
            "paid_at": "2026-03-13 11:30:00",
            "starts_at": "2026-03-13 11:30:00",
            "expires_at": "2026-04-12 11:30:00",
            "notes": "希望盡快審核",
            "orderable": { ... },
            "items": [ ... ],
            "payment_logs": [ ... ],
            "created_at": "2026-03-13 11:00:00",
            "updated_at": "2026-03-13 11:30:00"
        }
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                               |
| ------ | ---------------------------------- |
| `401`  | 未授權                             |
| `404`  | 訂單不存在（或不屬於目前登入會員） |

---

## POST `/api/orders/{orderNumber}/cancel` — 取消訂單 🔒

> ⚠️ 只有 `payment_status = unpaid` 的訂單可以取消，已付款的訂單需改用退款流程。

**Path Parameters**

| 參數          | 類型     | 說明     |
| ------------- | -------- | -------- |
| `orderNumber` | `string` | 訂單編號 |

**Request Body（JSON）**

| 欄位     | 類型     | 必填 | 說明     |
| -------- | -------- | ---- | -------- |
| `reason` | `string` | ❌   | 取消原因 |

**請求範例**

```json
{
    "reason": "不需要了"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "訂單取消成功",
    "data": {
        "order": {
            "order_number": "ORD20260312AB12CD34",
            "order_status": "cancelled",
            "payment_status": "unpaid",
            "updated_at": "2026-03-13 12:00:00"
        }
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 訂單無法取消（已付款） |
| `401`  | 未授權                 |
| `404`  | 訂單不存在             |

---

## POST `/api/orders/{orderNumber}/pay` — 支付訂單 🔒

**Path Parameters**

| 參數          | 類型     | 說明     |
| ------------- | -------- | -------- |
| `orderNumber` | `string` | 訂單編號 |

**Request Body（JSON）**

| 欄位             | 類型     | 必填 | 說明                                          |
| ---------------- | -------- | ---- | --------------------------------------------- |
| `payment_method` | `string` | ✅   | 付款方式（如 `credit_card`、`bank_transfer`） |
| `payment_data`   | `object` | ❌   | 付款相關資料（依金流商而定）                  |

**請求範例**

```json
{
    "payment_method": "credit_card",
    "payment_data": {
        "card_number": "4111111111111111",
        "cvv": "123",
        "expiry": "12/28"
    }
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "付款成功",
    "data": {
        "payment_log": {
            "id": 1,
            "status": "success",
            "payment_method": "credit_card",
            "amount": "3898.00",
            "currency": "TWD",
            "transaction_id": "TXN_XXXXXX",
            "processed_at": "2026-03-13 11:30:00",
            "created_at": "2026-03-13 11:30:00"
        }
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                  |
| ------ | --------------------- |
| `400`  | 付款失敗 / 訂單已付款 |
| `401`  | 未授權                |
| `404`  | 訂單不存在            |
| `422`  | 驗證失敗              |

---

## POST `/api/orders/{orderNumber}/refund` — 申請退款 🔒

> 支援**全額退款**（不傳 `amount`）及**部分退款**（傳入 `amount`）。
> 系統會計算**累計已退款金額**，退款金額不可超過可退餘額。

**退款行為說明：**

- **全額退款**（累計退款 = 訂單總額）：訂單狀態改為 `cancelled`，付款狀態改為 `refunded`
- **部分退款**（累計退款 < 訂單總額）：訂單狀態**不變**，付款狀態改為 `partial_refund`
- **可退餘額** = `total_amount` - 累計已退款金額

**Path Parameters**

| 參數          | 類型     | 說明     |
| ------------- | -------- | -------- |
| `orderNumber` | `string` | 訂單編號 |

**Request Body（JSON）**

| 欄位     | 類型     | 必填 | 說明                              |
| -------- | -------- | ---- | --------------------------------- |
| `amount` | `number` | ❌   | 退款金額（不填則全額退款，min:0） |
| `reason` | `string` | ❌   | 退款原因（max:500）               |

**請求範例（部分退款）**

```json
{
    "amount": 500.0,
    "reason": "部分服務未使用"
}
```

**請求範例（全額退款）**

```json
{
    "reason": "決定不使用"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "退款成功",
    "data": {
        "payment_log": {
            "id": 2,
            "status": "refunded",
            "payment_method": "credit_card",
            "amount": "500.00",
            "currency": "TWD",
            "processed_at": "2026-03-13 14:00:00",
            "created_at": "2026-03-13 14:00:00"
        }
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                                       |
| ------ | ------------------------------------------ |
| `400`  | 退款失敗（訂單未付款、退款金額超過可退餘額等） |
| `401`  | 未授權                                     |
| `404`  | 訂單不存在                                 |
| `422`  | 驗證失敗                                   |

---

## GET `/api/orders/{orderNumber}/payments` — 付款記錄 🔒

**Path Parameters**

| 參數          | 類型     | 說明     |
| ------------- | -------- | -------- |
| `orderNumber` | `string` | 訂單編號 |

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得付款記錄",
    "data": {
        "payment_logs": [
            {
                "id": 1,
                "status": "success",
                "payment_method": "credit_card",
                "amount": "3898.00",
                "currency": "TWD",
                "transaction_id": "TXN_XXXXXX",
                "gateway": null,
                "error_code": null,
                "error_message": null,
                "processed_at": "2026-03-13 11:30:00",
                "created_at": "2026-03-13 11:30:00"
            },
            {
                "id": 2,
                "status": "refunded",
                "payment_method": "credit_card",
                "amount": "500.00",
                "currency": "TWD",
                "transaction_id": "REFUND_1710312000",
                "processed_at": "2026-03-13 14:00:00",
                "created_at": "2026-03-13 14:00:00"
            }
        ]
    }
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `401`  | 未授權     |
| `404`  | 訂單不存在 |
