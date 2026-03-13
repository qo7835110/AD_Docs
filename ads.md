# 📢 Ads — 廣告管理 API

> **廣告功能分為兩組路由：**
> - **使用者端** (`auth:api`)：建立草稿、編輯、上傳圖片、提交審核、刪除
> - **管理員端** (`auth:admin`)：瀏覽所有廣告、審核通過/拒絕、上架/下架

---

## 端點索引

### 使用者端（auth:api 🔒）

| 方法     | 路徑                    | 說明         |
| -------- | ----------------------- | ------------ |
| `GET`    | `/api/ads`              | 我的廣告列表 |
| `POST`   | `/api/ads`              | 建立廣告草稿 |
| `GET`    | `/api/ads/{id}`         | 廣告詳情     |
| `PUT`    | `/api/ads/{id}`         | 更新廣告     |
| `DELETE` | `/api/ads/{id}`         | 刪除廣告     |
| `POST`   | `/api/ads/{id}/image`   | 上傳廣告圖片 |
| `POST`   | `/api/ads/{id}/submit`  | 提交審核     |

### 管理員端（auth:admin 🔒）

| 方法   | 路徑                              | 說明         |
| ------ | --------------------------------- | ------------ |
| `GET`  | `/api/admin/ads`                  | 所有廣告列表 |
| `GET`  | `/api/admin/ads/{id}`             | 廣告詳情     |
| `POST` | `/api/admin/ads/{id}/approve`     | 審核通過     |
| `POST` | `/api/admin/ads/{id}/reject`      | 審核拒絕     |
| `POST` | `/api/admin/ads/{id}/activate`    | 上架廣告     |
| `POST` | `/api/admin/ads/{id}/deactivate`  | 下架廣告     |

---

## 廣告狀態生命週期

```
draft → pending_review → approved → active → inactive
                       ↘ rejected → (可重新編輯後再提交)
                                     draft → pending_review → ...
```

| 狀態               | 說明                     |
| ------------------ | ------------------------ |
| `draft`            | 草稿（可編輯）           |
| `pending_review`   | 待審核                   |
| `approved`         | 審核通過（待上架）       |
| `rejected`         | 審核拒絕（可重新編輯）   |
| `active`           | 已上架（展示中）         |
| `inactive`         | 已下架                   |
| `expired`          | 已過期                   |

---

## 使用者端 API

### GET `/api/ads` — 我的廣告列表 🔒

僅回傳目前登入使用者擁有的廣告。

**Query Parameters**

| 參數     | 類型     | 說明                                   |
| -------- | -------- | -------------------------------------- |
| `status` | `string` | 篩選狀態（見上方狀態值）               |

**請求範例**

```
GET /api/ads?status=draft
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得廣告列表",
    "data": [
        {
            "id": 1,
            "title": "春季促銷廣告",
            "status": "draft",
            "created_at": "2026-03-13 11:00:00"
        }
    ]
}
```

---

### POST `/api/ads` — 建立廣告草稿 🔒

> ⚠️ **IDOR 防護：** 系統會驗證 `order_item_id` 對應的訂單是否屬於目前登入使用者。

**Request Body（JSON）**

| 欄位            | 類型      | 必填 | 說明                               |
| --------------- | --------- | ---- | ---------------------------------- |
| `order_item_id` | `integer` | ✅   | 對應的訂單項目 ID（需存在於資料庫） |
| `title`         | `string`  | ✅   | 廣告標題（max:255）                |
| `description`   | `string`  | ❌   | 廣告說明                           |
| `link_url`      | `string`  | ❌   | 點擊連結 URL（需為合法 URL，max:2048）|

**請求範例**

```json
{
    "order_item_id": 1,
    "title": "春季促銷廣告",
    "description": "限時優惠，全館 5 折",
    "link_url": "https://example.com/promo"
}
```

**回應範例（201）**

