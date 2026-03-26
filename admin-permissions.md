# 🔑 Admin Permissions — 管理員權限管理 API

> **Prefix：** `/api/admin`
> **Guard：** `admin`（`admins` 表）
> **認證方式：** 所有端點需要 `Authorization: Bearer {admin_token}`
> **權限需求：** 需具備 `admin_permissions` 模組對應動作的權限，超級管理員自動跳過檢查

---

## 概述

管理員權限系統提供**模組式 CRUD 權限控制**，可精細管理每位管理員對各 API 模組的存取權限，並自動記錄所有權限變更日誌。

### 核心概念

| 概念 | 說明 |
| ---- | ---- |
| **超級管理員** | `is_super = true`，跳過所有權限檢查，可存取全部 API |
| **一般管理員** | `is_super = false`，僅能存取被授權的 模組 × 動作 組合 |
| **免權限路由** | 管理員帳號相關路由（login、logout、me、refresh、change-password）不受權限限制 |

### 可管理的模組

| 模組 | 說明 | 對應路由 |
| ---- | ---- | -------- |
| `ads` | 廣告審核管理 | `/api/admin/ads/*` |
| `categories` | 分類管理 | `/api/admin/categories/*` |
| `ad_plans` | 廣告方案管理 | `/api/admin/ad-plans/*`（不含 options） |
| `plan_options` | 方案選項管理 | `/api/admin/ad-plans/*/options/*` |
| `category_permissions` | 使用者分類購買權限 | `/api/admin/users/*/category-permissions/*` |
| `admin_permissions` | 管理員權限管理 | `/api/admin/admins/*/permissions/*` |

### 可操作的動作

| 動作 | 說明 | 對應 HTTP 方法 |
| ---- | ---- | -------------- |
| `view` | 檢視 / 列表 | `GET` |
| `create` | 新增 | `POST` |
| `update` | 修改 / 批次設定 | `PUT`、`POST`（approve/reject 等） |
| `delete` | 刪除 / 撤銷 | `DELETE` |

### 路由與權限對應

```
GET    /api/admin/ads                 → ads, view
GET    /api/admin/ads/{id}            → ads, view
POST   /api/admin/ads/{id}/approve    → ads, update
POST   /api/admin/ads/{id}/reject     → ads, update
POST   /api/admin/ads/{id}/activate   → ads, update
POST   /api/admin/ads/{id}/deactivate → ads, update

GET    /api/admin/categories          → categories, view
POST   /api/admin/categories          → categories, create
GET    /api/admin/categories/{id}     → categories, view
PUT    /api/admin/categories/{id}     → categories, update
DELETE /api/admin/categories/{id}     → categories, delete
POST   /api/admin/categories/{id}/restore → categories, update

POST   /api/admin/ad-plans            → ad_plans, create
PUT    /api/admin/ad-plans/{id}       → ad_plans, update
DELETE /api/admin/ad-plans/{id}       → ad_plans, delete

POST   /api/admin/ad-plans/{planId}/options → plan_options, create
PUT    /api/admin/ad-plans/options/{id}     → plan_options, update
DELETE /api/admin/ad-plans/options/{id}     → plan_options, delete

GET    /api/admin/users/{userId}/category-permissions       → category_permissions, view
PUT    /api/admin/users/{userId}/category-permissions       → category_permissions, update
POST   /api/admin/users/{userId}/category-permissions/batch → category_permissions, create
DELETE /api/admin/users/{userId}/category-permissions/{id}  → category_permissions, delete
```

---

## 端點索引

| 方法     | 路徑                                              | 說明                   | 權限                        |
| -------- | ------------------------------------------------- | ---------------------- | --------------------------- |
| `GET`    | `/api/admin/permissions/modules`                  | 取得可用模組與動作列表 | `admin_permissions, view`   |
| `GET`    | `/api/admin/admins/{adminId}/permissions`          | 取得管理員權限列表     | `admin_permissions, view`   |
| `POST`   | `/api/admin/admins/{adminId}/permissions`          | 授予管理員權限         | `admin_permissions, create` |
| `DELETE` | `/api/admin/admins/{adminId}/permissions`          | 撤銷管理員權限         | `admin_permissions, delete` |
| `PUT`    | `/api/admin/admins/{adminId}/permissions/batch`    | 批次設定管理員權限     | `admin_permissions, update` |
| `GET`    | `/api/admin/admins/{adminId}/permissions/logs`     | 取得權限變更日誌       | `admin_permissions, view`   |

---

## GET `/api/admin/permissions/modules` — 取得可用模組與動作列表 🔒

> 回傳系統中所有可分配的模組名稱與動作名稱，供前端渲染權限設定介面使用。

**回應範例（200）**

```json
{
    "success": true,
    "message": "取得成功",
    "data": {
        "modules": [
            "ads",
            "categories",
            "ad_plans",
            "plan_options",
            "category_permissions",
            "admin_permissions"
        ],
        "actions": [
            "view",
            "create",
            "update",
            "delete"
        ]
    }
}
```

