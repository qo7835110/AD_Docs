# Orders API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [GET] `/api/orders`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/orders`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `items` | `required|array|min:1` |  |
| `items.*.plan_option_id` | `required|integer|exists:plan_options,id` |  |
| `items.*.quantity` | `nullable|integer|min:1` |  |
| `discount` | `nullable|numeric|min:0` |  |
| `notes` | `nullable|string|max:1000` |  |

---

## [POST] `/api/orders/with-ads`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `items` | `required|array|min:1` |  |
| `items.*.plan_option_id` | `required|integer|exists:plan_options,id` |  |
| `items.*.quantity` | `nullable|integer|min:1` |  |
| `items.*.ads` | `nullable|array` |  |
| `items.*.ads.*.title` | `required|string|max:255` |  |
| `items.*.ads.*.description` | `nullable|string` |  |
| `items.*.ads.*.link_url` | `nullable|url|max:2048` |  |
| `discount` | `nullable|numeric|min:0` |  |
| `notes` | `nullable|string|max:1000` |  |

---

## [GET] `/api/orders/{orderNumber}`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/orders/{orderNumber}/cancel`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/orders/{orderNumber}/pay`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `payment_method` | `required|string` |  |
| `payment_data` | `nullable|array` |  |

---

## [GET] `/api/orders/{orderNumber}/payments`

**Authentication:** User (Bearer auth:api)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/orders/{orderNumber}/refund`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `amount` | `nullable|numeric|min:0` |  |
| `reason` | `nullable|string|max:500` |  |

---

