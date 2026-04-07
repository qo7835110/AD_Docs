# 管理員訂單管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 取得列表 / 詳情 / 統計 | `orders:view` |
| 開立發票 / 確認付款 | `orders:update` |

---

## 訂單狀態說明

| 狀態值 | 說明 |
|--------|------|
| `pending` | 待處理（剛建立） |
| `processing` | 處理中 |
| `active` | 已啟用 |
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

## 取得訂單統計資料

**GET** `/api/admin/orders/statistics`

> **注意**：此路由必須在 `/api/admin/orders/{id}` 之前存取，路由定義中 `statistics` 優先匹配。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `date_from` | string | 否 | 統計起始日期，格式：`Y-m-d` |
| `date_to` | string | 否 | 統計結束日期，格式：`Y-m-d` |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得訂單統計資料",
  "data": {
    "total_orders": 100,
    "total_revenue": 50000.00,
    "total_tax": 2500.00,
    "total_discount": 1000.00,
    "by_order_status": {
      "pending": 10,
      "active": 60,
      "completed": 25,
      "cancelled": 5
    },
    "by_payment_status": {
      "unpaid": 10,
      "paid": 85,
      "refunded": 5
    }
  }
}
```

---

## 取得所有訂單列表

**GET** `/api/admin/orders`

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `order_status` | string | 否 | 篩選訂單狀態 |
| `payment_status` | string | 否 | 篩選付款狀態 |
| `order_number` | string | 否 | 依訂單編號搜尋（模糊比對） |
| `date_from` | string | 否 | 建立日期起始，格式：`Y-m-d` |
| `date_to` | string | 否 | 建立日期結束，格式：`Y-m-d` |
| `per_page` | integer | 否 | 每頁筆數，預設 `15` |

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
        "total": 2833,
        "orderable": {
          "id": 5,
          "name": "王小明",
          "email": "user@example.com"
        },
        "created_at": "2025-12-16T10:00:00+08:00"
      }
    ],
    "meta": {
      "current_page": 1,
      "last_page": 7,
      "per_page": 15,
      "total": 100
    }
  }
}
```

---

## 取得訂單詳情

**GET** `/api/admin/orders/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 訂單 ID（非訂單編號） |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得訂單詳情",
  "data": {
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
    "orderable": {
      "id": 5,
      "name": "王小明",
      "email": "user@example.com"
    },
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
```

---

## 開立發票

**POST** `/api/admin/orders/{id}/invoice`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 訂單 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `invoice_number` | string | 是 | 發票號碼（格式：`AA-12345678`） |

```json
{
  "invoice_number": "AA-12345678"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "發票開立成功",
  "data": {
    "id": 1,
    "order_number": "ORD20251216ABCD1234",
    "invoice_number": "AA-12345678"
  }
}
```

### Response 400 - 業務邏輯錯誤

```json
{
  "success": false,
  "message": "此訂單尚未付款，無法開立發票",
  "data": null
}
```

---

## 確認付款

**POST** `/api/admin/orders/{id}/confirm-payment`

手動確認訂單付款（適用於線下付款如銀行匯款的核對）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 訂單 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `payment_method` | string | 是 | 付款方式，如 `bank_transfer` |
| `payment_transaction_id` | string | 否 | 交易編號 |
| `notes` | string | 否 | 備註說明 |

```json
{
  "payment_method": "bank_transfer",
  "payment_transaction_id": "TXN202603261234",
  "notes": "已確認銀行匯款"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "付款確認成功",
  "data": {
    "id": 1,
    "order_number": "ORD20251216ABCD1234",
    "payment_status": "paid",
    "order_status": "processing"
  }
}
```

### Response 400 - 業務邏輯錯誤

```json
{
  "success": false,
  "message": "訂單已付款，無法重複確認",
  "data": null
}
```
