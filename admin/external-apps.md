# 外部應用管理 API

> 以下 API 需在 Header 帶入管理員 JWT Token：
> `Authorization: Bearer {token}`
>
> 所有端點均需對應的模組權限（以 `admin.permission` Middleware 控管）。

---

## 所需權限

| API | 所需權限 |
|-----|----------|
| 取得列表 / 詳情 | `external_apps:view` |
| 建立外部應用 | `external_apps:create` |
| 更新外部應用 | `external_apps:update` |
| 重新產生 Secret | `external_apps:update` |
| 刪除外部應用 | `external_apps:delete` |

---

## 外部應用說明

外部應用（External App）是讓第三方網站後端透過 **API Key + API Secret** 代替使用者操作廣告的機制。

- 每個外部應用綁定一個 **owner user**，操作時權限等同該 user。
- `api_secret` 僅在**建立**或**重新產生**時以明文回傳一次，之後無法查詢（bcrypt 儲存）。
- 可設定 IP 白名單、權限範圍、速率上限與到期時間。

---

## 取得外部應用列表

**GET** `/api/admin/external-apps`

### Query Parameters

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `is_active` | boolean | 否 | 篩選啟用狀態 |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "apps": [
      {
        "id": 1,
        "name": "合作網站A",
        "api_key": "ext_abcdefghij1234567890",
        "owner_user_id": 5,
        "owner_email": "partner@example.com",
        "permissions": ["ads.read", "ads.create"],
        "allowed_ips": null,
        "rate_limit_per_minute": 60,
        "is_active": true,
        "expires_at": null,
        "created_at": "2026-06-01 10:00:00",
        "updated_at": "2026-06-01 10:00:00"
      }
    ]
  }
}
```

---

## 取得指定外部應用

**GET** `/api/admin/external-apps/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 外部應用 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "取得成功",
  "data": {
    "app": {
      "id": 1,
      "name": "合作網站A",
      "api_key": "ext_abcdefghij1234567890",
      "owner_user_id": 5,
      "owner_email": "partner@example.com",
      "permissions": ["ads.read", "ads.create"],
      "allowed_ips": ["203.0.113.1"],
      "rate_limit_per_minute": 60,
      "is_active": true,
      "expires_at": "2027-01-01 00:00:00",
      "created_at": "2026-06-01 10:00:00",
      "updated_at": "2026-06-01 10:00:00"
    }
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "外部應用不存在"
}
```

---

## 建立外部應用

**POST** `/api/admin/external-apps`

`api_key` 由系統自動產生（`ext_` 開頭，44 字元），`api_secret` 以明文回傳**一次**，請立即保存。

### Request Body

| 欄位 | 型別 | 必填 | 說明 |
|------|------|------|------|
| `name` | string | 是 | 應用名稱，最長 255 字元 |
| `owner_user_id` | integer | 是 | 擁有者 User ID |
| `permissions` | array | 否 | 允許的操作範圍，見下方說明 |
| `allowed_ips` | array | 否 | IP 白名單；`null` 表示不限制 |
| `rate_limit_per_minute` | integer | 否 | 每分鐘請求上限，1–600，預設 60 |
| `is_active` | boolean | 否 | 是否啟用，預設 `true` |
| `expires_at` | datetime | 否 | 到期時間；必須在現在之後 |

#### 可用 permissions 值

| 值 | 說明 |
|----|------|
| `ads.read` | 讀取廣告 |
| `ads.create` | 建立廣告 |
| `ads.update` | 更新廣告 |
| `ads.delete` | 刪除廣告 |
| `orders.read` | 讀取訂單 / 付款記錄 / 草稿 |
| `orders.create` | 建立訂單 / 訂單草稿 |
| `orders.update` | 取消 / 支付 / 退款訂單 |

```json
{
  "name": "合作網站A",
  "owner_user_id": 5,
  "permissions": ["ads.read", "ads.create"],
  "allowed_ips": ["203.0.113.1", "203.0.113.2"],
  "rate_limit_per_minute": 120,
  "is_active": true,
  "expires_at": "2027-01-01 00:00:00"
}
```

### Response 201 - 建立成功

```json
{
  "success": true,
  "message": "外部應用建立成功",
  "data": {
    "app": {
      "id": 1,
      "name": "合作網站A",
      "api_key": "ext_abcdefghij1234567890abcdefghij1234567890ab",
      "owner_user_id": 5,
      "owner_email": "partner@example.com",
      "permissions": ["ads.read", "ads.create"],
      "allowed_ips": ["203.0.113.1", "203.0.113.2"],
      "rate_limit_per_minute": 120,
      "is_active": true,
      "expires_at": "2027-01-01 00:00:00",
      "created_at": "2026-06-01 10:00:00",
      "updated_at": "2026-06-01 10:00:00"
    },
    "api_secret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "notice": "請妥善保存 api_secret，此為唯一一次顯示機會"
  }
}
```

### Response 422 - 驗證失敗

```json
{
  "success": false,
  "message": "驗證失敗",
  "errors": {
    "owner_user_id": ["指定的使用者不存在"],
    "allowed_ips.0": ["IP 位址格式不正確"]
  }
}
```

---

## 更新外部應用

**PUT** `/api/admin/external-apps/{id}`

所有欄位皆為選填（`sometimes`）。**注意：`owner_user_id` 與 `api_key` 不可透過此端點變更。**

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 外部應用 ID |

### Request Body

| 欄位 | 型別 | 說明 |
|------|------|------|
| `name` | string | 應用名稱 |
| `permissions` | array | 覆寫整個權限陣列 |
| `allowed_ips` | array\|null | IP 白名單；傳 `null` 解除限制 |
| `rate_limit_per_minute` | integer | 速率上限 |
| `is_active` | boolean | 啟用狀態 |
| `expires_at` | datetime\|null | 到期時間；傳 `null` 永不過期 |

```json
{
  "is_active": false,
  "permissions": ["ads.read"]
}
```

### Response 200 - 成功

```json
{
  "success": true,
  "message": "更新成功",
  "data": {
    "app": { ... }
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "外部應用不存在"
}
```

---

## 重新產生 API Secret

**POST** `/api/admin/external-apps/{id}/rotate-secret`

舊的 `api_secret` 立即失效，新的明文 Secret **僅回傳一次**。

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 外部應用 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "API Secret 已重新產生",
  "data": {
    "app": { ... },
    "api_secret": "yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy",
    "notice": "請妥善保存 api_secret，此為唯一一次顯示機會"
  }
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "外部應用不存在"
}
```

---

## 刪除外部應用

**DELETE** `/api/admin/external-apps/{id}`

### Path Parameters

| 參數 | 型別 | 說明 |
|------|------|------|
| `id` | integer | 外部應用 ID |

### Response 200 - 成功

```json
{
  "success": true,
  "message": "外部應用已刪除"
}
```

### Response 404 - 不存在

```json
{
  "success": false,
  "message": "外部應用不存在"
}
```
