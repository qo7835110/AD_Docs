# 公開廣告瀏覽與追蹤 API

> 以下 API 無需任何認證 Token 即可存取，供廣告展示前台使用。

---

## 取得上架廣告列表

**GET** `/api/public/ads`

僅回傳狀態為 `active`（上架中）的廣告，支援關鍵字搜尋、分類篩選、語意向量搜尋與分頁。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `keyword` | string | 否 | 搜尋關鍵字（比對廣告標題或說明） |
| `category_id` | integer | 否 | 依分類 ID 篩選 |
| `sort_by` | string | 否 | 排序欄位，可選值：`created_at`、`starts_at`、`title`，預設 `created_at` |
| `sort_dir` | string | 否 | 排序方向，可選值：`asc`、`desc`，預設 `desc` |
| `per_page` | integer | 否 | 每頁筆數，範圍 1～100，預設 `15` |
| `page` | integer | 否 | 頁碼，預設 `1` |
| `visitor_text` | string | 否 | 訪客自由描述文字（最多 500 字元），傳入後系統以 Qdrant 語意向量搜尋取代傳統關鍵字篩選 |
| `visitor_tags` | string \| array | 否 | 訪客所屬標籤 ID，可傳陣列或以逗號分隔的字串（例如 `"3,7,12"`）；系統依此比對廣告的 Target / Exclude 標籤設定 |

> **語意搜尋說明**：提供 `visitor_text` 或 `visitor_tags` 時，後端啟動 Qdrant 混合搜尋模式，以向量相似度與標籤行為（`behavior_type: 1` 命中、`-1` 排除）排序廣告。若 Qdrant 不可用，系統自動降級為傳統關鍵字模糊搜尋。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得廣告列表",
  "data": {
    "ads": [
      {
        "id": 1,
        "title": "春季促銷廣告",
        "description": "限時優惠，全館 5 折",
        "link_url": "https://example.com/promo",
        "image_path": "ads/ad_1.jpg",
        "status": "active",
        "review": {
          "rejection_reason": null,
          "reviewed_by": 2,
          "reviewer_name": "管理員 A",
          "submitted_at": "2025-02-18 09:00:00",
          "approved_at": "2025-02-19 11:30:00"
        },
        "schedule": {
          "starts_at": "2025-03-01 00:00:00",
          "expires_at": "2025-03-31 23:59:59"
        },
        "owner": {
          "type": "User",
          "id": 5,
          "name": "王小明",
          "email": "wang@example.com"
        },
        "order_item_id": 10,
        "order_item": null,
        "files": [
          {
            "id": 10,
            "url": "https://example.com/storage/ads/ad_1.jpg"
          }
        ],
        "created_at": "2025-02-20 10:00:00",
        "updated_at": "2025-02-21 08:00:00"
      }
    ],
    "pagination": {
      "current_page": 1,
      "last_page": 5,
      "per_page": 15,
      "total": 72
    }
  }
}
```

---

## 取得上架廣告詳情

**GET** `/api/public/ads/{id}`

僅回傳狀態為 `active` 的廣告；如廣告不存在或已下架，回傳 404。

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
    "status": "active",
    "review": {
      "rejection_reason": null,
      "reviewed_by": 2,
      "reviewer_name": "管理員 A",
      "submitted_at": "2025-02-18 09:00:00",
      "approved_at": "2025-02-19 11:30:00"
    },
    "schedule": {
      "starts_at": "2025-03-01 00:00:00",
      "expires_at": "2025-03-31 23:59:59"
    },
    "owner": {
      "type": "User",
      "id": 5,
      "name": "王小明",
      "email": "wang@example.com"
    },
    "order_item_id": 10,
    "order_item": null,
    "files": [],
    "created_at": "2025-02-20 10:00:00",
    "updated_at": "2025-02-21 08:00:00"
  }
}
```

### Response 404 - 廣告不存在或已下架

```json
{
  "success": false,
  "message": "廣告不存在或已下架",
  "data": null
}
```

---

## 記錄廣告曝光

**POST** `/api/public/ads/{id}/impression`

當廣告進入使用者視野時呼叫此 API。系統有防重複機制，短時間內同一 IP 的重複請求會被忽略。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

不需傳任何 Body，系統自動記錄請求者的 IP 與 User-Agent。

### Response 201 - 成功記錄

```json
{
  "success": true,
  "message": "曝光已記錄",
  "data": {
    "recorded": true
  }
}
```

### Response 201 - 重複曝光（已略過）

同 IP 短時間內再次觸發，`recorded` 回傳 `false`：

```json
{
  "success": true,
  "message": "重複曝光，已略過",
  "data": {
    "recorded": false
  }
}
```

### Response 404 - 廣告不存在或已下架

```json
{
  "success": false,
  "message": "廣告不存在或已下架",
  "data": null
}
```

---

## 記錄廣告點擊並導向目標網址

**GET** `/api/public/ads/{id}/click`

記錄點擊事件後，自動以 HTTP 302 導向該廣告的 `link_url`。

> **注意**：此 API 回傳的不是 JSON，而是 HTTP 302 Redirect。前端若以 `<a>` 標籤或 `window.location` 呼叫時行為正常；若以 `fetch` / `axios` 呼叫，需確認是否允許 Redirect 跟隨。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Response 302 - 導向成功

```
HTTP/1.1 302 Found
Location: https://example.com/promo
```

### Response 404 - 廣告不存在或已下架

```json
{
  "success": false,
  "message": "廣告不存在或已下架",
  "data": null
}
```
