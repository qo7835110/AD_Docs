# 🛡️ Admin Auth — 管理員認證 API

> **Prefix：** `/api/admin`
> **Guard：** `admin`（`admins` 表）
> **認證方式：** 有 🔒 標記的端點需要 `Authorization: Bearer {admin_token}`

---

## 端點索引

| 方法   | 路徑                         | 說明           | 需要驗證 |
| ------ | ---------------------------- | -------------- | -------- |
| `POST` | `/api/admin/login`           | 管理員登入     | ❌       |
| `POST` | `/api/admin/logout`          | 管理員登出     | ✅       |
| `GET`  | `/api/admin/me`              | 取得管理員資料 | ✅       |
| `POST` | `/api/admin/refresh`         | 刷新 Token     | ✅       |
| `PUT`  | `/api/admin/change-password` | 修改密碼       | ✅       |

---

## POST `/api/admin/login` — 管理員登入

**Request Body（JSON）**

| 欄位       | 類型     | 必填 | 說明  |
| ---------- | -------- | ---- | ----- |
| `email`    | `string` | ✅   | Email |
| `password` | `string` | ✅   | 密碼  |

**請求範例**

```json
{
    "email": "admin@gmail.com",
    "password": "123456789"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "登入成功",
    "data": {
        "admin": {
            "id": 1,
            "name": "Admin",
            "email": "admin@gmail.com",
            "status": "active",
            "last_login_at": "2026-03-13 10:30:00"
        },
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
        "token_type": "bearer",
        "expires_in": 3600
    }
}
```

> **注意：** 管理員回應中使用 `data.admin`（非 `data.user`），以區別於會員和訪客。

**錯誤回應**

| 狀態碼 | 說明         |
| ------ | ------------ |
| `401`  | 帳號或密碼錯誤 |
| `403`  | 帳號已被停用 |
| `422`  | 驗證失敗     |

---

## POST `/api/admin/logout` — 管理員登出 🔒

**回應範例（200）**

```json
{
    "success": true,
    "message": "登出成功"
}
```

---

## GET `/api/admin/me` — 取得管理員資料 🔒

**回應範例（200）**

```json
{
    "success": true,
    "data": {
        "admin": {
            "id": 1,
            "name": "Admin",
            "email": "admin@gmail.com",
            "status": "active",
            "last_login_at": "2026-03-13 10:30:00"
        }
    }
}
```

---

## POST `/api/admin/refresh` — 刷新 Token 🔒

**回應範例（200）**

```json
{
    "success": true,
    "message": "登入成功",
    "data": {
        "admin": { ... },
        "token": "eyJ0eXAiOiJKV1Qi...",
        "token_type": "bearer",
        "expires_in": 3600
    }
}
```

---

## PUT `/api/admin/change-password` — 修改密碼 🔒

**Request Body（JSON）**

| 欄位                        | 類型     | 必填 | 說明            |
| --------------------------- | -------- | ---- | --------------- |
| `old_password`              | `string` | ✅   | 目前密碼        |
| `new_password`              | `string` | ✅   | 新密碼（min:6） |
| `new_password_confirmation` | `string` | ✅   | 確認新密碼      |

**請求範例**

```json
{
    "old_password": "123456789",
    "new_password": "newpassword456",
    "new_password_confirmation": "newpassword456"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "密碼修改成功"
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `400`  | 舊密碼錯誤 |
| `422`  | 驗證失敗   |

---

## 預設管理員帳號

透過 `AdminUserSeeder` 建立：

| 欄位     | 值                |
| -------- | ----------------- |
| Email    | admin@gmail.com   |
| Password | 123456789         |
| Status   | active            |
