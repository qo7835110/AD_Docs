# Auth Profile API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [PUT] `/api/auth/profile`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `name` | `sometimes|required|string|max:255` |  |
| `phone` | `nullable|string|max:20` |  |
| `tax_id` | `nullable|string|max:20` |  |
| `address` | `nullable|string|max:255` |  |

---

