# 管理員帳號管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`
>
> 所有端點均需對應的模組權限（以 `admin.permission` Middleware 控管）。

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 取得管理員列表 / 詳情 | `admins:view` |
| 建立管理員 | `admins:create` |
| 更新管理員 | `admins:update` |
| 刪除管理員 | `admins:delete` |

---

## 管理員狀態說明

| 狀態值 | 說明 |
|--------|------|
| `active` | 啟用中 |
| `inactive` | 已停用 |

> 目前此組 API 的請求驗證僅接受 `active` 與 `inactive`。

---

## 取得管理員列表

**GET** `/api/admin/admins`

可透過狀態或是否超級管理員進行篩選。

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `status` | string | 否 | 篩選狀態：`active`、`inactive` |
| `is_super` | boolean | 否 | 篩選是否為超級管理員 |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "admins": [
      {
        "id": 1,
        "name": "系統管理員",
        "email": "admin@example.com",
        "phone": "0912345678",
        "status": "active",
        "is_super": true,
        "last_login_at": "2026-04-29 09:30:00",
        "created_at": "2026-03-13 10:00:00",
        "updated_at": "2026-04-29 09:30:00"
      }
    ]
  }
}
```

---

## 取得指定管理員

**GET** `/api/admin/admins/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 管理員 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "admin": {
      "id": 2,
      "name": "審核管理員",
      "email": "reviewer@example.com",
      "phone": null,
      "status": "active",
      "is_super": false,
      "last_login_at": null,
      "created_at": "2026-04-01 10:00:00",
      "updated_at": "2026-04-01 10:00:00"
    }
  }
}
```

### Response 404 - 管理員不存在

```json
{
  "success": false,
  "message": "管理員不存在"
}
```

---

## 建立管理員帳號

**POST** `/api/admin/admins`

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 是 | 姓名 |
| `email` | string | 是 | Email |
| `password` | string | 是 | 密碼，至少 8 碼 |
| `password_confirmation` | string | 是 | 密碼確認，需與 `password` 一致 |
| `phone` | string | 否 | 電話，最長 20 字元 |
| `status` | string | 否 | 狀態：`active`、`inactive`，預設 `active` |
| `is_super` | boolean | 否 | 是否為超級管理員，預設 `false` |

```json
{
  "name": "新管理員",
  "email": "new-admin@example.com",
  "password": "password123",
  "password_confirmation": "password123",
  "phone": "0988000111",
  "status": "active",
  "is_super": false
}
```

### Response 201 - 成功

```json
{
  "success": true,
  "message": "管理員帳號建立成功",
  "data": {
    "admin": {
      "id": 10,
      "name": "新管理員",
      "email": "new-admin@example.com",
      "phone": "0988000111",
      "status": "active",
      "is_super": false,
      "last_login_at": null,
      "created_at": "2026-04-29 10:30:00",
      "updated_at": "2026-04-29 10:30:00"
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

### Response 422 - 驗證失敗

```json
{
  "success": false,
  "message": "驗證失敗",
  "errors": {
    "password": ["密碼至少 8 個字元"]
  }
}
```

---

## 更新管理員資料

**PUT** `/api/admin/admins/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 管理員 ID |

### Request Body

以下欄位皆為選填：

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 否 | 姓名 |
| `email` | string | 否 | Email |
| `password` | string | 否 | 新密碼，至少 8 碼 |
| `password_confirmation` | string | 否 | 密碼確認 |
| `phone` | string/null | 否 | 電話；傳 `null` 可清空 |
| `status` | string | 否 | 狀態：`active`、`inactive` |
| `is_super` | boolean | 否 | 是否為超級管理員 |

```json
{
  "name": "審核主管",
  "phone": null,
  "status": "inactive"
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "管理員資料更新成功",
  "data": {
    "admin": {
      "id": 2,
      "name": "審核主管",
      "email": "reviewer@example.com",
      "phone": null,
      "status": "inactive",
      "is_super": false,
      "last_login_at": null,
      "created_at": "2026-04-01 10:00:00",
      "updated_at": "2026-04-29 11:00:00"
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

### Response 404 - 管理員不存在

```json
{
  "success": false,
  "message": "管理員不存在"
}
```

---

## 刪除管理員帳號

**DELETE** `/api/admin/admins/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 管理員 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "管理員帳號已刪除"
}
```

### Response 400 - 業務規則限制

```json
{
  "success": false,
  "message": "無法刪除自己的帳號"
}
```

```json
{
  "success": false,
  "message": "無法刪除超級管理員帳號"
}
```

### Response 404 - 管理員不存在

```json
{
  "success": false,
  "message": "管理員不存在"
}
```

### Response 500 - 刪除失敗

```json
{
  "success": false,
  "message": "刪除管理員失敗"
}
```
