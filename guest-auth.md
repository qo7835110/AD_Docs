# 👤 Guest Auth — 訪客認證 API

> **Prefix：** `/api/guest`
> 訪客帳號獨立於正式會員（`users` 表），使用 `guests` 表儲存，JWT Guard 為 `guest`。
> 有 🔒 標記的端點需要 `Authorization: Bearer {guest_token}`

---

## 端點索引

| 方法   | 路徑                  | 說明             | 需要驗證 |
| ------ | --------------------- | ---------------- | -------- |
| `POST` | `/api/guest/register` | 訪客註冊         | ❌       |
| `POST` | `/api/guest/login`    | 訪客登入         | ❌       |
| `GET`  | `/api/guest/me`       | 取得目前訪客資料 | ✅       |
| `POST` | `/api/guest/logout`   | 訪客登出         | ✅       |
| `POST` | `/api/guest/refresh`  | 刷新訪客 Token   | ✅       |

---

## POST `/api/guest/register` — 訪客註冊

**Request Body（JSON）**

| 欄位                    | 類型     | 必填 | 說明                     |
| ----------------------- | -------- | ---- | ------------------------ |
| `name`                  | `string` | ✅   | 姓名（max:255）          |
| `email`                 | `string` | ✅   | Email（guests 表唯一值） |
| `password`              | `string` | ✅   | 密碼（min:6）            |
| `password_confirmation` | `string` | ✅   | 確認密碼                 |
| `phone`                 | `string` | ❌   | 手機號碼（max:20）       |

**請求範例**

```json
{
    "name": "張三",
    "email": "guest@example.com",
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
        "guest": {
            "id": 1,
            "name": "張三",
            "email": "guest@example.com",
            "phone": "0912345678",
            "created_at": "2026-03-12 11:00:00"
        },
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9..."
    }
}
```

**錯誤回應**

| 狀態碼 | 說明                                   |
| ------ | -------------------------------------- |
| `422`  | 驗證失敗（email 已存在於 guests 表等） |

---

## POST `/api/guest/login` — 訪客登入

**Request Body（JSON）**

| 欄位       | 類型     | 必填 | 說明  |
| ---------- | -------- | ---- | ----- |
| `email`    | `string` | ✅   | Email |
| `password` | `string` | ✅   | 密碼  |

**請求範例**

```json
{
    "email": "guest@example.com",
    "password": "password123"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "登入成功",
    "guest": {
        "id": 1,
        "name": "張三",
        "email": "guest@example.com"
    },
    "token": "eyJ0eXAiOiJKV1Qi...",
    "token_type": "bearer",
    "expires_in": 3600
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `401`  | 帳號或密碼錯誤 |
| `422`  | 驗證失敗       |

---

## GET `/api/guest/me` — 取得訪客資料 🔒

**回應範例（200）**

```json
{
    "success": true,
    "data": {
        "guest": {
            "id": 1,
            "name": "張三",
            "email": "guest@example.com",
            "phone": "0912345678"
        }
    }
}
```

---

## POST `/api/guest/logout` — 訪客登出 🔒

**回應範例（200）**

```json
{
    "success": true,
    "message": "登出成功",
    "data": null
}
```

---

## POST `/api/guest/refresh` — 刷新 Token 🔒

**回應範例（200）**

```json
{
  "success": true,
  "message": "登入成功",
  "guest": { ... },
  "token": "eyJ0eXAiOiJKV1Qi...",
  "token_type": "bearer",
  "expires_in": 3600
}
```
