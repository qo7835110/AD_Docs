# Public API - 會員與管理員認證

> 本文件包含不需任何 Bearer Token 驗證即可呼叫的公開認證介面。

## [POST] `/api/auth/register`
註冊新會員帳號。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `name` | string | max:255 | 是 | 會員姓名或暱稱 |
| `email` | string | email, unique:users | 是 | 登入用信箱 |
| `password` | string | min:8, confirmed | 是 | 密碼（長度需至少 8 碼） |
| `password_confirmation` | string | | 是 | 密碼確認用欄位 |

### Payload 範例 (JSON)
```json
{
  "name": "王大明",
  "email": "daming@example.com",
  "password": "Password123!",
  "password_confirmation": "Password123!"
}
```

- **Response:**
  - 客戶端將收到 HTTP 201 與 `user` 實體以及有效的 `access_token`。

---

## [POST] `/api/auth/login`
會員登入並取得 JWT Token。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `email` | string | email | 是 | 註冊信箱 |
| `password` | string | | 是 | 密碼 |

### Payload 範例 (JSON)
```json
{
  "email": "daming@example.com",
  "password": "Password123!"
}
```

- **Response:**
  - 回傳 `access_token` 與 `token_type` (bearer)。

---

## [POST] `/api/admin/login`
管理員登入並取得專屬 JWT Token。

### Payload 說明
與會員登入欄位相同，但後端系統將會獨立驗證 `admins` 表格與配發專屬 Auth Guard 取代 User Session。

### Payload 範例 (JSON)
```json
{
  "email": "admin@example.com",
  "password": "AdminPassword!"
}
```
