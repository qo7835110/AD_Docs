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

## 訂單草稿

每位使用者僅能持有**一份**草稿，儲存於 Cache，**7 天**後自動過期。草稿用於前端暫存尚未確認的訂單內容；正式下單請呼叫 [建立訂單](#建立訂單) 或 [建立訂單並同時建立廣告草稿](#建立訂單並同時建立廣告草稿)。

### 儲存訂單草稿（新增或覆蓋）

**POST** `/api/orders/drafts`

呼叫此端點會直接覆蓋舊草稿（upsert），不會累加。

#### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `items` | array | 是 | 訂單項目陣列，至少一筆 |
| `items[].plan_option_id` | integer | 是 | 方案選項 ID |
| `items[].quantity` | integer | 否 | 數量，預設 1 |
| `items[].ads` | array | 否 | 預填廣告草稿資料 |
| `items[].ads[].title` | string | 是（若有 ads） | 廣告標題 |
| `items[].ads[].description` | string | 否 | 廣告說明 |
| `items[].ads[].link_url` | string | 否 | 廣告連結 URL |
| `notes` | string | 否 | 備註 |

```json
{
  "items": [
    {
      "plan_option_id": 1,
      "quantity": 1,
      "ads": [
        {
          "title": "春季促銷廣告",
          "description": "限時優惠",
          "link_url": "https://example.com/promo"
        }
      ]
    }
  ],
  "notes": "稍後確認"
}
```

#### Response 200 - 成功

```json
{
  "success": true,
  "message": "草稿已儲存",
  "data": {
    "draft": {
      "items": [
        {
          "plan_option_id": 1,
          "quantity": 1,
          "ads": [
            {
              "title": "春季促銷廣告",
              "description": "限時優惠",
              "link_url": "https://example.com/promo"
            }
          ]
        }
      ],
      "notes": "稍後確認",
      "expires_at": "2026-05-12T14:56:00+08:00",
      "saved_at": "2026-05-05T14:56:00+08:00"
    }
  }
}
```

---

### 取得目前的訂單草稿

**GET** `/api/orders/drafts`

#### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得草稿",
  "data": {
    "draft": {
      "items": [ ... ],
      "notes": "稍後確認",
      "expires_at": "2026-05-12T14:56:00+08:00",
      "saved_at": "2026-05-05T14:56:00+08:00"
    }
  }
}
```

#### Response 404 - 草稿不存在

```json
{
  "success": false,
  "message": "草稿不存在",
  "data": null
}
```

---

### 刪除訂單草稿

**DELETE** `/api/orders/drafts`

#### Response 200 - 成功

```json
{
  "success": true,
  "message": "草稿已刪除",
  "data": null
}
```

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
    "data": [
      {
        "id": 1,
        "order_number": "ORD20251216ABCD1234",
        "order_status": "pending",
        "payment_status": "unpaid",
        "amounts": {
          "subtotal": 2699.00,
          "discount": 0.00,
          "tax": 134.95,
          "total": 2833.95
        },
        "notes": null,
        "created_at": "2025-12-16 10:00:00",
        "updated_at": "2025-12-16 10:00:00"
      }
    ],
    "summary": {
      "total_count": 1,
      "total_amount": 2833.95,
      "paid_count": 0,
      "unpaid_count": 1
    }
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
      "amounts": {
        "subtotal": 2699.00,
        "discount": 0.00,
        "tax": 134.95,
        "total": 2833.95
      },
      "payment": {
        "method": null,
        "transaction_id": null,
        "invoice_number": null,
        "paid_at": null
      },
      "validity": {
        "starts_at": null,
        "expires_at": null,
        "is_expired": false,
        "is_active": false
      },
      "orderable": {
        "type": "User",
        "id": 1,
        "name": "Test User",
        "email": "test@example.com"
      },
      "items": [
        {
          "id": 1,
          "plan_option_id": 1,
          "quantity": 1,
          "unit_price": 2699.00,
          "subtotal": 2699.00
        }
      ],
      "payment_logs": [],
      "notes": "請盡快處理",
      "request_info": {
        "ip_address": "127.0.0.1",
        "user_agent": null
      },
      "created_at": "2025-12-16 10:00:00",
      "updated_at": "2025-12-16 10:00:00"
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
      "amounts": {
        "subtotal": 2699.00,
        "discount": 0.00,
        "tax": 134.95,
        "total": 2833.95
      },
      "payment": {
        "method": null,
        "transaction_id": null,
        "invoice_number": null,
        "paid_at": null
      },
      "validity": {
        "starts_at": null,
        "expires_at": null,
        "is_expired": false,
        "is_active": false
      },
      "orderable": {
        "type": "User",
        "id": 1,
        "name": "Test User",
        "email": "test@example.com"
      },
      "items": [ ... ],
      "payment_logs": [],
      "notes": "備註資訊",
      "created_at": "2025-12-16 10:00:00",
      "updated_at": "2025-12-16 10:00:00"
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
      "amounts": {
        "subtotal": 2699.00,
        "discount": 0.00,
        "tax": 134.95,
        "total": 2833.95
      },
      "payment": {
        "method": null,
        "transaction_id": null,
        "invoice_number": null,
        "paid_at": null
      },
      "validity": {
        "starts_at": null,
        "expires_at": null,
        "is_expired": false,
        "is_active": false
      },
      "orderable": {
        "type": "User",
        "id": 1,
        "name": "Test User",
        "email": "test@example.com"
      },
      "items": [
        {
          "id": 1,
          "plan_option_id": 1,
          "quantity": 1,
          "unit_price": 2699.00,
          "subtotal": 2699.00
        }
      ],
      "payment_logs": [],
      "notes": null,
      "request_info": {
        "ip_address": "127.0.0.1",
        "user_agent": null
      },
      "created_at": "2025-12-16 10:00:00",
      "updated_at": "2025-12-16 10:00:00"
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
  "message": "已付款訂單無法直接取消"
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
      "type": null,
      "payment_method": "credit_card",
      "amount": 2833,
      "status": "success",
      "transaction_id": null,
      "gateway_response": {
        "status": "success",
        "message": "付款成功",
        "amount": 2833
      },
      "notes": null,
      "processed_at": "2025-12-16 10:30:00",
      "created_at": "2025-12-16 10:30:00"
    }
  }
}
```

### Response 400 - 無法付款（已取消）

```json
{
  "success": false,
  "message": "已取消訂單不可付款"
}
```

---

## 申請退款

**POST** `/api/orders/{orderNumber}/refund`

目前僅支援**全額退款**，退款金額由系統依訂單金額決定，前端不可傳入 `amount`。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `orderNumber` | string | 訂單編號 |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `reason` | string | 否 | 退款原因 |

```json
{
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
      "type": null,
      "payment_method": "credit_card",
      "amount": -2833.00,
      "status": "refunded",
      "transaction_id": "REFUND_1760000000",
      "gateway_response": null,
      "notes": null,
      "processed_at": "2025-12-17 09:00:00",
      "created_at": "2025-12-17 09:00:00"
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
        "type": null,
        "payment_method": "credit_card",
        "amount": 2833,
        "status": "success",
        "transaction_id": null,
        "gateway_response": {
          "status": "success",
          "message": "付款成功",
          "amount": 2833
        },
        "notes": null,
        "processed_at": "2025-12-16 10:30:00",
        "created_at": "2025-12-16 10:30:00"
      }
    ]
  }
}
```
