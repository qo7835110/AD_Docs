# 管理員廣告審核 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`
>
> 所有端點均需對應的模組權限（以 `admin.permission` Middleware 控管）。

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 取得廣告列表 / 詳情 / 統計 | `ads:view` |
| 審核通過 / 拒絕 / 上架 / 下架 | `ads:update` |

---

## 取得所有廣告列表

**GET** `/api/admin/ads`

取得平台上所有廣告，可依狀態與日期篩選。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選廣告狀態，可選值：`draft`、`pending_review`、`approved`、`active`、`inactive`、`rejected`、`expired` |
| `date_from` | string | 否 | 建立日期起始，格式：`Y-m-d`（如 `2026-01-01`） |
| `date_to` | string | 否 | 建立日期結束，格式：`Y-m-d`（如 `2026-12-31`） |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得廣告列表",
  "data": [
    {
      "id": 1,
      "title": "春季促銷廣告",
      "description": "限時優惠，全館 5 折",
      "link_url": "https://example.com/promo",
      "image_path": "ads/ad_1.jpg",
      "status": "pending_review",
      "target_tags": [
        { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
      ],
      "exclude_tags": [],
      "review": {
        "rejection_reason": null,
        "reviewed_by": null,
        "submitted_at": "2025-02-18 09:00:00",
        "approved_at": null
      },
      "schedule": {
        "starts_at": null,
        "expires_at": null
      },
      "owner": {
        "type": "User",
        "id": 5
      },
      "order_item_id": 1,
      "created_at": "2025-03-01 10:00:00",
      "updated_at": "2025-03-01 10:00:00"
    }
  ]
}
```

> 列表端點未載入 `owner` 關聯，因此 `owner.name` / `owner.email` 不會出現；如需完整擁有者資訊請使用詳情端點。

---

## 取得廣告詳情

**GET** `/api/admin/ads/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得廣告詳情",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "description": "限時優惠，全館 5 折",
    "link_url": "https://example.com/promo",
    "image_path": "ads/ad_1.jpg",
    "status": "pending_review",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [],
    "review": {
      "rejection_reason": null,
      "reviewed_by": null,
      "reviewer_name": null,
      "submitted_at": "2025-02-18 09:00:00",
      "approved_at": null
    },
    "schedule": {
      "starts_at": "2025-03-01 00:00:00",
      "expires_at": "2025-03-31 23:59:59"
    },
    "owner": {
      "type": "User",
      "id": 5,
      "name": "王小明",
      "email": "user@example.com"
    },
    "order_item_id": 1,
    "order_item": {
      "id": 1,
      "plan_option_id": 2,
      "item_name": "基本方案 - 30天",
      "item_description": null,
      "quantity": 1,
      "unit_price": 1000.0,
      "subtotal": 1000.0,
      "validity": {
        "duration_days": 30,
        "valid_start_date": "2025-03-01",
        "valid_end_date": "2025-03-31"
      },
      "metadata": null,
      "created_at": "2025-02-15 10:00:00"
    },
    "files": [
      {
        "id": 10,
        "file_name": "ad_image.jpg",
        "url": "https://example.com/storage/ads/ad_image.jpg"
      }
    ],
    "created_at": "2025-03-01 10:00:00",
    "updated_at": "2025-03-01 10:00:00"
  }
}
```

---

## 審核通過

**POST** `/api/admin/ads/{id}/approve`

將廣告由 `pending_review` 狀態改為 `approved`（已核准，尚未上架）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

不需傳入任何 Body。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告審核通過",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "description": "限時優惠，全館 5 折",
    "link_url": "https://example.com/promo",
    "image_path": "ads/ad_1.jpg",
    "status": "approved",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [],
    "review": {
      "rejection_reason": null,
      "reviewed_by": 1,
      "submitted_at": "2025-02-18 09:00:00",
      "approved_at": "2025-02-19 11:30:00"
    },
    "schedule": {
      "starts_at": "2025-03-01 00:00:00",
      "expires_at": "2025-03-31 23:59:59"
    },
    "owner": {
      "type": "User",
      "id": 5
    },
    "order_item_id": 1,
    "created_at": "2025-03-01 10:00:00",
    "updated_at": "2025-02-19 11:30:00"
  }
}
```

