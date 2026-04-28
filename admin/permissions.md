# 管理員權限管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`
>
> 本文件包含兩組權限系統：
> 1. **管理員操作權限**（Admin Permissions）：控管各管理員帳號對各模組的存取權限
> 2. **使用者分類購買權限**（Category Permissions）：控管特定使用者是否可購買特定分類的廣告方案

---

## 一、管理員操作權限 (Admin Permissions)

### 所需權限

| API | 所需權限 |
|-----|----------|
| 查看權限列表 / 日誌 | `admin_permissions:view` |
| 授予權限 | `admin_permissions:create` |
| 撤銷權限 | `admin_permissions:delete` |
| 批次設定 | `admin_permissions:update` |
| 查看模組清單 | `admin_permissions:view` |

### 可用模組與動作

| 模組名稱 | 說明 |
|----------|------|
| `ads` | 廣告審核管理 |
| `categories` | 分類管理 |
| `ad_plans` | 廣告方案管理 |
| `plan_options` | 方案選項管理 |
| `orders` | 訂單管理 |
| `category_permissions` | 使用者分類購買權限 |
| `admin_permissions` | 管理員權限管理（超級管理員） |
| `admins` | 管理員帳號管理 |

| 動作名稱 | 說明 |
|----------|------|
| `view` | 查看 |
| `create` | 建立 |
| `update` | 修改 |
| `delete` | 刪除 |

---

### 取得可用模組與動作列表

**GET** `/api/admin/permissions/modules`

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "modules": ["ads", "categories", "ad_plans", "plan_options", "orders", "category_permissions", "admin_permissions", "admins"],
    "actions": ["view", "create", "update", "delete"]
  }
}
```

---

### 取得指定管理員的權限列表

**GET** `/api/admin/admins/{adminId}/permissions`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `adminId` | integer | 管理員 ID |

### Response 200 - 成功

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
        "granted_by": 1
      },
      {
        "id": 2,
        "admin_id": 2,
        "module": "ads",
        "action": "update",
        "granted_by": 1
      }
    ]
  }
}
```

---

### 授予管理員權限

**POST** `/api/admin/admins/{adminId}/permissions`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `adminId` | integer | 管理員 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `module` | string | 是 | 模組名稱（見上方模組表） |
| `action` | string | 是 | 動作名稱（`view`、`create`、`update`、`delete`） |
| `reason` | string | 否 | 授予原因（記錄於日誌） |

```json
{
  "module": "ads",
  "action": "view",
  "reason": "新增廣告審核員"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "權限授予成功",
  "data": {
    "permission": {
      "id": 3,
      "admin_id": 2,
      "module": "ads",
      "action": "view",
      "granted_by": 1
    }
  }
}
```

### Response 400 - 操作失敗

```json
{
  "success": false,
  "message": "模組或動作無效",
  "data": null
}
```

---

### 撤銷管理員權限

**DELETE** `/api/admin/admins/{adminId}/permissions`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `adminId` | integer | 管理員 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `module` | string | 是 | 模組名稱 |
| `action` | string | 是 | 動作名稱 |
| `reason` | string | 否 | 撤銷原因 |

```json
{
  "module": "ads",
  "action": "update",
  "reason": "調整職責"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "權限已撤銷",
  "data": null
}
```

---

### 批次設定管理員權限

**PUT** `/api/admin/admins/{adminId}/permissions/batch`

覆蓋該管理員現有的所有權限（先清除所有舊權限，再新增傳入的權限）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `adminId` | integer | 管理員 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `permissions` | array | 是 | 權限陣列 |
| `permissions[].module` | string | 是 | 模組名稱 |
| `permissions[].action` | string | 是 | 動作名稱 |
| `reason` | string | 否 | 操作原因 |

```json
{
  "permissions": [
    {"module": "ads", "action": "view"},
    {"module": "ads", "action": "update"},
    {"module": "orders", "action": "view"}
  ],
  "reason": "重設審核員權限"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "批次權限設定成功",
  "data": {
    "permissions": [
      {"id": 10, "module": "ads", "action": "view"},
      {"id": 11, "module": "ads", "action": "update"},
      {"id": 12, "module": "orders", "action": "view"}
    ]
  }
}
```

---

### 取得管理員權限變更日誌

**GET** `/api/admin/admins/{adminId}/permissions/logs`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `adminId` | integer | 管理員 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "logs": [
      {
        "id": 1,
        "admin_id": 2,
        "module": "ads",
        "action": "view",
        "permission": "grant",
        "operated_by": 1,
        "reason": "新增廣告審核員",
        "ip_address": "192.168.1.1",
        "created_at": "2026-04-01T10:00:00+08:00"
      }
    ]
  }
}
```

---

## 二、使用者分類購買權限 (Category Permissions)

控管特定使用者是否可購買特定分類的廣告方案。預設情況下使用者可購買所有分類；設定 `deny` 後，該使用者無法購買對應分類的方案。

### 所需權限

| API | 所需權限 |
|-----|----------|
| 查看 | `category_permissions:view` |
| 設定（單一） | `category_permissions:update` |
| 批次設定 | `category_permissions:create` |
| 移除 | `category_permissions:delete` |

---

### 取得使用者的分類權限列表

**GET** `/api/admin/users/{userId}/category-permissions`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `userId` | integer | 使用者 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "permissions": [
      {
        "id": 1,
        "user_id": 5,
        "category_id": 2,
        "permission": "deny",
        "reason": "違反使用條款",
        "category": {
          "id": 2,
          "name": "會議管理",
          "slug": "meeting-management"
        }
      }
    ]
  }
}
```

---

### 設定使用者的單一分類權限

**PUT** `/api/admin/users/{userId}/category-permissions`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `userId` | integer | 使用者 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `category_id` | integer | 是 | 分類 ID |
| `permission` | string | 是 | 權限值，可選：`allow`（允許）、`deny`（拒絕） |
| `reason` | string | 否 | 設定原因 |

```json
{
  "category_id": 2,
  "permission": "deny",
  "reason": "違反使用條款"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "權限設定成功",
  "data": {
    "permission": {
      "id": 1,
      "user_id": 5,
      "category_id": 2,
      "permission": "deny",
      "reason": "違反使用條款",
      "category": {
        "id": 2,
        "name": "會議管理"
      }
    }
  }
}
```

---

### 批次設定使用者的分類權限

**POST** `/api/admin/users/{userId}/category-permissions/batch`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `userId` | integer | 使用者 ID |

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `permissions` | array | 是 | 權限設定陣列 |
| `permissions[].category_id` | integer | 是 | 分類 ID |
| `permissions[].permission` | string | 是 | 權限值：`allow`、`deny` |
| `permissions[].reason` | string | 否 | 設定原因 |

```json
{
  "permissions": [
    {"category_id": 1, "permission": "allow"},
    {"category_id": 2, "permission": "deny", "reason": "違反規範"},
    {"category_id": 3, "permission": "deny", "reason": "惡意刊登"}
  ]
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "批次權限設定成功",
  "data": {
    "permissions": [
      {"id": 1, "category_id": 1, "permission": "allow"},
      {"id": 2, "category_id": 2, "permission": "deny"},
      {"id": 3, "category_id": 3, "permission": "deny"}
    ]
  }
}
```

---

### 移除使用者的分類權限（恢復預設允許）

**DELETE** `/api/admin/users/{userId}/category-permissions/{categoryId}`

刪除特定的分類權限設定，該使用者恢復預設狀態（允許購買）。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `userId` | integer | 使用者 ID |
| `categoryId` | integer | 分類 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "權限已移除",
  "data": null
}
```
