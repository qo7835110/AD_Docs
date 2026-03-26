# Admin API - 訂單管理
**身份驗證:** `auth:admin`   
**所需模組權限:** `orders`

> 管理員訂單管理模組，提供訂單審視、開立發票、手動確認付款及金額統計功能。

## [GET] `/api/admin/orders`
取得全平台訂單列表（分頁），支援多種條件篩選。
- **權限要求:** `admin.permission:orders,view`

### Query Params
| 參數 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `order_status` | string | 否 | 篩選訂單狀態：`pending`, `processing`, `active`, `completed`, `cancelled`, `expired` |
| `payment_status` | string | 否 | 篩選付款狀態：`unpaid`, `paid`, `partial_refund`, `refunded`, `failed` |
| `order_number` | string | 否 | 訂單編號模糊搜尋 |
| `date_from` | string (Y-m-d) | 否 | 建立日期起始 |
| `date_to` | string (Y-m-d) | 否 | 建立日期結束 |
| `per_page` | integer | 否 | 每頁筆數，預設 15 |

### Response 範例
```json
{
  "success": true,
  "message": "成功取得訂單列表",
  "data": {
    "data": [
      {
        "id": 1,
        "order_number": "ORD20260326ABCD1234",
        "order_status": "pending",
        "payment_status": "unpaid",
        "amounts": {
          "subtotal": 1000.00,
          "discount": 0.00,
          "tax": 50.00,
          "total": 1050.00
        },
        "payment": {
          "method": null,
          "transaction_id": null,
          "invoice_number": null,
          "paid_at": null
        },
        "orderable": {
          "type": "User",
          "id": 1,
          "name": "使用者名稱",
          "email": "user@example.com"
        },
        "created_at": "2026-03-26 10:00:00"
      }
    ],
    "meta": { "current_page": 1, "last_page": 5, "per_page": 15, "total": 72 }
  }
}
```

---

## [GET] `/api/admin/orders/{id}`
取得單一訂單完整詳情，包含訂單明細（items）、付款記錄（payment_logs）及訂購者資訊。
- **權限要求:** `admin.permission:orders,view`

### Path Params
| 參數 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `id` | integer | 是 | 訂單 ID |

### Response 範例
```json
{
  "success": true,
  "message": "成功取得訂單詳情",
  "data": {
    "id": 1,
    "order_number": "ORD20260326ABCD1234",
    "order_status": "active",
    "payment_status": "paid",
    "amounts": {
      "subtotal": 1000.00,
      "discount": 0.00,
      "tax": 50.00,
      "total": 1050.00
    },
    "payment": {
      "method": "bank_transfer",
      "transaction_id": "TXN202603261234",
      "invoice_number": "AA-12345678",
      "paid_at": "2026-03-26 12:00:00"
    },
    "validity": {
      "starts_at": "2026-03-26 12:00:00",
      "expires_at": "2026-04-25 12:00:00",
      "is_expired": false,
      "is_active": true
    },
    "orderable": {
      "type": "User",
      "id": 1,
      "name": "使用者名稱",
      "email": "user@example.com"
    },
    "items": [
      {
        "id": 1,
        "plan_option_id": 1,
        "item_name": "基本方案",
        "quantity": 1,
        "unit_price": 1000.00,
        "subtotal": 1000.00,
        "validity": { "duration_days": 30 }
      }
    ],
    "payment_logs": [
      {
        "id": 1,
        "status": "success",
        "amount": 1050.00,
        "payment_method": "bank_transfer",
        "transaction_id": "TXN202603261234",
        "processed_at": "2026-03-26 12:00:00"
      }
    ],
    "notes": null,
    "created_at": "2026-03-26 10:00:00",
    "updated_at": "2026-03-26 12:00:00"
  }
}
```

---

## [POST] `/api/admin/orders/{id}/invoice`
為已付款訂單開立發票，寫入發票號碼。
- **權限要求:** `admin.permission:orders,update`
- **業務規則:** 訂單必須為已付款（`payment_status = paid`）且尚未開立過發票。

### Payload 說明
| 欄位 | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `invoice_number` | string | required, max:50, unique | 是 | 發票號碼（不可重複） |

