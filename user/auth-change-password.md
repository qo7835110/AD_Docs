# Auth Change Password API

> 自動生成之 API 文件，反映最新之後端路由與驗證規則 (Validation Rules)。

## [PUT] `/api/auth/change-password`

**Authentication:** User (Bearer auth:api)  

### Request Payload (Validation Rules):

| Field | Rules | Description |
|---|---|---|
| `old_password` | `required|string` |  |
| `new_password` | `required|string|min:6|confirmed` |  |

---

