# Admin Users API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [GET] `/api/admin/users/{userId}/category-permissions`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [PUT] `/api/admin/users/{userId}/category-permissions`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `category_id` | `required|integer|exists:categories,id` |  |
| `permission` | `required|string|in:allow,deny` |  |
| `reason` | `nullable|string|max:500` |  |

---

## [POST] `/api/admin/users/{userId}/category-permissions/batch`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `permissions` | `required|array|min:1` |  |
| `permissions.*.category_id` | `required|integer|exists:categories,id` |  |
| `permissions.*.permission` | `required|string|in:allow,deny` |  |
| `permissions.*.reason` | `nullable|string|max:500` |  |

---

## [DELETE] `/api/admin/users/{userId}/category-permissions/{categoryId}`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

