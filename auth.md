# 🔐 Auth — 會員認證 API

> **Prefix：** `/api/auth`
> **認證方式：** 有 🔒 標記的端點需要 `Authorization: Bearer {token}` Header

---

## 端點索引

| 方法   | 路徑                        | 說明                        | 需要驗證 |
| ------ | --------------------------- | --------------------------- | -------- |
| `POST` | `/api/auth/register`        | 會員註冊                    | ❌       |
| `POST` | `/api/auth/login`           | 會員登入（支援 JOB 第三方） | ❌       |
| `GET`  | `/api/auth/me`              | 取得目前登入的會員資料      | ✅       |
| `POST` | `/api/auth/logout`          | 登出                        | ✅       |
| `POST` | `/api/auth/refresh`         | 刷新 JWT Token              | ✅       |
| `PUT`  | `/api/auth/profile`         | 更新個人資料                | ✅       |
| `PUT`  | `/api/auth/change-password` | 修改密碼                    | ✅       |

---

## POST `/api/auth/register` — 會員註冊

**Request Body（JSON）**

| 欄位                    | 類型     | 必填 | 說明               |
| ----------------------- | -------- | ---- | ------------------ |
| `name`                  | `string` | ✅   | 姓名（max:255）    |
| `email`                 | `string` | ✅   | Email（唯一值）    |
| `password`              | `string` | ✅   | 密碼（min:6）      |
| `password_confirmation` | `string` | ✅   | 確認密碼           |
| `phone`                 | `string` | ❌   | 手機號碼（max:20） |

**請求範例**

```json
{
    "name": "王小明",
    "email": "user@example.com",
    "password": "password123",
    "password_confirmation": "password123",
    "phone": "0912345678"
}
```

**回應範例（201）**

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
            "status": "active",
            "created_at": "2026-03-12 11:00:00"
        },
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                                   |
| ------ | -------------------------------------- |
| `422`  | 驗證失敗（email 已存在、密碼不一致等） |
| `500`  | 系統錯誤                               |

---

## POST `/api/auth/login` — 會員登入

支援一般登入及 JOB 第三方登入。

**Request Body（JSON）**

| 欄位        | 類型     | 必填 | 說明                               |
| ----------- | -------- | ---- | ---------------------------------- |
| `email`     | `string` | ✅   | Email                              |
| `password`  | `string` | ✅   | 密碼                               |
| `auth_type` | `string` | ❌   | 傳入 `JOB` 使用第三方 JOB API 驗證 |

**請求範例（一般登入）**

```json
{
    "email": "user@example.com",
    "password": "password123"
}
```

**請求範例（JOB 第三方登入）**

```json
{
    "email": "user@example.com",
    "password": "password123",
    "auth_type": "JOB"
}
```

> **JOB 登入說明：** 系統會向 `https://findjob.lemaxim.tw/v1/user/login` 驗證，若驗證成功且使用者不存在本地資料庫，將自動建立帳號。

**回應範例（200）**

```json
{
    "success": true,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "bearer",
    "expires_in": 3600,
    "user": {
        "id": 1,
        "name": "王小明",
        "email": "user@example.com"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                            |
| ------ | ------------------------------- |
| `401`  | 帳號或密碼錯誤 / 第三方驗證失敗 |
| `422`  | 驗證失敗（欄位格式錯誤）        |
| `500`  | 第三方驗證服務異常              |

---

## GET `/api/auth/me` — 取得目前會員資料 🔒

**回應範例（200）**

```json
{
    "success": true,
    "data": {
        "user": {
            "id": 1,
            "name": "王小明",
            "email": "user@example.com",
            "phone": "0912345678",
            "status": "active",
            "last_login_at": "2026-03-12 10:30:00"
        }
    }
}
```

---

## POST `/api/auth/logout` — 登出 🔒

**回應範例（200）**

```json
{
    "success": true,
    "message": "登出成功",
    "data": null
}
```

---

## POST `/api/auth/refresh` — 刷新 Token 🔒

刷新 JWT Token，舊 Token 將失效。

**回應範例（200）**

```json
{
  "success": true,
  "access_token": "eyJ0eXAiOiJKV1Qi...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": { ... }
}
```

---

## PUT `/api/auth/profile` — 更新個人資料 🔒

**Request Body（JSON）**

| 欄位    | 類型     | 必填 | 說明               |
| ------- | -------- | ---- | ------------------ |
| `name`  | `string` | ❌   | 姓名（max:255）    |
| `phone` | `string` | ❌   | 手機號碼（max:20） |

**請求範例**

```json
{
    "name": "王大明",
    "phone": "0987654321"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "資料更新成功",
    "data": {
        "user": {
            "id": 1,
            "name": "王大明",
            "phone": "0987654321"
        }
    }
}
```

---

## PUT `/api/auth/change-password` — 修改密碼 🔒

**Request Body（JSON）**

| 欄位                        | 類型     | 必填 | 說明            |
| --------------------------- | -------- | ---- | --------------- |
| `old_password`              | `string` | ✅   | 目前密碼        |
| `new_password`              | `string` | ✅   | 新密碼（min:6） |
| `new_password_confirmation` | `string` | ✅   | 確認新密碼      |

**請求範例**

```json
{
    "old_password": "password123",
    "new_password": "newpassword456",
    "new_password_confirmation": "newpassword456"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "密碼修改成功",
    "data": null
}
```

**錯誤回應**

| 狀態碼 | 說明       |
| ------ | ---------- |
| `400`  | 舊密碼錯誤 |
| `422`  | 驗證失敗   |
