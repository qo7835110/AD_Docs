# 管理員認證 API

> 以下標記為「需認證」的 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`
>
> Token 由管理員登入 API 取得（見 `/api/admin/login`，該端點屬公開 API 詳見 [public/auth.md](../public/auth.md)）。

---

## 管理員登出

**POST** `/api/admin/logout`

需認證。使當前管理員 Token 失效。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "登出成功",
  "data": null
}
```

---

## 取得目前登入管理員資料

**GET** `/api/admin/me`

需認證。

### Response 200 - 成功

```json
{
  "success": true,
  "message": null,
  "data": {
    "admin": {
      "id": 1,
      "name": "系統管理員",
      "email": "admin@example.com",
      "status": "active",
      "last_login_at": "2026-04-08T01:00:00+08:00"
    }
  }
}
```

---

## 刷新管理員 Token

**POST** `/api/admin/refresh`

需認證。使用舊 Token 換取新 Token，舊 Token 立即失效。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "登入成功",
  "data": {
    "admin": { ... },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "bearer",
    "expires_in": 3600
  }
}
```

---

## 管理員修改密碼

**PUT** `/api/admin/change-password`

需認證。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `old_password` | string | 是 | 目前密碼 |
| `new_password` | string | 是 | 新密碼（最少 8 碼） |
| `new_password_confirmation` | string | 是 | 確認新密碼（需與 `new_password` 相同） |

```json
{
  "old_password": "current_password",
  "new_password": "new_secure_password",
  "new_password_confirmation": "new_secure_password"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "密碼修改成功",
  "data": null
}
```

### Response 400 - 舊密碼錯誤

```json
{
  "success": false,
  "message": "舊密碼錯誤",
  "data": null
}
```
