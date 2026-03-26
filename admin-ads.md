# Admin Ads API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [GET] `/api/admin/ads`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [GET] `/api/admin/ads/{id}`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/ads/{id}/activate`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/ads/{id}/approve`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/ads/{id}/deactivate`

**Authentication:** Admin (Bearer auth:admin)  

*No specific FormRequest validation rules required.*

---

## [POST] `/api/admin/ads/{id}/reject`

**Authentication:** Admin (Bearer auth:admin)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `reason` | `required|string|max:1000` |  |

---

