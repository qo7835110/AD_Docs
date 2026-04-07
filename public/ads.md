# 公開廣告瀏覽與追蹤 API

> 以下 API 無需任何認證 Token 即可存取，供廣告展示前台使用。

---

## 取得上架廣告列表

**GET** `/api/public/ads`

僅回傳狀態為 `active`（上架中）的廣告，支援關鍵字搜尋、分類篩選與分頁。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `keyword` | string | 否 | 搜尋關鍵字（比對廣告標題或說明） |
| `category_id` | integer | 否 | 依分類 ID 篩選 |
| `sort_by` | string | 否 | 排序欄位，可選值：`created_at`、`starts_at`、`title`，預設 `created_at` |
| `sort_dir` | string | 否 | 排序方向，可選值：`asc`、`desc`，預設 `desc` |
| `per_page` | integer | 否 | 每頁筆數，範圍 1～100，預設 `15` |
| `page` | integer | 否 | 頁碼，預設 `1` |

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
        "status": "active",
        "category_id": 2,
        "starts_at": "2025-03-01",
        "ends_at": "2025-03-31",
        "files": [
          {
            "id": 10,
            "url": "https://example.com/storage/ads/ad_1.jpg"
          }
        ],
        "created_at": "2025-02-20T10:00:00+08:00"
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

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

僅回傳狀態為 `active` 的廣告；如廣告不存在或已下架，回傳 404。

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
    "status": "active",
    "category_id": 2,
    "starts_at": "2025-03-01",
    "ends_at": "2025-03-31",
    "files": [],
    "created_at": "2025-02-20T10:00:00+08:00"
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

### Response 201 - 成功

```json
{
  "success": true,
  "message": "曝光已記錄",
  "data": {
    "recorded": true
  }
}
```

若為重複曝光（短時間內同 IP 再次觸發），`recorded` 回傳 `false`，`message` 為 `"重複曝光，已略過"`：

```json
{
  "success": true,
  "message": "重複曝光，已略過",
  "data": {
    "recorded": false
  }
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
