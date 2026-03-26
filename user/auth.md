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

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `current_password` | string | required | 是 | 目前正在使用的密碼 |
| `new_password` | string | min:8, confirmed | 是 | 欲變更的全新密碼 |
| `new_password_confirmation` | string | | 是 | 新密碼確認欄位 |

### Payload 範例 (JSON)
```json
{
  "current_password": "OldPassword123!",
  "new_password": "NewStrongPassword456!",
  "new_password_confirmation": "NewStrongPassword456!"
}
```

## [PUT] `/api/auth/profile`
更新會員可變更之個人資料。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `name` | string | sometimes, max:255 | 否 | 使用者名稱/暱稱 |
| `phone` | string | nullable, max:20 | 否 | 聯絡電話 |

### Payload 範例 (JSON)
```json
{
  "name": "王小明 (更新後)",
  "phone": "0912345678"
}
```
