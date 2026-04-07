# 分類查詢 API (會員)

> 以下 API 需在 Header 帶入會員 JWT Token：
> `Authorization: Bearer {token}`

---

## 取得分類列表

**GET** `/api/categories`

取得平台上的廣告分類清單，供使用者在建立訂單時選擇使用。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選狀態，可選值：`active`、`inactive` |

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
      "status": "active"
    },
    {
      "id": 2,
      "name": "會議管理",
      "slug": "meeting-management",
      "description": "會議管理類廣告",
      "sort_order": 2,
      "status": "active"
    }
  ]
}
```

---

## 取得分類詳情

**GET** `/api/categories/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 分類 ID |

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `with_plans` | boolean | 否 | 傳入 `true` 時，同時回傳該分類下的廣告方案列表 |

### Response 200 - 成功（不含方案）

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

### Response 200 - 成功（含方案，`?with_plans=true`）

```json
{
  "success": true,
  "message": "成功取得分類詳情",
  "data": {
    "id": 1,
    "name": "企業徵才",
    "slug": "recruitment",
    "status": "active",
    "ad_plans": [
      {
        "id": 1,
        "name": "基礎方案",
        "ad_limit": 10,
        "status": "active"
      }
    ]
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "分類不存在",
  "data": null
}
```
