# 外部應用訂單 API

> 外部應用以 **API Key + API Secret** 認證，操作對象為綁定的 owner user 的訂單資料。
>
> 所有請求需在 Header 帶入：
> ```
> X-Api-Key: ext_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
> X-Api-Secret: <your-api-secret>
> ```
>
> 認證說明及錯誤代碼請參閱 [外部應用廣告 API](./ads.md)。

---

## Permission Scope 對照

| 端點 | 所需 scope |
|------|-----------|
| 取得訂單列表 | `orders.read` |
| 取得訂單詳情 | `orders.read` |
| 取得付款記錄 | `orders.read` |
| 取得訂單草稿 | `orders.read` |
| 建立訂單 | `orders.create` |
| 建立訂單並建廣告 | `orders.create` |
| 儲存 / 刪除訂單草稿 | `orders.create` |
| 取消訂單 | `orders.update` |
| 支付訂單 | `orders.update` |
| 申請退款 | `orders.update` |

---

## 取得訂單列表

**GET** `/api/external/orders`

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `order_status` | string | 否 | `pending` / `processing` / `active` / `completed` / `cancelled` / `expired` |
| `payment_status` | string | 否 | `unpaid` / `paid` / `partial_refund` / `refunded` / `failed` |
| `date_from` | date | 否 | 建立日期起始，格式 `YYYY-MM-DD` |
| `date_to` | date | 否 | 建立日期結束，格式 `YYYY-MM-DD` |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得訂單列表",
  "data": {
    "data": [
      {
        "id": 10,
        "order_number": "ORD20260601ABCD1234",
        "order_status": "pending",
        "payment_status": "unpaid",
        "amounts": {
          "subtotal": 1000,
          "discount": 0,
          "tax": 50,
          "total": 1050
        },
        "payment": {
          "payment_method": null,
          "payment_transaction_id": null,
          "paid_at": null
        },
        "validity": {
          "starts_at": null,
          "expires_at": null
        },
        "created_at": "2026-06-01 12:00:00",
        "updated_at": "2026-06-01 12:00:00"
      }
    ],
    "summary": {
      "total_count": 1,
      "total_amount": 1050,
      "paid_count": 0,
      "unpaid_count": 1
    }
  }
}
```

---

## 取得訂單詳情

**GET** `/api/external/orders/{orderNumber}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號，格式如 `ORD20260601ABCD1234` |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得訂單詳情",
  "data": {
    "order": {
      "id": 10,
      "order_number": "ORD20260601ABCD1234",
      "order_status": "pending",
      "payment_status": "unpaid",
      "subtotal": 1000,
      "discount": 0,
      "tax": 0,
      "total": 1000,
      "notes": null,
      "paid_at": null,
      "items": [ ... ],
      "payment_logs": []
    }
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "訂單不存在"
}
```

---

## 建立訂單

**POST** `/api/external/orders`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `items` | array | 是 | 訂單項目陣列 |
| `items[].plan_option_id` | integer | 是 | 方案選項 ID |
| `items[].quantity` | integer | 否 | 數量，最小 1，選填（預設為 1） |
| `notes` | string | 否 | 備註 |

```json
{
  "items": [
    { "plan_option_id": 3, "quantity": 1 }
  ],
  "notes": "外部系統自動建單"
}
```

### Response 201 - 建立成功

```json
{
  "success": true,
  "message": "訂單建立成功",
  "data": {
    "order": { ... }
  }
}
```

### Response 400 - 業務邏輯錯誤

```json
{
  "success": false,
  "message": "方案選項不存在或已下架"
}
```

---

## 建立訂單並同時建立廣告草稿

**POST** `/api/external/orders/with-ads`

建立訂單的同時，依各 item 的 `ads` 陣列建立廣告草稿（`draft` 狀態）。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `items` | array | 是 | 訂單項目陣列 |
| `items[].plan_option_id` | integer | 是 | 方案選項 ID |
| `items[].quantity` | integer | 是 | 數量 |
| `items[].ads` | array | 否 | 該 item 對應的廣告草稿 |
| `items[].ads[].title` | string | 是 | 廣告標題 |
| `items[].ads[].description` | string | 否 | 廣告描述 |
| `items[].ads[].link_url` | string | 否 | 廣告連結 |
| `notes` | string | 否 | 備註 |

```json
{
  "items": [
    {
      "plan_option_id": 3,
      "quantity": 1,
      "ads": [
        {
          "title": "春季徵才",
          "description": "誠徵資深工程師",
          "link_url": "https://partner.com/jobs/1"
        }
      ]
    }
  ]
}
```

### Response 201 - 建立成功

```json
{
  "success": true,
  "message": "訂單與廣告建立成功",
  "data": {
    "order": { ... },
    "ads": [ ... ]
  }
}
```

---

## 取消訂單

**POST** `/api/external/orders/{orderNumber}/cancel`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `reason` | string | 否 | 取消原因 |

```json
{
  "reason": "不需要了"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "訂單取消成功",
  "data": {
    "order": { ... }
  }
}
```

### Response 400 - 無法取消

```json
{
  "success": false,
  "message": "訂單狀態不允許取消"
}
```

---

## 支付訂單

**POST** `/api/external/orders/{orderNumber}/pay`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `payment_method` | string | 是 | 付款方式，如 `credit_card` |
| `payment_data` | object | 否 | 付款附加資料 |

```json
{
  "payment_method": "credit_card",
  "payment_data": {}
}
```

### Response 200 - 付款成功

```json
{
  "success": true,
  "message": "付款成功",
  "data": {
    "payment_log": { ... }
  }
}
```

### Response 400 - 付款失敗

```json
{
  "success": false,
  "message": "訂單已付款"
}
```

---

## 申請退款

**POST** `/api/external/orders/{orderNumber}/refund`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `reason` | string | 否 | 退款原因 |

### Response 200 - 退款成功

```json
{
  "success": true,
  "message": "退款成功",
  "data": {
    "payment_log": { ... }
  }
}
```

---

## 取得付款記錄

**GET** `/api/external/orders/{orderNumber}/payments`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得付款記錄",
  "data": {
    "payment_logs": [ ... ]
  }
}
```

---

## 訂單草稿

草稿以 Cache 儲存，每個 owner user 一份，**7 天**後自動過期。

> 草稿僅供暫存用途，多個外部應用若共用同一個 owner user，草稿會互相覆蓋。

### 儲存草稿（新增或覆蓋）

**POST** `/api/external/orders/drafts`

Request Body 格式同 `POST /api/external/orders/with-ads`，另可加 `discount` 欄位。

### 取得草稿

**GET** `/api/external/orders/drafts`

### 刪除草稿

**DELETE** `/api/external/orders/drafts`