```json
{
    "success": true,
    "message": "廣告草稿建立成功",
    "data": {
        "id": 1,
        "title": "春季促銷廣告",
        "description": "限時優惠，全館 5 折",
        "link_url": "https://example.com/promo",
        "status": "draft",
        "owner_id": 1,
        "order_item_id": 1,
        "created_at": "2026-03-13 11:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                         |
| ------ | ---------------------------- |
| `403`  | 該訂單項目不屬於您           |
| `422`  | 驗證失敗                     |

---

### GET `/api/ads/{id}` — 廣告詳情 🔒

> 僅限廣告擁有者可檢視。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

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
        "order_item": { ... },
        "files": [ ... ],
        "reviewer": null,
        "created_at": "2026-03-13 11:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明               |
| ------ | ------------------ |
| `403`  | 無權限檢視此廣告   |
| `404`  | 廣告不存在         |

---

### PUT `/api/ads/{id}` — 更新廣告 🔒

> 僅限 `draft` 或 `rejected` 狀態的廣告可以編輯。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**Request Body（JSON）**

| 欄位          | 類型     | 必填 | 說明                                 |
| ------------- | -------- | ---- | ------------------------------------ |
| `title`       | `string` | ❌   | 廣告標題（max:255）                  |
| `description` | `string` | ❌   | 廣告說明                             |
| `link_url`    | `string` | ❌   | 點擊連結 URL（需為合法 URL，max:2048）|

**請求範例**

```json
{
    "title": "修改後的標題",
    "description": "修改後的說明"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告更新成功",
    "data": {
        "id": 1,
        "title": "修改後的標題",
        "description": "修改後的說明",
        "status": "draft"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                             |
| ------ | -------------------------------- |
| `400`  | 目前狀態不允許編輯               |
| `403`  | 無權限編輯此廣告                 |
| `404`  | 廣告不存在                       |
| `422`  | 驗證失敗                         |

---

### DELETE `/api/ads/{id}` — 刪除廣告 🔒

> 僅限 `draft`、`rejected` 或 `inactive` 狀態的廣告可以刪除。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告刪除成功"
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 目前狀態不允許刪除     |
| `403`  | 無權限刪除此廣告       |
| `404`  | 廣告不存在             |

---

### POST `/api/ads/{id}/image` — 上傳廣告圖片 🔒

> 僅限 `draft` 或 `rejected` 狀態。使用 `multipart/form-data` 上傳。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**Request Body（multipart/form-data）**

| 欄位    | 類型   | 必填 | 說明                                       |
| ------- | ------ | ---- | ------------------------------------------ |
| `image` | `file` | ✅   | 圖片檔案（jpeg/jpg/png/gif/webp，max:2MB） |

**回應範例（200）**

```json
{
    "success": true,
    "message": "圖片上傳成功",
    "data": {
        "id": 1,
        "title": "春季促銷廣告",
        "files": [
            {
                "id": 1,
                "path": "ads/1/image.jpg",
                "mime_type": "image/jpeg"
            }
        ]
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 目前狀態不允許上傳     |
| `403`  | 無權限                 |
| `404`  | 廣告不存在             |
| `422`  | 驗證失敗（檔案格式等） |

---

### POST `/api/ads/{id}/submit` — 提交審核 🔒

> 僅限 `draft` 或 `rejected` 狀態。提交後狀態變為 `pending_review`。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

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

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 目前狀態不允許提交     |
| `403`  | 無權限                 |
| `404`  | 廣告不存在             |

---

## 管理員端 API

### GET `/api/admin/ads` — 所有廣告列表 🔒 Admin

可瀏覽系統中所有使用者的廣告。

**Query Parameters**

| 參數        | 類型     | 說明                     |
| ----------- | -------- | ------------------------ |
| `status`    | `string` | 篩選狀態（見上方狀態值） |
| `date_from` | `date`   | 建立日期起（`Y-m-d`）   |
| `date_to`   | `date`   | 建立日期迄（`Y-m-d`）   |

**請求範例**

```
GET /api/admin/ads?status=pending_review
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得廣告列表",
    "data": [
        {
            "id": 1,
            "title": "春季促銷廣告",
            "status": "pending_review",
            "owner_id": 1,
            "created_at": "2026-03-13 11:00:00"
        }
    ]
}
```

---

### GET `/api/admin/ads/{id}` — 廣告詳情 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得廣告詳情",
    "data": {
        "id": 1,
        "title": "春季促銷廣告",
        "description": "限時優惠",
        "link_url": "https://example.com/promo",
        "status": "pending_review",
        "order_item": { ... },
        "owner": { "id": 1, "name": "王小明" },
        "files": [ ... ],
        "reviewer": null
    }
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `404`  | 廣告不存在 |

---

### POST `/api/admin/ads/{id}/approve` — 審核通過 🔒 Admin

> 僅限 `pending_review` 狀態。通過後狀態變為 `approved`。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告審核通過",
    "data": {
        "id": 1,
        "status": "approved",
        "reviewed_by": 1,
        "reviewed_at": "2026-03-13 15:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 目前狀態不允許審核通過 |
| `404`  | 廣告不存在             |

---

### POST `/api/admin/ads/{id}/reject` — 審核拒絕 🔒 Admin

> 僅限 `pending_review` 狀態。拒絕後狀態變為 `rejected`，使用者可修改後重新提交。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**Request Body（JSON）**

| 欄位     | 類型     | 必填 | 說明                   |
| -------- | -------- | ---- | ---------------------- |
| `reason` | `string` | ✅   | 拒絕原因（max:1000）   |

**請求範例**

```json
{
    "reason": "圖片不符合規範，請重新上傳"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告審核已拒絕",
    "data": {
        "id": 1,
        "status": "rejected",
        "rejection_reason": "圖片不符合規範，請重新上傳",
        "reviewed_by": 1,
        "reviewed_at": "2026-03-13 15:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 目前狀態不允許審核拒絕 |
| `404`  | 廣告不存在             |
| `422`  | 驗證失敗（reason 必填）|

---

### POST `/api/admin/ads/{id}/activate` — 上架廣告 🔒 Admin

> 僅限 `approved` 狀態。上架後狀態變為 `active`。
> ⚠️ 系統會驗證對應訂單是否已付款，未付款的廣告無法上架。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告已上架",
    "data": {
        "id": 1,
        "status": "active"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                               |
| ------ | ---------------------------------- |
| `400`  | 目前狀態不允許上架 / 訂單尚未付款  |
| `404`  | 廣告不存在                         |

---

### POST `/api/admin/ads/{id}/deactivate` — 下架廣告 🔒 Admin

> 僅限 `active` 狀態。下架後狀態變為 `inactive`。

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 廣告 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告已下架",
    "data": {
        "id": 1,
        "status": "inactive"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 目前狀態不允許下架     |
| `404`  | 廣告不存在             |