---

## GET `/api/admin/admins/{adminId}/permissions` — 取得管理員權限列表 🔒

**Path Parameters**

| 參數      | 類型      | 說明       |
| --------- | --------- | ---------- |
| `adminId` | `integer` | 管理員 ID  |

**回應範例（200）**

```json
{
    "success": true,
    "message": "取得成功",
    "data": {
        "permissions": [
            {
                "id": 1,
                "admin_id": 2,
                "module": "ads",
                "action": "view",
                "granted_by": 1,
                "grantor": {
                    "id": 1,
                    "name": "Super Admin"
                },
                "created_at": "2026-03-26 10:00:00",
                "updated_at": "2026-03-26 10:00:00"
            },
            {
                "id": 2,
                "admin_id": 2,
                "module": "ads",
                "action": "update",
                "granted_by": 1,
                "grantor": {
                    "id": 1,
                    "name": "Super Admin"
                },
                "created_at": "2026-03-26 10:00:00",
                "updated_at": "2026-03-26 10:00:00"
            }
        ]
    }
}
```

---

## POST `/api/admin/admins/{adminId}/permissions` — 授予管理員權限 🔒

**Path Parameters**

| 參數      | 類型      | 說明       |
| --------- | --------- | ---------- |
| `adminId` | `integer` | 管理員 ID  |

**Request Body（JSON）**

| 欄位     | 類型     | 必填 | 說明                                         |
| -------- | -------- | ---- | -------------------------------------------- |
| `module` | `string` | ✅   | 模組名稱（見可用模組列表）                   |
| `action` | `string` | ✅   | 動作名稱：`view`、`create`、`update`、`delete` |
| `reason` | `string` | ❌   | 授權原因（max:500）                          |

**請求範例**