### Response 400 - 廣告狀態不允許審核

```json
{
  "success": false,
  "message": "廣告狀態不允許此操作",
  "data": null
}
```

---

## 審核拒絕

**POST** `/api/admin/ads/{id}/reject`

將廣告由 `pending_review` 狀態改為 `rejected`，需填寫拒絕原因。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `reason` | string | 是 | 拒絕原因（將顯示給廣告主） |

```json
{
  "reason": "圖片不符合廣告規範，請重新上傳"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告審核已拒絕",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "description": "限時優惠，全館 5 折",
    "link_url": "https://example.com/promo",
    "image_path": "ads/ad_1.jpg",
    "status": "rejected",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [],
    "review": {
      "rejection_reason": "圖片不符合廣告規範，請重新上傳",
      "reviewed_by": 1,
      "submitted_at": "2025-02-18 09:00:00",
      "approved_at": null
    },
    "schedule": {
      "starts_at": null,
      "expires_at": null
    },
    "owner": {
      "type": "User",
      "id": 5
    },
    "order_item_id": 1,
    "created_at": "2025-03-01 10:00:00",
    "updated_at": "2025-02-19 14:00:00"
  }
}
```

---

## 上架廣告

**POST** `/api/admin/ads/{id}/activate`

將廣告由 `approved`（已核准）狀態改為 `active`（上架中）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

不需傳入任何 Body。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告已上架",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "status": "active",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [],
    "review": {
      "rejection_reason": null,
      "reviewed_by": 1,
      "submitted_at": "2025-02-18 09:00:00",
      "approved_at": "2025-02-19 11:30:00"
    },
    "schedule": {
      "starts_at": "2025-03-01 00:00:00",
      "expires_at": "2025-03-31 23:59:59"
    },
    "owner": { "type": "User", "id": 5 },
    "order_item_id": 1,
    "created_at": "2025-03-01 10:00:00",
    "updated_at": "2025-03-01 00:00:00"
  }
}
```

---

## 下架廣告

**POST** `/api/admin/ads/{id}/deactivate`

將廣告由 `active` 狀態改為 `inactive`（已下架）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

不需傳入任何 Body。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告已下架",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "status": "inactive",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [],
    "review": {
      "rejection_reason": null,
      "reviewed_by": 1,
      "submitted_at": "2025-02-18 09:00:00",
      "approved_at": "2025-02-19 11:30:00"
    },
    "schedule": {
      "starts_at": "2025-03-01 00:00:00",
      "expires_at": "2025-03-31 23:59:59"
    },
    "owner": { "type": "User", "id": 5 },
    "order_item_id": 1,
    "created_at": "2025-03-01 10:00:00",
    "updated_at": "2025-03-20 15:00:00"
  }
}
```

---

## 取得廣告成效統計

**GET** `/api/admin/ads/{id}/stats`

取得廣告在指定日期區間內的曝光次數、點擊次數、CTR 與每日趨勢。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `from` | string | 否 | 起始日期，格式：`Y-m-d`，預設為近 30 天 |
| `to` | string | 否 | 結束日期，格式：`Y-m-d`，預設為今日 |

> 日期區間最多 **365 天**，且 `to` 必須大於等於 `from`。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得廣告統計",
  "data": {
    "ad_id": 1,
    "period": {
      "from": "2026-03-09",
      "to": "2026-04-08"
    },
    "total_impressions": 1800,
    "total_clicks": 90,
    "ctr": "5.00%",
    "daily": [
      {
        "date": "2026-04-07",
        "impressions": 120,
        "clicks": 6
      },
      {
        "date": "2026-04-08",
        "impressions": 80,
        "clicks": 3
      }
    ]
  }
}
```

### Response 422 - 日期格式錯誤

```json
{
  "success": false,
  "message": "日期格式錯誤",
  "data": {
    "from": ["起始日期格式必須為 Y-m-d"]
  }
}
```
