# 公開標籤瀏覽 API

> 無需任何認證 Token 即可存取。標籤用於廣告精準投放，前台可透過此 API 取得標籤樹，供訪客自行選擇所屬標籤（`visitor_tags`），結合廣告列表的語意搜尋使用。

---

## 取得啟用中的標籤樹

**GET** `/api/public/tags`

回傳所有 `is_active = true` 的標籤，以階層樹狀結構組織（Adjacency List 轉換）。根節點（`parent_id = null`）包含 `children` 陣列，葉節點（`is_targetable = true`）為可實際投放的具體標籤。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `type` | string | 否 | 依標籤類型篩選，例如 `interest`、`demographic`、`education_statuses`。省略時回傳所有類型 |

### Response 200 - 成功

```json
{
  "success": true,
  "message": null,
  "data": [
    {
      "id": 1,
      "name": "興趣",
      "raw_name": "Interest",
      "type": "interest",
      "is_targetable": false,
      "description": null,
      "children": [
        {
          "id": 2,
          "name": "科技",
          "raw_name": "Technology",
          "type": "interest",
          "is_targetable": false,
          "description": null,
          "children": [
            {
              "id": 5,
              "name": "人工智慧",
              "raw_name": "Artificial Intelligence",
              "type": "interest",
              "is_targetable": true,
              "description": null,
              "children": []
            }
          ]
        },
        {
          "id": 3,
          "name": "運動",
          "raw_name": "Sports",
          "type": "interest",
          "is_targetable": false,
          "description": null,
          "children": [
            {
              "id": 6,
              "name": "籃球",
              "raw_name": "Basketball",
              "type": "interest",
              "is_targetable": true,
              "description": null,
              "children": []
            }
          ]
        }
      ]
    },
    {
      "id": 10,
      "name": "教育程度",
      "raw_name": "Education Status",
      "type": "education_statuses",
      "is_targetable": false,
      "description": null,
      "children": [
        {
          "id": 11,
          "name": "大學",
          "raw_name": "College",
          "type": "education_statuses",
          "is_targetable": true,
          "description": null,
          "children": []
        }
      ]
    }
  ]
}
```

### 欄位說明

| 欄位 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 標籤 ID，可作為廣告列表 `visitor_tags` 的值 |
| `name` | string | 本地化顯示名稱（中文） |
| `raw_name` | string | 原始英文名稱 |
| `type` | string \| null | 標籤類型，用於分群（如 `interest`、`demographic`、`education_statuses`） |
| `is_targetable` | boolean | `true` 表示此為可投放的具體葉節點；`false` 表示為分類目錄節點，不可直接作為投放標籤 |
| `description` | string \| null | 標籤補充說明 |
| `children` | array | 子節點陣列，葉節點為空陣列 `[]` |

### 使用情境

1. **廣告投放標籤選擇**：廣告主在後台建立廣告時，從此樹選取 `is_targetable = true` 的標籤，設定 Target（命中）或 Exclude（排除）規則。

2. **前台訪客標籤偏好**：前台在展示廣告前，引導訪客選擇所屬標籤（僅選 `is_targetable = true` 的葉節點），再將 ID 陣列作為 `visitor_tags` 傳入 `GET /api/public/ads`，觸發語意精準投放。

   ```
   GET /api/public/ads?visitor_tags=5,6,11
   ```
