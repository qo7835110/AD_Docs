# 會員認證 API

> 以下 API 需在 Header 帶入會員 JWT Token：
> `Authorization: Bearer {token}`

---

## 取得目前登入會員資料

**GET** `/api/auth/me`

### Response 200 - 成功

```json
{
  "success": true,
  "message": null,
  "data": {
    "user": {
      "id": 1,
      "name": "王小明",
      "email": "user@example.com",
      "phone": "0912345678",
      "tax_id": "12345678",
      "address": "台北市信義區信義路五段7號",
      "status": "active",
      "original": null,
      "last_login_at": "2025-03-01T10:00:00+08:00",
      "created_at": "2025-01-01T00:00:00+08:00"
    }
  }
}
```

---

## 登出

**POST** `/api/auth/logout`

使當前 Token 失效。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "登出成功",
  "data": null
}
```

---

## 刷新 Token

**POST** `/api/auth/refresh`

使用舊 Token 換取新 Token，舊 Token 立即失效。

### Response 200 - 成功

```json
{
  "success": true,
  "message": "登入成功",
  "data": {
    "user": { ... },
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "bearer",
    "expires_in": 3600
  }
}
```

---

## 修改密碼

**PUT** `/api/auth/change-password`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `old_password` | string | 是 | 目前密碼 |
| `new_password` | string | 是 | 新密碼（最少 8 碼） |
| `new_password_confirmation` | string | 是 | 確認新密碼（需與 `new_password` 相同） |

```json
{
  "old_password": "password123",
  "new_password": "newpassword456",
  "new_password_confirmation": "newpassword456"
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

---

## 更新個人資料

**PUT** `/api/auth/profile`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 否 | 姓名 |
| `phone` | string | 否 | 手機號碼 |
| `tax_id` | string | 否 | 統一編號（8 碼數字） |
| `address` | string | 否 | 地址 |

```json
{
  "name": "王大明",
  "phone": "0987654321",
  "tax_id": "87654321",
  "address": "新北市板橋區文化路一段1號"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "資料更新成功",
  "data": {
    "user": {
      "id": 1,
      "name": "王大明",
      "email": "user@example.com",
      "phone": "0987654321",
      "tax_id": "87654321",
      "address": "新北市板橋區文化路一段1號",
      "status": "active"
    }
  }
}
```
