# 📁 Categories — 廣告分類管理 API

> **分類功能分為兩組路由：**
> - **會員端** (`auth:api`)：僅可讀取分類列表與詳情
> - **管理員端** (`auth:admin`)：完整 CRUD（含軟刪除與還原）
>
> `include_deleted` 參數僅管理員可使用。

---

## 端點索引

### 會員端（auth:api）

| 方法  | 路徑                     | 說明     | Guard |
| ----- | ------------------------ | -------- | ----- |
| `GET` | `/api/categories`        | 分類列表 | api   |
| `GET` | `/api/categories/{id}`   | 分類詳情 | api   |

### 管理員端（auth:admin）

| 方法     | 路徑                                  | 說明             | Guard |
| -------- | ------------------------------------- | ---------------- | ----- |
| `GET`    | `/api/admin/categories`               | 分類列表         | admin |
| `POST`   | `/api/admin/categories`               | 建立分類         | admin |
| `GET`    | `/api/admin/categories/{id}`          | 分類詳情         | admin |
| `PUT`    | `/api/admin/categories/{id}`          | 更新分類         | admin |
| `DELETE` | `/api/admin/categories/{id}`          | 刪除分類（軟刪除）| admin |
| `POST`   | `/api/admin/categories/{id}/restore`  | 還原已刪除的分類 | admin |

---

## GET `/api/categories` — 分類列表

> 會員端與管理員端共用同一個 Controller method，但行為略有不同。

**Query Parameters**

| 參數              | 類型      | 說明                                                 |
| ----------------- | --------- | ---------------------------------------------------- |
| `status`          | `string`  | 篩選狀態：`active` / `inactive`                      |
| `include_deleted` | `boolean` | 是否包含軟刪除的分類（⚠️ 僅管理員 guard 有效）       |

**請求範例**

```
GET /api/categories?status=active
GET /api/admin/categories?include_deleted=true
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得分類列表",
    "data": [
        {
            "id": 1,
            "name": "會務管理",
            "slug": "meeting-management",
            "description": "適合企業會議室管理的廣告方案",
            "sort_order": 0,
            "status": "active",
            "created_at": "2026-01-01 00:00:00",
            "updated_at": "2026-01-01 00:00:00"
        }
    ]
}
```

---

## POST `/api/admin/categories` — 建立分類 🔒 Admin

**Request Body（JSON）**

| 欄位          | 類型      | 必填 | 說明                                  |
| ------------- | --------- | ---- | ------------------------------------- |
| `name`        | `string`  | ✅   | 分類名稱（max:255，唯一值）           |
| `slug`        | `string`  | ❌   | URL slug（自動從 name 產生，唯一值）  |
| `description` | `string`  | ❌   | 分類說明                              |
| `sort_order`  | `integer` | ❌   | 排序（預設 0）                        |
| `status`      | `string`  | ❌   | `active`（預設）/ `inactive`          |

**請求範例**

```json
{
    "name": "新型態廣告",
    "description": "創新廣告形式",
    "sort_order": 5,
    "status": "active"
}
```

**回應範例（201）**

```json
{
    "success": true,
    "message": "分類建立成功",
    "data": {
        "id": 5,
        "name": "新型態廣告",
        "slug": "xin-xing-tai-guang-gao",
        "description": "創新廣告形式",
        "sort_order": 5,
        "status": "active",
        "created_at": "2026-03-13 11:00:00",
        "updated_at": "2026-03-13 11:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                    |
| ------ | ----------------------- |
| `422`  | 驗證失敗（name 重複等） |

---

## GET `/api/categories/{id}` — 分類詳情

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 分類 ID |

**Query Parameters**

| 參數         | 類型      | 說明                                       |
| ------------ | --------- | ------------------------------------------ |
| `with_plans` | `boolean` | 是否同時載入該分類的廣告方案，預設 `false` |

**請求範例**

```
GET /api/categories/1?with_plans=true
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "成功取得分類詳情",
    "data": {
        "id": 1,
        "name": "會務管理",
        "slug": "meeting-management",
        "description": "適合企業會議室管理",
        "status": "active",
        "ad_plans": [
            { "id": 1, "name": "基礎方案", "price": "999.00" }
        ]
    }
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `404`  | 分類不存在 |

---

## PUT `/api/admin/categories/{id}` — 更新分類 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 分類 ID |

**Request Body（JSON）**（所有欄位選填，至少提供一個）

| 欄位          | 類型      | 說明                                  |
| ------------- | --------- | ------------------------------------- |
| `name`        | `string`  | 分類名稱（max:255，唯一值，排除自身） |
| `slug`        | `string`  | URL slug（唯一值，排除自身）          |
| `description` | `string`  | 分類說明                              |
| `sort_order`  | `integer` | 排序                                  |
| `status`      | `string`  | `active` / `inactive`                 |

**請求範例**

```json
{
    "name": "會議管理升級版",
    "status": "inactive"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "分類更新成功",
    "data": {
        "id": 1,
        "name": "會議管理升級版",
        "status": "inactive",
        "updated_at": "2026-03-13 12:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `404`  | 分類不存在 |
| `422`  | 驗證失敗   |

---

## DELETE `/api/admin/categories/{id}` — 刪除分類 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 分類 ID |

> ⚠️ 為軟刪除（Soft Delete），資料仍保留於資料庫，可透過 `/restore` 還原。

**回應範例（200）**

```json
{
    "success": true,
    "message": "分類刪除成功"
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `404`  | 分類不存在 |

---

## POST `/api/admin/categories/{id}/restore` — 還原已刪除的分類 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明                    |
| ---- | --------- | ----------------------- |
| `id` | `integer` | 分類 ID（包含已軟刪除） |

**回應範例（200）**

```json
{
    "success": true,
    "message": "分類還原成功",
    "data": {
        "id": 1,
        "name": "會務管理",
        "deleted_at": null
    }
}
```

**錯誤回應**

| 狀態碼 | 說明               |
| ------ | ------------------ |
| `400`  | 分類未被刪除       |
| `404`  | 分類不存在         |
        "updated_at": "2026-03-12 13:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                   |
| ------ | ---------------------- |
| `400`  | 分類未被刪除，無需還原 |
| `404`  | 分類不存在             |
