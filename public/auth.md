# Public API - 會員與管理員認證

> 本文件包含不需任何 Bearer Token 驗證即可呼叫的公開認證介面。

## [POST] `/api/auth/register`
註冊新會員帳號。
- **Payload Validation:**
  - `name` (string, required, max:255)
  - `email` (string, required, email, unique:users)
  - `password` (string, required, min:8, confirmed)
- **Response:**
  - 客戶端將收到 `user` 實體與有效的 `access_token`。

## [POST] `/api/auth/login`
會員登入並取得 JWT Token。
- **Payload Validation:**
  - `email` (string, required, email)
  - `password` (string, required)
- **Response:**
  - 回傳 `access_token` 與 `token_type` (bearer)。

## [POST] `/api/admin/login`
管理員登入並取得專屬 JWT Token。
- **Payload Validation:**
  - `email` (string, required, email)
  - `password` (string, required)
- **Response:**
  - 成功將回傳 Guard `admin` 專屬的 `access_token`。
