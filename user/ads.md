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
      "status": "draft",
      "order_item_id": 1,
      "files": [],
      "created_at": "2025-03-01T10:00:00+08:00"
    }
  ]
}
```

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
    "status": "draft",
    "order_item_id": 1,
    "order_item": {
      "id": 1,
      "plan_option_id": 1,
      "quantity": 1
    },
    "reviewer": null,
    "rejected_reason": null,
    "files": [
      {
        "id": 1,
        "file_name": "ad_image.jpg",
        "url": "https://example.com/storage/ads/ad_image.jpg"
      }
    ],
    "created_at": "2025-03-01T10:00:00+08:00",
    "updated_at": "2025-03-01T10:00:00+08:00"
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

```json
{
  "order_item_id": 1,
  "title": "春季促銷廣告",
  "description": "限時優惠，全館 5 折",
  "link_url": "https://example.com/promo"
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
    "status": "draft",
    "order_item_id": 1,
    "created_at": "2025-03-01T10:00:00+08:00"
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

```json
{
  "title": "修改後的廣告標題",
  "description": "更新後的說明",
  "link_url": "https://example.com/new-promo"
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
    "status": "draft"
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
    "status": "draft",
    "files": [
      {
        "id": 10,
        "file_name": "ad_image.jpg",
        "url": "https://example.com/storage/ads/ad_image.jpg"
      }
    ]
  }
}
```

---

## 提交廣告審核

**POST** `/api/ads/{id}/submit`

將廣告由 `draft` 或 `rejected` 狀態提交至 `pending_review`，等待管理員審核。

> 建議在提交前已上傳廣告圖片，否則可能被退回。

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
    "status": "pending_review"
  }
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