```json
{
    "module": "ads",
    "action": "view",
    "reason": "授予廣告瀏覽權限"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "權限授予成功",
    "data": {
        "permission": {
            "id": 1,
            "admin_id": 2,
            "module": "ads",
            "action": "view",
            "granted_by": 1,
            "grantor": {
                "id": 1,
                "name": "Super Admin"
            },
            "created_at": "2026-03-26 10:00:00",
            "updated_at": "2026-03-26 10:00:00"
        }
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                          |
| ------ | ----------------------------- |
| `400`  | 無法修改超級管理員的權限      |
| `403`  | 權限不足                      |
| `422`  | 驗證失敗（無效的模組或動作）  |

> ⚠️ 若該權限已存在則更新 `granted_by`，不會產生重複記錄。

---

## DELETE `/api/admin/admins/{adminId}/permissions` — 撤銷管理員權限 🔒

**Path Parameters**

| 參數      | 類型      | 說明       |
| --------- | --------- | ---------- |
| `adminId` | `integer` | 管理員 ID  |

**Request Body（JSON）**

| 欄位     | 類型     | 必填 | 說明             |
| -------- | -------- | ---- | ---------------- |
| `module` | `string` | ✅   | 模組名稱         |
| `action` | `string` | ✅   | 動作名稱         |
| `reason` | `string` | ❌   | 撤銷原因（max:500） |

**請求範例**

```json
{
    "module": "ads",
    "action": "view",
    "reason": "取消廣告瀏覽權限"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "權限已撤銷"
}
```

**錯誤回應**

| 狀態碼 | 說明                          |
| ------ | ----------------------------- |
| `400`  | 無法修改超級管理員的權限      |
| `403`  | 權限不足                      |
| `422`  | 驗證失敗                      |

---

## PUT `/api/admin/admins/{adminId}/permissions/batch` — 批次設定管理員權限 🔒

> ⚠️ **此操作會覆蓋（清除並重建）該管理員的所有現有權限。**

**Path Parameters**

| 參數      | 類型      | 說明       |
| --------- | --------- | ---------- |
| `adminId` | `integer` | 管理員 ID  |

**Request Body（JSON）**

| 欄位                     | 類型     | 必填 | 說明                |
| ------------------------ | -------- | ---- | ------------------- |
| `permissions`            | `array`  | ✅   | 權限陣列（min: 1）  |
| `permissions.*.module`   | `string` | ✅   | 模組名稱            |
| `permissions.*.action`   | `string` | ✅   | 動作名稱            |
| `reason`                 | `string` | ❌   | 批次操作原因（max:500） |

**請求範例**

```json
{
    "permissions": [
        { "module": "categories", "action": "view" },
        { "module": "categories", "action": "create" },
        { "module": "categories", "action": "update" },
        { "module": "ads", "action": "view" },
        { "module": "ads", "action": "update" }
    ],
    "reason": "設定分類管理+廣告審核權限"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "批次權限設定成功",
    "data": {
        "permissions": [
            {
                "id": 10,
                "admin_id": 2,
                "module": "categories",
                "action": "view",
                "granted_by": 1,
                "grantor": { "id": 1, "name": "Super Admin" },
                "created_at": "2026-03-26 10:00:00",
                "updated_at": "2026-03-26 10:00:00"
            },
            {
                "id": 11,
                "admin_id": 2,
                "module": "categories",
                "action": "create",
                "granted_by": 1,
                "grantor": { "id": 1, "name": "Super Admin" },
                "created_at": "2026-03-26 10:00:00",
                "updated_at": "2026-03-26 10:00:00"
            }
        ]
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                          |
| ------ | ----------------------------- |
| `400`  | 無法修改超級管理員的權限      |
| `403`  | 權限不足                      |
| `422`  | 驗證失敗                      |

---

## GET `/api/admin/admins/{adminId}/permissions/logs` — 取得權限變更日誌 🔒

> 回傳指定管理員的所有權限變更紀錄，依建立時間降冪排列。

**Path Parameters**

| 參數      | 類型      | 說明       |
| --------- | --------- | ---------- |
| `adminId` | `integer` | 管理員 ID  |

**回應範例（200）**

```json
{
    "success": true,
    "message": "取得成功",
    "data": {
        "logs": [
            {
                "id": 3,
                "admin_id": 2,
                "module": "categories",
                "action": "view",
                "permission": "grant",
                "operated_by": 1,
                "operator": {
                    "id": 1,
                    "name": "Super Admin"
                },
                "reason": "設定分類管理權限",
                "ip_address": "127.0.0.1",
                "created_at": "2026-03-26 10:05:00"
            },
            {
                "id": 2,
                "admin_id": 2,
                "module": "ads",
                "action": "view",
                "permission": "revoke",
                "operated_by": 1,
                "operator": {
                    "id": 1,
                    "name": "Super Admin"
                },
                "reason": "移除廣告權限",
                "ip_address": "127.0.0.1",
                "created_at": "2026-03-26 10:02:00"
            }
        ]
    }
}
```

---

## 資料結構

### admin_permissions 表

| 欄位         | 類型         | 說明                  |
| ------------ | ------------ | --------------------- |
| `id`         | `bigint`     | 主鍵                  |
| `admin_id`   | `bigint`     | 管理員 ID（FK→admins）|
| `module`     | `varchar(50)` | 模組名稱             |
| `action`     | `varchar(20)` | 動作名稱             |
| `granted_by` | `bigint`     | 授權者 ID（可為 null）|
| `created_at` | `timestamp`  | 建立時間              |
| `updated_at` | `timestamp`  | 更新時間              |

> 唯一索引：`(admin_id, module, action)`

### admin_permission_logs 表

| 欄位          | 類型          | 說明                          |
| ------------- | ------------- | ----------------------------- |
| `id`          | `bigint`      | 主鍵                          |
| `admin_id`    | `bigint`      | 被操作的管理員 ID（FK→admins）|
| `module`      | `varchar(50)` | 模組名稱                      |
| `action`      | `varchar(20)` | 動作名稱                      |
| `permission`  | `varchar(10)` | `grant`（授予）/ `revoke`（撤銷）|
| `operated_by` | `bigint`      | 操作者管理員 ID（FK→admins）  |
| `reason`      | `text`        | 操作原因（可為 null）         |
| `ip_address`  | `varchar(45)` | 操作者 IP（可為 null）        |
| `created_at`  | `timestamp`   | 操作時間                      |

> 日誌為不可修改記錄（無 `updated_at`）。

### admins 表（新增欄位）

| 欄位       | 類型      | 說明                                 |
| ---------- | --------- | ------------------------------------ |
| `is_super` | `boolean` | 是否為超級管理員（預設 `false`）      |

---

## Middleware 機制

權限檢查透過 `admin.permission` 中介軟體實現，套用在各管理員路由上：

```php
Route::get('/ads', [AdController::class, 'adminIndex'])
    ->middleware('admin.permission:ads,view');
```

**檢查流程：**

1. 檢查是否已認證 → 未認證回傳 `401`
2. 檢查 `is_super` → 超級管理員直接放行
3. 查詢 `admin_permissions` 表 → 有對應記錄放行，否則回傳 `403`

**403 回應格式：**

```json
{
    "success": false,
    "message": "權限不足，無法執行此操作"
}
```

---

## 使用情境範例

### 建立一個只能管理分類的管理員

```bash
# 1. 以超級管理員登入取得 token
POST /api/admin/login
{ "email": "admin@gmail.com", "password": "123456789" }

# 2. 批次設定新管理員的權限
PUT /api/admin/admins/2/permissions/batch
{
    "permissions": [
        { "module": "categories", "action": "view" },
        { "module": "categories", "action": "create" },
        { "module": "categories", "action": "update" },
        { "module": "categories", "action": "delete" }
    ],
    "reason": "設定為分類管理專員"
}
```

### 查看某管理員的歷史權限變更

```bash
GET /api/admin/admins/2/permissions/logs
# → 回傳所有 grant / revoke 紀錄，含操作者、原因、IP
```
