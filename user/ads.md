# 廣告管理 API (會員)

> 以下 API 需在 Header 帶入會員 JWT Token：
> `Authorization: Bearer {token}`

---

## 廣告狀態流程說明

```
draft (草稿)
  |
  | [submit] 提交審核
  v
pending_review (待審核)
  |
  |-- [approve] 審核通過 --> approved (已核准)
  |                              |
  |                              | [activate] 上架
  |                              v
  |                           active (上架中)
  |                              |
  |                              | [deactivate] 下架
  |                              v
  |                           inactive (已下架)
  |
  |-- [reject] 審核拒絕 --> rejected (已拒絕)
                              |
                              | [update + submit] 修改後重新提交
                              v
                           pending_review (重新待審核)
```

可刪除的狀態：`draft`、`rejected`、`inactive`

---

## 取得我的廣告列表

**GET** `/api/ads`

只回傳目前登入會員所擁有的廣告。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選廣告狀態，可選值：`draft`、`pending_review`、`approved`、`active`、`inactive`、`rejected`、`expired` |

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
      "status": "draft",
      "target_tags": [
        { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
      ],
      "exclude_tags": [],
      "review": {
        "rejection_reason": null,
        "reviewed_by": null,
        "submitted_at": null,
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

> `target_tags` / `exclude_tags`：廣告的投放目標標籤與排除標籤物件陣列。
> `review.reviewer_name`、`order_item`、`files` 僅在詳情端點中回傳（需載入關聯）。

---

## 取得廣告詳情

**GET** `/api/ads/{id}`

只能查看自己的廣告，存取他人廣告回傳 403。

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
    "status": "draft",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [
      { "id": 5, "name": "兒童", "raw_name": "children", "type": "demographic", "is_targetable": true, "description": null }
    ],
    "review": {
      "rejection_reason": null,
      "reviewed_by": null,
      "reviewer_name": null,
      "submitted_at": null,
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
      "created_at": "2025-03-01 10:00:00"
    },
    "files": [
      {
        "id": 1,
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

## 建立廣告草稿

**POST** `/api/ads`

建立廣告前，需先有已付款的訂單項目，並以其 `order_item_id` 關聯。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `order_item_id` | integer | 是 | 對應的訂單項目 ID（必須屬於此會員） |
| `title` | string | 是 | 廣告標題 |
| `description` | string | 否 | 廣告說明 |
| `link_url` | string | 否 | 廣告點擊連結 URL（需為合法 URL） |
| `target_tags` | integer[] | 否 | 投放目標標籤 ID 陣列（`behavior_type=1`） |
| `exclude_tags` | integer[] | 否 | 排除標籤 ID 陣列（`behavior_type=-1`） |

```json
{
  "order_item_id": 1,
  "title": "春季促銷廣告",
  "description": "限時優惠，全館 5 折",
  "link_url": "https://example.com/promo",
  "target_tags": [1, 3],
  "exclude_tags": [5]
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "廣告草稿建立成功",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "description": "限時優惠，全館 5 折",
    "link_url": "https://example.com/promo",
    "image_path": null,
    "status": "draft",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [
      { "id": 5, "name": "兒童", "raw_name": "children", "type": "demographic", "is_targetable": true, "description": null }
    ],
    "review": {
      "rejection_reason": null,
      "reviewed_by": null,
      "submitted_at": null,
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
}
```

### Response 403 - 訂單項目不屬於此會員

```json
{
  "success": false,
  "message": "該訂單項目不屬於您",
  "data": null
}
```

---

## 更新廣告內容

**PUT** `/api/ads/{id}`

僅限狀態為 `draft` 或 `rejected` 的廣告可編輯。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `title` | string | 否 | 廣告標題 |
| `description` | string | 否 | 廣告說明 |
| `link_url` | string | 否 | 廣告點擊連結 URL |
| `target_tags` | integer[] | 否 | 投放目標標籤 ID 陣列（`behavior_type=1`）；傳入即全量覆寫 |
| `exclude_tags` | integer[] | 否 | 排除標籤 ID 陣列（`behavior_type=-1`）；傳入即全量覆寫 |

```json
{
  "title": "修改後的廣告標題",
  "description": "更新後的說明",
  "link_url": "https://example.com/new-promo",
  "target_tags": [1, 3],
  "exclude_tags": [5]
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告更新成功",
  "data": {
    "id": 1,
    "title": "修改後的廣告標題",
    "description": "更新後的說明",
    "link_url": "https://example.com/new-promo",
    "image_path": "ads/ad_1.jpg",
    "status": "draft",
    "target_tags": [
      { "id": 1, "name": "科技愛好者", "raw_name": "tech", "type": "interest", "is_targetable": true, "description": null }
    ],
    "exclude_tags": [
      { "id": 5, "name": "兒童", "raw_name": "children", "type": "demographic", "is_targetable": true, "description": null }
    ],
    "review": {
      "rejection_reason": null,
      "reviewed_by": null,
      "submitted_at": null,
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
    "updated_at": "2025-03-02 09:00:00"
  }
}
```

### Response 400 - 狀態不允許編輯

```json
{
  "success": false,
  "message": "廣告狀態不允許編輯",
  "data": null
}
```

---

## 上傳廣告圖片

**POST** `/api/ads/{id}/image`

上傳或替換廣告的展示圖片。僅限狀態為 `draft` 或 `rejected` 的廣告。

**Content-Type**: `multipart/form-data`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body (Form Data)

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `image` | file | 是 | 圖片檔案，支援格式：`jpeg`、`png`、`gif`、`webp`，最大 **2MB** |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "圖片上傳成功",
  "data": {
    "id": 1,
    "title": "春季促銷廣告",
    "description": "限時優惠，全館 5 折",
    "link_url": "https://example.com/promo",
    "image_path": "ads/ad_1.jpg",
    "status": "draft",
    "target_tags": [],
    "exclude_tags": [],
    "review": {
      "rejection_reason": null,
      "reviewed_by": null,
      "submitted_at": null,
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

## 提交廣告審核

**POST** `/api/ads/{id}/submit`

將廣告由 `draft` 或 `rejected` 狀態提交至 `pending_review`，等待管理員審核。

> 建議在提交前已上傳廣告圖片，否則可能被退回。
>
> **數量限制**：同一訂單項目下，狀態不為 `rejected` 的廣告總數不可超過該訂單項目的 `quantity`。超出時回傳 422。

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
  "message": "廣告已提交審核",
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
      "submitted_at": "2025-03-02 09:30:00",
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
    "updated_at": "2025-03-02 09:30:00"
  }
}
```

### Response 422 - 超出廣告數量上限

```json
{
  "success": false,
  "message": "此訂單項目的廣告數量已達上限（1）",
  "data": null
}
```

### Response 400 - 狀態不允許提交

```json
{
  "success": false,
  "message": "目前狀態不允許提交審核",
  "data": null
}
```

---

## 刪除廣告

**DELETE** `/api/ads/{id}`

軟刪除廣告，僅限狀態為 `draft`、`rejected`、`inactive` 的廣告。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告刪除成功",
  "data": null
}
```

### Response 400 - 狀態不允許刪除

```json
{
  "success": false,
  "message": "廣告狀態不允許刪除",
  "data": null
}
```
