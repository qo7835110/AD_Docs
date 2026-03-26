# Auth Login API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [POST] `/api/auth/login`

**Authentication:** Guest (None)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `email` | `required|email` |  |
| `password` | `required|string` |  |
| `auth_type` | `nullable|string|in:JOB` |  |

---

