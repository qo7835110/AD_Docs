# Auth Register API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [POST] `/api/auth/register`

**Authentication:** Guest (None)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `name` | `required|string|max:255` |  |
| `email` | `required|string|email|max:255|unique:users` |  |
| `password` | `required|string|min:6|confirmed` |  |
| `phone` | `nullable|string|max:20` |  |
| `tax_id` | `nullable|string|max:20` |  |
| `address` | `nullable|string|max:255` |  |

---

