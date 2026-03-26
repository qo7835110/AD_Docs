# Admin Ad Plans API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [POST] `/api/admin/ad-plans`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `category_id` | `required|integer|exists:categories,id` |  |
| `name` | `required|string|max:255` |  |
| `description` | `nullable|string` |  |
| `ad_limit` | `nullable|integer|min:1` |  |
| `status` | `nullable|in:active,inactive` |  |
| `features` | `nullable|array` |  |
| `image` | `nullable|Closure` |  |
| `options` | `nullable|array` |  |
| `options.*.name` | `required|string|max:255` |  |
| `options.*.description` | `nullable|string` |  |
| `options.*.duration_days` | `required|integer|min:1` |  |
| `options.*.price` | `required|integer|min:0` |  |
| `options.*.valid_start_date` | `nullable|date` |  |
| `options.*.valid_end_date` | `nullable|date|after_or_equal:options.*.valid_start_date` |  |
| `options.*.sort_order` | `nullable|integer` |  |

---

## [PUT] `/api/admin/ad-plans/options/{id}`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `name` | `sometimes|required|string|max:255` |  |
| `description` | `nullable|string` |  |
| `duration_days` | `sometimes|required|integer|min:1` |  |
| `price` | `sometimes|required|integer|min:0` |  |
| `valid_start_date` | `nullable|date` |  |
| `valid_end_date` | `nullable|date|after_or_equal:valid_start_date` |  |
| `sort_order` | `nullable|integer` |  |

---

## [DELETE] `/api/admin/ad-plans/options/{id}`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [PUT] `/api/admin/ad-plans/{id}`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `category_id` | `sometimes|required|integer|exists:categories,id` |  |
| `name` | `sometimes|required|string|max:255` |  |
| `description` | `nullable|string` |  |
| `ad_limit` | `nullable|integer|min:1` |  |
| `status` | `nullable|in:active,inactive` |  |
| `features` | `nullable|array` |  |
| `image` | `nullable|Closure` |  |
| `options` | `nullable|array` |  |
| `options.*.id` | `nullable|integer` |  |
| `options.*.name` | `required|string|max:255` |  |
| `options.*.description` | `nullable|string` |  |
| `options.*.duration_days` | `required|integer|min:1` |  |
| `options.*.price` | `required|integer|min:0` |  |
| `options.*.valid_start_date` | `nullable|date` |  |
| `options.*.valid_end_date` | `nullable|date|after_or_equal:options.*.valid_start_date` |  |
| `options.*.sort_order` | `nullable|integer` |  |

---

## [DELETE] `/api/admin/ad-plans/{id}`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/ad-plans/{planId}/options`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `name` | `required|string|max:255` |  |
| `description` | `nullable|string` |  |
| `duration_days` | `required|integer|min:1` |  |
| `price` | `required|integer|min:0` |  |
| `valid_start_date` | `nullable|date` |  |
| `valid_end_date` | `nullable|date|after_or_equal:valid_start_date` |  |
| `sort_order` | `nullable|integer` |  |

---

