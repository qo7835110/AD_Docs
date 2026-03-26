# Admin API - 權限治理核心 (Permissions)
**身份驗證:** `auth:admin`   
**所需模組權限:** 極機密 `category_permissions` 或 `admin_permissions`

> 系統內最高層級的安全管理。分為「使用者私有白名單設定」與「後台子帳號的模組分發指派」。

---

## ── 會員端的私有分類權限 (Category Permissions) ──

**權限要求:** 操作者必須具有 `admin.permission:category_permissions` 系列權限。

## [GET] `/api/admin/users/{userId}/category-permissions`
盤點該會員特例能購買哪些受到管制、非公開的廣告分類。

## [PUT] `/api/admin/users/{userId}/category-permissions`
單筆加入一筆分類的開通權或封鎖。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `category_id` | integer | exists:categories,id | 是 | 欲賦予白名單的分類 ID |
| `status` | string | in:active,inactive | 否 | 特權覆蓋狀態（預設 active） |

### Payload 範例 (JSON)
```json
{
  "category_id": 8,
  "status": "active"
}
```

## [POST] `/api/admin/users/{userId}/category-permissions/batch`
批次一次性開啟極多個私有分類。特別適用於大型合約客戶。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `category_ids` | array | array | 是 | 大量綁定的分類 IDs |
| `category_ids.*` | integer | exists:categories,id | 是 | 單一分類的有效性檢查 |
| `status` | string | in:active,inactive | 否 | 特權覆蓋狀態 |

### Payload 範例 (JSON)
```json
{
  "category_ids": [2, 5, 8],
  "status": "active"
}
```

## [DELETE] `/api/admin/users/{userId}/category-permissions/{categoryId}`
強硬拔除該使用者的白名單資格。

---

## ── 後台管理員子權限指派 (Admin Permissions) ──

**權限要求:** 操作者通常為 SuperAdmin，需具備 `admin.permission:admin_permissions`。

## [GET] `/api/admin/permissions/modules`
列出系統所有設計為可權限化的架構元件表單（如 `ads`, `categories`）及可選 Action（`view`, `create`, `update`, `delete`）。

## [POST] `/api/admin/admins/{adminId}/permissions`
新增一項細部權限給特定後台員工。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `module` | string | required | 是 | 權限模組代稱 (`ads`, `ad_plans`...) |
| `action` | string | required | 是 | 開放之行為 (`view`, `create`...) |

### Payload 範例 (JSON)
```json
{
  "module": "ads",
  "action": "update"
}
```

## [DELETE] `/api/admin/admins/{adminId}/permissions`
拔除該員工的一項功能入口與後端攔截。

## [PUT] `/api/admin/admins/{adminId}/permissions/batch`
用前端送入的新 Array，覆蓋清理舊有的權限設定（最常見的全重設做法）。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `permissions` | array | required | 是 | 設定清單集合 |
| `permissions.*.module` | string | required | 是 | |
| `permissions.*.action` | string | required | 是 | |

### Payload 範例 (JSON)
```json
{
  "permissions": [
    {
      "module": "ads",
      "action": "view"
    },
    {
      "module": "ads",
      "action": "update"
    },
    {
      "module": "categories",
      "action": "view"
    }
  ]
}
```
