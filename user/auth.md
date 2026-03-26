# User API - 會員帳號管理
**權限:** `auth:api` (需於 HTTP Header 攜帶 `Authorization: Bearer {token}`)

## [POST] `/api/auth/logout`
登出並使當前 JWT Token 進入黑名單 (Blacklist) 永久失效。
- **Response:** 成功無回傳 Payload (HTTP 204 or 200 message)。

## [POST] `/api/auth/refresh`
換發新的 JWT Token，延長登入效期。
- **Response:** 回傳新的 `access_token`。

## [GET] `/api/auth/me`
取得當前登入會員的詳細資料。
- **Response:**
  - 除了基本使用者資料，還會一併回傳該會員已被授權購買的「分類購買權限」(`category_permissions`)。

## [PUT] `/api/auth/change-password`
修改當前會員的密碼。
- **Payload Validation:**
  - `current_password` (string, required)
  - `new_password` (string, required, min:8, confirmed)

## [PUT] `/api/auth/profile`
更新會員可變更之個人資料。
- **Payload Validation:**
  - `name` (string, sometimes|required)
  - `phone` (string, nullable) 等視 Schema 而定的非敏感欄位。
