# Admin Admins API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [GET] `/api/admin/admins/{adminId}/permissions`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/admins/{adminId}/permissions`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `module` | `required|string|in:ads,categories,ad_plans,plan_options,category_permissions,admin_permissions` |  |
| `action` | `required|string|in:view,create,update,delete` |  |
| `reason` | `nullable|string|max:500` |  |

---

## [DELETE] `/api/admin/admins/{adminId}/permissions`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `module` | `required|string|in:ads,categories,ad_plans,plan_options,category_permissions,admin_permissions` |  |
| `action` | `required|string|in:view,create,update,delete` |  |
| `reason` | `nullable|string|max:500` |  |

---

## [PUT] `/api/admin/admins/{adminId}/permissions/batch`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `permissions` | `required|array|min:1` |  |
| `permissions.*.module` | `required|string|in:ads,categories,ad_plans,plan_options,category_permissions,admin_permissions` |  |
| `permissions.*.action` | `required|string|in:view,create,update,delete` |  |
| `reason` | `nullable|string|max:500` |  |

---

## [GET] `/api/admin/admins/{adminId}/permissions/logs`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

