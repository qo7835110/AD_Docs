# Admin API - 管理員認證
**權限:** `auth:admin` (需攜帶 Bearer Token)

> 管理員身份具有獨立 Guard 與 JWT Table，切勿與一般 `auth:api` (User) 混用。

## [POST] `/api/admin/logout`
登出管理員帳號，並強制註銷伺服器上的 JWT Token。
- **Response:** HTTP 204 無內文回傳。

## [POST] `/api/admin/refresh`
換發新的效期管理員 Token。
- **Logistics:** 若正在長時間審核或作業，將過期前自動觸發 Refresh。

## [GET] `/api/admin/me`
取得當前登入管理員的詳細資料。
- **Response:**
  - `id`, `name`, `email` 等基礎資料。
  - 重要：`permissions` 陣列（揭露了該管理員能看見與操作哪些內部模組，包含 `ads`, `categories` 等操作權限）。

## [PUT] `/api/admin/change-password`
修改當前管理員本身的密碼以防安全外洩。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `current_password` | string | required | 是 | 嚴格對比目前密碼 |
| `new_password` | string | min:8, confirmed | 是 | 管理員新密碼（需具備強度） |
| `new_password_confirmation` | string | | 是 | 新密碼確認欄位 |

### Payload 範例 (JSON)
```json
{
  "current_password": "OldAdminPassword123",
  "new_password": "NewSecureAdmin456!",
  "new_password_confirmation": "NewSecureAdmin456!"
}
```
