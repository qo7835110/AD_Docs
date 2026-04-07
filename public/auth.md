# 公開認證 API

> 以下 API 無需任何認證 Token 即可存取。

---

## 會員註冊

**POST** `/api/auth/register`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 是 | 姓名 |
| `email` | string | 是 | 電子信箱，需唯一 |
| `password` | string | 是 | 密碼（最少 8 碼） |
| `password_confirmation` | string | 是 | 確認密碼（需與 `password` 相同） |
| `phone` | string | 否 | 手機號碼 |
| `tax_id` | string | 否 | 統一編號（8 碼數字） |
| `address` | string | 否 | 地址 |

```json
{
  "name": "王小明",
  "email": "user@example.com",
  "password": "password123",
  "password_confirmation": "password123",
  "phone": "0912345678",
  "tax_id": "12345678",
  "address": "台北市信義區信義路五段7號"
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "註冊成功",
  "data": {
    "user": {
      "id": 1,
      "name": "王小明",
      "email": "user@example.com",
      "phone": "0912345678",
      "status": "active"
    },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
  }
}
```

### Response 422 - 驗證失敗

```json
{
  "success": false,
  "message": "驗證失敗",
  "data": {
    "email": ["The email has already been taken."]
  }
}
```

---

## 會員登入

**POST** `/api/auth/login`

支援兩種登入模式：平台帳號登入與第三方 JOB 帳號登入。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `email` | string | 是 | 電子信箱 |
| `password` | string | 是 | 密碼 |
| `auth_type` | string | 否 | 認證類型，填入 `"JOB"` 表示使用第三方 JOB 平台驗證 |

```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

JOB 第三方登入範例：

```json
{
  "email": "user@example.com",
  "password": "password123",
  "auth_type": "JOB"
}
```

> **JOB 登入行為說明**：系統會使用提供的帳密向 JOB 第三方 API 驗證。若驗證成功，系統會自動以該 email 建立或找到對應的會員帳號，並同步姓名與手機號碼。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "登入成功",
  "data": {
    "user": {
      "id": 1,
      "name": "王小明",
      "email": "user@example.com",
      "status": "active"
    },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "bearer",
    "expires_in": 3600
  }
}
```

### Response 401 - 帳號或密碼錯誤

```json
{
  "success": false,
  "message": "帳號或密碼錯誤",
  "data": null
}
```

### Response 403 - 帳號被停用

```json
{
  "success": false,
  "message": "帳號已被停用",
  "data": null
}
```

---

## 管理員登入

**POST** `/api/admin/login`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `email` | string | 是 | 管理員帳號 |
| `password` | string | 是 | 密碼 |

```json
{
  "email": "admin@example.com",
  "password": "admin_password"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "登入成功",
  "data": {
    "admin": {
      "id": 1,
      "name": "系統管理員",
      "email": "admin@example.com",
      "status": "active"
    },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "bearer",
    "expires_in": 3600
  }
}
```

---

## Token 使用方式

登入成功後，後續所有需要認證的請求，請在 HTTP Header 加入：

```
Authorization: Bearer {token}
```

Token 到期後，請透過各自的 `refresh` API 換取新 Token，無需重新登入。
