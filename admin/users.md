# 管理員使用者帳號管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`
>
> 所有端點均需對應的模組權限（以 `admin.permission` Middleware 控管）。

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 取得使用者列表 / 詳情 | `users:view` |
| 更新使用者資料 / 啟用 / 停用 | `users:update` |
| 刪除使用者 | `users:delete` |

---

## 使用者狀態說明

| 狀態值 | 說明 |
|--------|------|
| `active` | 啟用中 |
| `inactive` | 未啟用 |
| `suspended` | 停用中 |

---

## 取得使用者列表

**GET** `/api/admin/users`

可透過狀態、關鍵字與分頁參數查詢。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選狀態：`active`、`inactive`、`suspended` |
| `keyword` | string | 否 | 依姓名或 Email 模糊搜尋 |
| `per_page` | integer | 否 | 每頁筆數，預設 `15` |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "data": [
      {
        "id": 5,
        "name": "王小明",
        "email": "user@example.com",
        "phone": "0912345678",
        "tax_id": "12345678",
        "address": "台北市信義區信義路五段7號",
        "status": "active",
        "last_login_at": "2026-04-30 09:30:00",
        "created_at": "2026-03-13 10:00:00",
        "updated_at": "2026-04-30 09:30:00"
      }
    ],
    "links": {
      "first": "http://localhost/api/admin/users?page=1",
      "last": "http://localhost/api/admin/users?page=4",
      "prev": null,
      "next": "http://localhost/api/admin/users?page=2"
    },
    "meta": {
      "current_page": 1,
      "from": 1,
      "last_page": 4,
      "path": "http://localhost/api/admin/users",
      "per_page": 15,
      "to": 15,
      "total": 60
    }
  }
}
```

---

## 取得指定使用者

**GET** `/api/admin/users/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 使用者 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "user": {
      "id": 5,
      "name": "王小明",
      "email": "user@example.com",
      "phone": "0912345678",
      "tax_id": "12345678",
      "address": "台北市信義區信義路五段7號",
      "status": "active",
      "last_login_at": "2026-04-30 09:30:00",
      "created_at": "2026-03-13 10:00:00",
      "updated_at": "2026-04-30 09:30:00"
    }
  }
}
```

### Response 404 - 使用者不存在

```json
{
  "success": false,
  "message": "使用者不存在"
}
```

---

## 更新使用者資料

**PUT** `/api/admin/users/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 使用者 ID |

### Request Body

以下欄位皆為選填：

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 否 | 姓名，最長 255 字元 |
| `email` | string | 否 | Email，最長 255 字元 |
| `password` | string | 否 | 新密碼，至少 8 碼 |
| `password_confirmation` | string | 否 | 密碼確認，需與 `password` 一致 |
| `phone` | string/null | 否 | 電話，最長 20 字元；傳 `null` 可清空 |
| `tax_id` | string/null | 否 | 統一編號，最長 20 字元；傳 `null` 可清空 |
| `address` | string/null | 否 | 地址，最長 255 字元；傳 `null` 可清空 |
| `status` | string | 否 | 狀態：`active`、`inactive`、`suspended` |

```json
{
  "name": "王小明(更新)",
  "phone": null,
  "status": "inactive"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "使用者資料更新成功",
  "data": {
    "user": {
      "id": 5,
      "name": "王小明(更新)",
      "email": "user@example.com",
      "phone": null,
      "tax_id": "12345678",
      "address": "台北市信義區信義路五段7號",
      "status": "inactive",
      "last_login_at": "2026-04-30 09:30:00",
      "created_at": "2026-03-13 10:00:00",
      "updated_at": "2026-05-01 10:00:00"
    }
  }
}
```

### Response 400 - Email 已被使用

```json
{
  "success": false,
  "message": "該 Email 已被使用"
}
```

### Response 404 - 使用者不存在

```json
{
  "success": false,
  "message": "使用者不存在"
}
```

### Response 422 - 驗證失敗

```json
{
  "success": false,
  "message": "驗證失敗",
  "errors": {
    "status": ["狀態必須為 active、inactive 或 suspended"]
  }
}
```

---

## 停用使用者帳號

**POST** `/api/admin/users/{id}/suspend`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 使用者 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "使用者帳號已停用",
  "data": {
    "user": {
      "id": 5,
      "status": "suspended"
    }
  }
}
```

### Response 400 - 已處於停用狀態

```json
{
  "success": false,
  "message": "該使用者已處於停用狀態"
}
```

### Response 404 - 使用者不存在

```json
{
  "success": false,
  "message": "使用者不存在"
}
```

---

## 啟用使用者帳號

**POST** `/api/admin/users/{id}/activate`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 使用者 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "使用者帳號已啟用",
  "data": {
    "user": {
      "id": 5,
      "status": "active"
    }
  }
}
```

### Response 400 - 已處於啟用狀態

```json
{
  "success": false,
  "message": "該使用者已處於啟用狀態"
}
```

### Response 404 - 使用者不存在

```json
{
  "success": false,
  "message": "使用者不存在"
}
```

---

## 刪除使用者帳號

**DELETE** `/api/admin/users/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 使用者 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "使用者帳號已刪除"
}
```

### Response 404 - 使用者不存在

```json
{
  "success": false,
  "message": "使用者不存在"
}
```