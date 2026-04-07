# 管理員廣告方案管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 建立方案 | `ad_plans:create` |
| 更新方案 | `ad_plans:update` |
| 刪除方案 | `ad_plans:delete` |
| 建立選項 | `plan_options:create` |
| 更新選項 | `plan_options:update` |
| 刪除選項 | `plan_options:delete` |

---

## 建立廣告方案

**POST** `/api/admin/ad-plans`

**Content-Type**: `multipart/form-data`（支援圖片上傳）

### Request Body (Form Data)

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `category` | string | 是 | 分類 slug（如 `meeting-management`）或分類 ID |
| `name` | string | 是 | 方案名稱 |
| `description` | string | 否 | 方案說明 |
| `ad_limit` | integer | 否 | 最大廣告數量限制 |
| `status` | string | 否 | 狀態，可選值：`active`、`inactive`，預設 `active` |
| `features` | string | 否 | 方案特色，JSON 字串格式，如：`["基本曝光","7x24 客服"]` |
| `image` | file | 否 | 方案封面圖片，支援：`jpeg`、`jpg`、`png`、`gif`、`webp`，最大 **2MB** |
| `options` | string | 否 | 方案選項，JSON 字串格式（見下方說明） |

**options 欄位格式範例（JSON 字串）：**

```json
[
  {
    "name": "1個月方案",
    "description": "月租優惠",
    "duration_days": 30,
    "price": 999,
    "valid_start_date": "2025-01-01",
    "valid_end_date": "2025-12-31",
    "sort_order": 1
  }
]
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "廣告方案建立成功",
  "data": {
    "id": 1,
    "category_id": 2,
    "name": "專業方案",
    "description": "適合中型企業使用",
    "ad_limit": 50,
    "status": "active",
    "features": ["進階曝光", "專屬客服"],
    "image": {
      "id": 5,
      "file_name": "plan_pro.jpg",
      "url": "https://example.com/storage/plans/plan_pro.jpg"
    },
    "options": [
      {
        "id": 1,
        "name": "1個月方案",
        "duration_days": 30,
        "price": 999,
        "sort_order": 1
      }
    ]
  }
}
```

---

## 更新廣告方案

**PUT** `/api/admin/ad-plans/{id}`

**Content-Type**: `multipart/form-data`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告方案 ID |

### Request Body

欄位說明同「建立廣告方案」，所有欄位皆為選填（僅更新有傳入的欄位）。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告方案更新成功",
  "data": {
    "id": 1,
    "name": "更新後的方案名稱",
    "status": "active",
    "image": { ... },
    "options": [ ... ]
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "廣告方案不存在",
  "data": null
}
```

---

## 刪除廣告方案

**DELETE** `/api/admin/ad-plans/{id}`

軟刪除廣告方案。若方案已有關聯的訂單，可能無法刪除。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告方案 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告方案刪除成功",
  "data": null
}
```

---

## 為方案新增選項

**POST** `/api/admin/ad-plans/{planId}/options`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `planId` | integer | 廣告方案 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 是 | 選項名稱（如「3個月方案」） |
| `description` | string | 否 | 選項說明 |
| `duration_days` | integer | 是 | 廣告刊登天數 |
| `price` | integer | 是 | 價格（新台幣，整數） |
| `valid_start_date` | string | 否 | 選項適用起始日，格式：`Y-m-d` |
| `valid_end_date` | string | 否 | 選項適用結束日，格式：`Y-m-d` |
| `sort_order` | integer | 否 | 排序序號，預設 `0` |

```json
{
  "name": "3個月方案",
  "description": "季度優惠方案",
  "duration_days": 90,
  "price": 2699,
  "valid_start_date": "2025-01-01",
  "valid_end_date": "2025-12-31",
  "sort_order": 1
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "方案選項建立成功",
  "data": {
    "id": 1,
    "ad_plan_id": 1,
    "name": "3個月方案",
    "duration_days": 90,
    "price": 2699,
    "sort_order": 1
  }
}
```

---

## 更新方案選項

**PUT** `/api/admin/ad-plans/options/{id}`

> **注意**：路徑中的 `ad-plans/options/{id}` 是固定格式，不需要帶入 `planId`。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 方案選項 ID |

### Request Body

欄位說明同「新增選項」，所有欄位皆為選填。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "方案選項更新成功",
  "data": {
    "id": 1,
    "name": "3個月方案",
    "price": 2999
  }
}
```

---

## 刪除方案選項

**DELETE** `/api/admin/ad-plans/options/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 方案選項 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "方案選項刪除成功",
  "data": null
}
```
