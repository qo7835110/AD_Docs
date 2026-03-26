# Ads API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [GET] `/api/ads`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/ads`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `order_item_id` | `required|integer|exists:order_items,id` |  |
| `title` | `required|string|max:255` |  |
| `description` | `nullable|string` |  |
| `link_url` | `nullable|url|max:2048` |  |

---

## [GET] `/api/ads/{id}`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [PUT] `/api/ads/{id}`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `title` | `sometimes|required|string|max:255` |  |
| `description` | `nullable|string` |  |
| `link_url` | `nullable|url|max:2048` |  |

---

## [DELETE] `/api/ads/{id}`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/ads/{id}/image`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `image` | `required|image|mimes:jpeg,jpg,png,gif,webp|max:2048` |  |

---

## [POST] `/api/ads/{id}/submit`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

