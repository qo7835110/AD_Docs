# Admin Categories API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [GET] `/api/admin/categories`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/categories`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `name` | `required|string|max:255|unique:categories,name` |  |
| `slug` | `nullable|string|max:255|unique:categories,slug` |  |
| `description` | `nullable|string` |  |
| `sort_order` | `nullable|integer|min:0` |  |
| `status` | `nullable|in:active,inactive` |  |

---

## [GET] `/api/admin/categories/{id}`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [PUT] `/api/admin/categories/{id}`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `name` | `sometimes|required|string|max:255|unique:categories,name,` |  |
| `slug` | `sometimes|required|string|max:255|unique:categories,slug,` |  |
| `description` | `nullable|string` |  |
| `sort_order` | `nullable|integer|min:0` |  |
| `status` | `nullable|in:active,inactive` |  |

---

## [DELETE] `/api/admin/categories/{id}`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/categories/{id}/restore`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

