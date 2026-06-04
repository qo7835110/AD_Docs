# 外部應用廣告 API

> 外部應用以 **API Key + API Secret** 認證，操作對象為綁定的 owner user 的廣告資料。
>
> 所有請求需在 Header 帶入：
> ```
> X-Api-Key: ext_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
> X-Api-Secret: <your-api-secret>
> ```
>
> 每個端點需對應的 **permission scope**（由管理員建立外部應用時設定）。

---

## 認證說明

| 認證方式 | 說明 |
|----------|------|
| `X-Api-Key` | 外部應用唯一識別碼，由管理員建立時系統自動產生 |
| `X-Api-Secret` | 對應的密鑰，**僅於建立或 rotate-secret 時以明文回傳一次** |

Middleware 驗證通過後，會將 owner user 注入 `auth:api` guard，後續行為與該使用者直接呼叫一般 User API 完全相同。

### 認證失敗回應

| 狀態碼 | 說明 | message |
|--------|------|---------|
| 401 | 缺少憑證 | `缺少 API 憑證` |
| 401 | API Key 無效 | `API Key 無效` |
| 401 | API Secret 不符 | `API Secret 無效` |
| 403 | 應用已停用 | `此應用已被停用` |
| 403 | 授權已過期 | `此應用授權已過期` |
| 403 | IP 不在白名單 | `IP 不在允許清單內` |
| 403 | 缺少操作權限 | `此應用無此操作權限` |
| 403 | 擁有者不存在 | `應用擁有者不存在` |

---

## Permission Scope 對照

| 端點 | 所需 scope |
|------|-----------|
| 取得廣告列表 | `ads.read` |
| 取得廣告詳情 | `ads.read` |
| 建立廣告 | `ads.create` |
| 更新廣告 | `ads.update` |
| 刪除廣告 | `ads.delete` |
| 提交廣告審核 | `ads.create` |

---

## 取得廣告列表

**GET** `/api/external/ads`

回傳 owner user 旗下的廣告，支援與 User API 相同的篩選參數。

### Request Headers

```
X-Api-Key: ext_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
X-Api-Secret: <secret>
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "成功取得廣告列表",
  "data": [
    {
      "id": 55,
      "title": "徵才廣告",
      "description": "誠徵資深工程師",
      "link_url": "https://partner.com/jobs/123",
      "image_path": null,
      "status": "draft",
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
        "id": 1
      },
      "order_item_id": 12,
      "created_at": "2026-06-01 12:00:00",
      "updated_at": "2026-06-01 12:00:00"
    }
  ]
}
```

---

## 取得廣告詳情

**GET** `/api/external/ads/{id}`

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
    "id": 55,
    "title": "徵才廣告",
    "description": "誠徵資深工程師",
    "link_url": "https://partner.com/jobs/123",
    "image_path": null,
    "status": "draft",
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
      "id": 1
    },
    "order_item_id": 12,
    "created_at": "2026-06-01 12:00:00",
    "updated_at": "2026-06-01 12:00:00"
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "廣告不存在"
}
```

---

## 建立廣告

**POST** `/api/external/ads`

建立廣告草稿（`draft` 狀態），行為與 User API 的 `POST /api/ads` 完全相同。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `order_item_id` | integer | 是 | 訂單項目 ID（owner user 已付款的訂單項目） |
| `title` | string | 是 | 廣告標題 |
| `description` | string | 否 | 廣告說明 |
| `link_url` | string | 否 | 廣告連結 URL |

```json
{
  "order_item_id": 12,
  "title": "徵才廣告",
  "description": "誠徵資深工程師",
  "link_url": "https://partner.com/jobs/123"
}
```

### Response 201 - 建立成功

```json
{
  "success": true,
  "message": "廣告草稿建立成功",
  "data": {
    "id": 55,
    "title": "徵才廣告",
    "description": "誠徵資深工程師",
    "link_url": "https://partner.com/jobs/123",
    "image_path": null,
    "status": "draft",
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
      "id": 1
    },
    "order_item_id": 12,
    "created_at": "2026-06-01 12:00:00",
    "updated_at": "2026-06-01 12:00:00"
  }
}
```

---

## 更新廣告

**PUT** `/api/external/ads/{id}`

僅限 `draft` 或 `rejected` 狀態的廣告可更新，行為與 User API 的 `PUT /api/ads/{id}` 完全相同。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Request Body

欄位同建立廣告，皆為選填。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告更新成功",
  "data": {
    "id": 55,
    "title": "修改後的標題",
    "description": "誠徵資深工程師",
    "link_url": "https://partner.com/jobs/123",
    "image_path": null,
    "status": "draft",
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
      "id": 1
    },
    "order_item_id": 12,
    "created_at": "2026-06-01 12:00:00",
    "updated_at": "2026-06-01 12:05:00"
  }
}
```

---

## 刪除廣告

**DELETE** `/api/external/ads/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告刪除成功"
}
```

---

## 提交廣告審核

**POST** `/api/external/ads/{id}/submit`

將廣告從 `draft` 狀態提交為 `pending_review`，待管理員審核。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 廣告 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "廣告已提交審核",
  "data": {
    "id": 55,
    "title": "徵才廣告",
    "description": "誠徵資深工程師",
    "link_url": "https://partner.com/jobs/123",
    "image_path": null,
    "status": "pending_review",
    "review": {
      "rejection_reason": null,
      "reviewed_by": null,
      "reviewer_name": null,
      "submitted_at": "2026-06-01 12:10:00",
      "approved_at": null
    },
    "schedule": {
      "starts_at": null,
      "expires_at": null
    },
    "owner": {
      "type": "User",
      "id": 1
    },
    "order_item_id": 12,
    "created_at": "2026-06-01 12:00:00",
    "updated_at": "2026-06-01 12:10:00"
  }
}
```

---

## 測試憑證（開發環境）

執行以下指令產生測試資料：

```bash
php artisan db:seed --class=ExternalAppSeeder
```

產生後終端機會顯示可直接使用的 `X-Api-Key` 與 `X-Api-Secret`，以及範例 `curl` 指令。