### Payload 範例
```json
{
  "invoice_number": "AA-12345678"
}
```

### Response 範例（成功）
```json
{
  "success": true,
  "message": "發票開立成功",
  "data": {
    "id": 1,
    "order_number": "ORD20260326ABCD1234",
    "payment": {
      "method": "bank_transfer",
      "transaction_id": "TXN202603261234",
      "invoice_number": "AA-12345678",
      "paid_at": "2026-03-26 12:00:00"
    }
  }
}
```

### 錯誤情境
| HTTP Code | 情境 |
|---|---|
| 400 | 訂單尚未付款 / 已開立過發票 |
| 404 | 訂單不存在 |
| 422 | 驗證失敗（缺少 invoice_number 或號碼重複） |

---

## [POST] `/api/admin/orders/{id}/confirm-payment`
管理員手動確認訂單付款（例如銀行匯款確認），系統會自動建立付款記錄並啟用訂單。
- **權限要求:** `admin.permission:orders,update`
- **業務規則:** 訂單不可為已付款或已取消狀態。
- **連動行為:** 確認後訂單狀態自動轉為 `active`，付款狀態轉為 `paid`，同時設定 `starts_at` 與 `expires_at`。

### Payload 說明
| 欄位 | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `payment_method` | string | required, max:50 | 是 | 付款方式（如 `bank_transfer`, `cash`） |
| `payment_transaction_id` | string | max:255 | 否 | 交易編號 |
| `notes` | string | max:1000 | 否 | 備註說明 |

### Payload 範例
```json
{
  "payment_method": "bank_transfer",
  "payment_transaction_id": "TXN202603261234",
  "notes": "已確認銀行匯款入帳"
}
```

### Response 範例（成功）
```json
{
  "success": true,
  "message": "付款確認成功",
  "data": {
    "id": 1,
    "order_number": "ORD20260326ABCD1234",
    "order_status": "active",
    "payment_status": "paid",
    "payment": {
      "method": "bank_transfer",
      "transaction_id": "TXN202603261234",
      "invoice_number": null,
      "paid_at": "2026-03-26 14:00:00"
    },
    "validity": {
      "starts_at": "2026-03-26 14:00:00",
      "expires_at": "2026-04-25 14:00:00"
    }
  }
}
```

### 錯誤情境
| HTTP Code | 情境 |
|---|---|
| 400 | 訂單已付款 / 訂單已取消 |
| 404 | 訂單不存在 |
| 422 | 驗證失敗（缺少 payment_method） |

---

## [GET] `/api/admin/orders/statistics`
取得訂單金額統計報表，包含總訂單數、總營收、稅額、折扣，以及按訂單狀態和付款狀態分組的彙總。
- **權限要求:** `admin.permission:orders,view`

### Query Params
| 參數 | 型別 | 必填 | 說明 |
|---|---|---|---|
| `date_from` | string (Y-m-d) | 否 | 統計起始日期 |
| `date_to` | string (Y-m-d) | 否 | 統計結束日期 |

### Response 範例
```json
{
  "success": true,
  "message": "成功取得訂單統計資料",
  "data": {
    "total_orders": 150,
    "total_revenue": 125000.00,
    "total_tax": 6250.00,
    "total_discount": 3000.00,
    "by_order_status": {
      "pending": { "order_status": "pending", "count": 20, "total_amount": 18000.00 },
      "active": { "order_status": "active", "count": 80, "total_amount": 72000.00 },
      "completed": { "order_status": "completed", "count": 35, "total_amount": 28000.00 },
      "cancelled": { "order_status": "cancelled", "count": 10, "total_amount": 5000.00 },
      "expired": { "order_status": "expired", "count": 5, "total_amount": 2000.00 }
    },
    "by_payment_status": {
      "unpaid": { "payment_status": "unpaid", "count": 20, "total_amount": 18000.00 },
      "paid": { "payment_status": "paid", "count": 120, "total_amount": 100000.00 },
      "refunded": { "payment_status": "refunded", "count": 10, "total_amount": 7000.00 }
    }
  }
}
```
