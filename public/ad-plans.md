# 公開廣告方案 API

> 以下 API 無需任何認證 Token 即可存取，供前台展示使用。

---

## 取得廣告方案列表

**GET** `/api/ad-plans`

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選狀態，可選值：`active`、`inactive` |
| `category` | string | 否 | 篩選分類，可傳分類 ID（如 `1`）或分類 slug（如 `meeting-management`） |

### Response 200 - 成功

```json
{
  "success": true,
  "message": null,
  "data": [
    {
      "id": 1,
      "category_id": 2,
      "category": "企業方案",
      "name": "基礎方案",
      "description": "適合小型企業",
      "ad_limit": 10,
      "status": "active",
      "features": ["基本曝光", "7x24 客服"],
      "image": {
        "id": 5,
        "file_name": "plan_basic.jpg",
        "url": "https://example.com/storage/plan_basic.jpg"
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
  ]
}
```

---

## 取得廣告方案詳情

**GET** `/api/ad-plans/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告方案 ID |

### Response 200 - 成功

回傳格式同方案列表中的單筆資料結構。

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "廣告方案不存在",
  "data": null
}
```

---

## 取得方案的選項列表

**GET** `/api/ad-plans/{planId}/options`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `planId` | integer | 廣告方案 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": null,
  "data": [
    {
      "id": 1,
      "ad_plan_id": 1,
      "name": "3個月方案",
      "description": "季度優惠方案",
      "duration_days": 90,
      "price": 2699,
      "valid_start_date": "2025-01-01",
      "valid_end_date": "2025-12-31",
      "sort_order": 1
    }
  ]
}
```

---

## 取得單一方案選項詳情

**GET** `/api/plan-options/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 方案選項 ID |

### Response 200 - 成功

回傳格式同選項列表中的單筆資料結構。

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "方案選項不存在",
  "data": null
}
```
