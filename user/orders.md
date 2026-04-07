# 訂單管理 API (會員)

> 以下 API 需在 Header 帶入會員 JWT Token：
> `Authorization: Bearer {token}`

---

## 訂單狀態說明

| 狀態值 | 說明 |
|--------|------|
| `pending` | 待處理（剛建立） |
| `processing` | 處理中 |
| `active` | 已啟用（廣告上架中） |
| `completed` | 已完成 |
| `cancelled` | 已取消 |
| `expired` | 已到期 |

## 付款狀態說明

| 狀態值 | 說明 |
|--------|------|
| `unpaid` | 未付款 |
| `paid` | 已付款 |
| `partial_refund` | 部分退款 |
| `refunded` | 已全額退款 |
| `failed` | 付款失敗 |

---

## 取得我的訂單列表

**GET** `/api/orders`

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `order_status` | string | 否 | 篩選訂單狀態 |
| `payment_status` | string | 否 | 篩選付款狀態 |
| `date_from` | string | 否 | 建立日期起始，格式：`Y-m-d` |
| `date_to` | string | 否 | 建立日期結束，格式：`Y-m-d` |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得訂單列表",
  "data": {
    "orders": [
      {
        "id": 1,
        "order_number": "ORD20251216ABCD1234",
        "order_status": "pending",
        "payment_status": "unpaid",
        "subtotal": 2699,
        "tax": 134,
        "discount": 0,
        "total": 2833,
        "notes": null,
        "created_at": "2025-12-16T10:00:00+08:00"
      }
    ]
  }
}
```

---

## 建立訂單

**POST** `/api/orders`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `items` | array | 是 | 訂單項目陣列，至少包含一筆 |
| `items[].plan_option_id` | integer | 是 | 方案選項 ID |
| `items[].quantity` | integer | 是 | 數量（最小 1） |
| `notes` | string | 否 | 訂單備註 |

```json
{
  "items": [
    {
      "plan_option_id": 1,
      "quantity": 1
    },
    {
      "plan_option_id": 3,
      "quantity": 2
    }
  ],
  "notes": "請盡快處理"
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "訂單建立成功",
  "data": {
    "order": {
      "id": 1,
      "order_number": "ORD20251216ABCD1234",
      "order_status": "pending",
      "payment_status": "unpaid",
      "subtotal": 2699,
      "tax": 134,
      "discount": 0,
      "total": 2833,
      "notes": "請盡快處理",
      "items": [
        {
          "id": 1,
          "plan_option_id": 1,
          "quantity": 1,
          "unit_price": 2699,
          "subtotal": 2699
        }
      ],
      "payment_logs": [],
      "created_at": "2025-12-16T10:00:00+08:00"
    }
  }
}
```

---

## 建立訂單並同時建立廣告草稿

**POST** `/api/orders/with-ads`

在建立訂單的同時，為每個訂單項目預先建立對應的廣告草稿，省去後續單獨建立廣告的步驟。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `items` | array | 是 | 訂單項目陣列 |
| `items[].plan_option_id` | integer | 是 | 方案選項 ID |
| `items[].quantity` | integer | 是 | 數量 |
| `items[].ads` | array | 否 | 該訂單項目對應的廣告資料陣列 |
| `items[].ads[].title` | string | 是（若有 ads） | 廣告標題 |
| `items[].ads[].description` | string | 否 | 廣告說明 |
| `items[].ads[].link_url` | string | 否 | 廣告點擊連結 URL |
| `notes` | string | 否 | 訂單備註 |

```json
{
  "items": [
    {
      "plan_option_id": 1,
      "quantity": 1,
      "ads": [
        {
          "title": "春季促銷廣告",
          "description": "限時優惠，全館 5 折",
          "link_url": "https://example.com/promo"
        }
      ]
    }
  ],
  "notes": "備註資訊"
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "訂單與廣告建立成功",
  "data": {
    "order": {
      "id": 1,
      "order_number": "ORD20251216ABCD1234",
      "order_status": "pending",
      "payment_status": "unpaid",
      "total": 2833,
      "items": [ ... ],
      "payment_logs": []
    },
    "ads": [
      {
        "id": 1,
        "title": "春季促銷廣告",
        "status": "draft",
        "order_item_id": 1
      }
    ]
  }
}
```

---

## 取得訂單詳情

**GET** `/api/orders/{orderNumber}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號，格式：`ORD{日期}{亂數}` |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得訂單詳情",
  "data": {
    "order": {
      "id": 1,
      "order_number": "ORD20251216ABCD1234",
      "order_status": "pending",
      "payment_status": "unpaid",
      "subtotal": 2699,
      "tax": 134,
      "discount": 0,
      "total": 2833,
      "invoice_number": null,
      "notes": null,
      "items": [
        {
          "id": 1,
          "plan_option_id": 1,
          "plan_option": {
            "name": "3個月方案",
            "duration_days": 90,
            "price": 2699
          },
          "quantity": 1,
          "unit_price": 2699,
          "subtotal": 2699,
          "starts_at": null,
          "ends_at": null
        }
      ],
      "payment_logs": [],
      "created_at": "2025-12-16T10:00:00+08:00"
    }
  }
}
```

---

## 取消訂單

**POST** `/api/orders/{orderNumber}/cancel`

僅限未付款（`payment_status: unpaid`）且處於可取消狀態的訂單。

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
    "order": {
      "id": 1,
      "order_number": "ORD20251216ABCD1234",
      "order_status": "cancelled",
      "payment_status": "unpaid"
    }
  }
}
```

### Response 400 - 無法取消

```json
{
  "success": false,
  "message": "訂單已付款，無法取消",
  "data": null
}
```

---

## 支付訂單

**POST** `/api/orders/{orderNumber}/pay`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `payment_method` | string | 是 | 付款方式，如：`credit_card`、`bank_transfer` |
| `payment_data` | object | 否 | 付款相關的額外資料（依付款方式而定） |

```json
{
  "payment_method": "credit_card",
  "payment_data": {
    "card_last4": "1234"
  }
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "付款成功",
  "data": {
    "payment_log": {
      "id": 1,
      "order_id": 1,
      "payment_method": "credit_card",
      "amount": 2833,
      "status": "success",
      "transaction_id": null,
      "created_at": "2025-12-16T10:30:00+08:00"
    }
  }
}
```

---

## 申請退款

**POST** `/api/orders/{orderNumber}/refund`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `amount` | number | 否 | 退款金額，不傳則全額退款 |
| `reason` | string | 否 | 退款原因 |

```json
{
  "amount": 500.00,
  "reason": "不符合需求"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "退款成功",
  "data": {
    "payment_log": {
      "id": 2,
      "order_id": 1,
      "payment_method": "refund",
      "amount": -500.00,
      "status": "success",
      "created_at": "2025-12-17T09:00:00+08:00"
    }
  }
}
```

---

## 取得訂單付款記錄

**GET** `/api/orders/{orderNumber}/payments`

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
    "payment_logs": [
      {
        "id": 1,
        "order_id": 1,
        "payment_method": "credit_card",
        "amount": 2833,
        "status": "success",
        "transaction_id": null,
        "notes": null,
        "created_at": "2025-12-16T10:30:00+08:00"
      }
    ]
  }
}
```
