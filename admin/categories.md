# 管理員分類管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 取得列表 / 詳情 | `categories:view` |
| 建立分類 | `categories:create` |
| 更新分類 | `categories:update` |
| 刪除分類 | `categories:delete` |
| 還原分類 | `categories:update` |

---

## 取得分類列表

**GET** `/api/admin/categories`

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選狀態，可選值：`active`、`inactive` |
| `include_deleted` | boolean | 否 | 傳入 `true` 時，同時回傳已軟刪除的分類 |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得分類列表",
  "data": [
    {
      "id": 1,
      "name": "企業徵才",
      "slug": "recruitment",
      "description": "企業徵才廣告分類",
      "sort_order": 1,
      "status": "active",
      "deleted_at": null
    }
  ]
}
```

---

## 取得分類詳情

**GET** `/api/admin/categories/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 分類 ID |

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `with_plans` | boolean | 否 | 傳入 `true` 時，同時回傳該分類下的廣告方案 |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得分類詳情",
  "data": {
    "id": 1,
    "name": "企業徵才",
    "slug": "recruitment",
    "description": "企業徵才廣告分類",
    "sort_order": 1,
    "status": "active"
  }
}
```

---

## 建立分類

**POST** `/api/admin/categories`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 是 | 分類名稱，需唯一 |
| `slug` | string | 否 | URL 識別碼，不傳則自動由 `name` 產生，需唯一 |
| `description` | string | 否 | 分類說明 |
| `sort_order` | integer | 否 | 排序序號，預設 `0` |
| `status` | string | 否 | 狀態，可選值：`active`、`inactive`，預設 `active` |

```json
{
  "name": "房產租售",
  "slug": "real-estate",
  "description": "房產租售廣告分類",
  "sort_order": 5,
  "status": "active"
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "分類建立成功",
  "data": {
    "id": 5,
    "name": "房產租售",
    "slug": "real-estate",
    "description": "房產租售廣告分類",
    "sort_order": 5,
    "status": "active"
  }
}
```

---

## 更新分類

**PUT** `/api/admin/categories/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 分類 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 否 | 分類名稱 |
| `slug` | string | 否 | URL 識別碼 |
| `description` | string | 否 | 分類說明 |
| `sort_order` | integer | 否 | 排序序號 |
| `status` | string | 否 | 狀態：`active`、`inactive` |

```json
{
  "name": "房產租售（更新）",
  "status": "inactive"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "分類更新成功",
  "data": {
    "id": 5,
    "name": "房產租售（更新）",
    "status": "inactive"
  }
}
```

---

## 刪除分類

**DELETE** `/api/admin/categories/{id}`

軟刪除分類（可還原）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 分類 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "分類刪除成功",
  "data": null
}
```

---

## 還原已刪除的分類

**POST** `/api/admin/categories/{id}/restore`

還原被軟刪除的分類。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 分類 ID（包含已刪除的） |

### Request Body

不需傳入任何 Body。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "分類還原成功",
  "data": {
    "id": 5,
    "name": "房產租售",
    "status": "active",
    "deleted_at": null
  }
}
```

### Response 400 - 分類未被刪除

```json
{
  "success": false,
  "message": "分類未被刪除，無需還原",
  "data": null
}
```
