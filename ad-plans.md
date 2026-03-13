# 📋 Ad Plans — 廣告方案管理 API

> **讀取端點為公開 API**，不需要驗證。
> **建立、更新、刪除為管理員專屬**，需要 `auth:admin` Guard 🔒。

---

## 端點索引

### 公開端點（無需驗證）

| 方法  | 路徑                 | 說明         |
| ----- | -------------------- | ------------ |
| `GET` | `/api/ad-plans`      | 廣告方案列表 |
| `GET` | `/api/ad-plans/{id}` | 廣告方案詳情 |

### 管理員端點（auth:admin 🔒）

| 方法     | 路徑                      | 說明         |
| -------- | ------------------------- | ------------ |
| `POST`   | `/api/admin/ad-plans`     | 建立廣告方案 |
| `PUT`    | `/api/admin/ad-plans/{id}`| 更新廣告方案 |
| `DELETE` | `/api/admin/ad-plans/{id}`| 刪除廣告方案 |

---

## GET `/api/ad-plans` — 廣告方案列表

**Query Parameters**

| 參數       | 類型     | 說明                                                             |
| ---------- | -------- | ---------------------------------------------------------------- |
| `status`   | `string` | 篩選狀態：`active` / `inactive`                                  |
| `category` | `string` | 篩選分類（支援分類 ID 或 slug，如：`1` 或 `meeting-management`） |

**請求範例**

```
GET /api/ad-plans?status=active&category=meeting-management
```

**回應範例（200）**

```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "category": "會務管理",
            "name": "基礎方案",
            "description": "適合小型企業",
            "price": 999.0,
            "duration_days": 30,
            "ad_limit": 10,
            "status": "active",
            "features": ["功能A", "功能B"],
            "created_at": "2026-01-01 00:00:00"
        }
    ]
}
```

---

## GET `/api/ad-plans/{id}` — 廣告方案詳情

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 方案 ID |

**回應範例（200）**

```json
{
    "success": true,
    "data": {
        "id": 1,
        "category_id": 1,
        "name": "基礎方案",
        "description": "適合小型企業",
        "price": "999.00",
        "duration_days": 30,
        "ad_limit": 10,
        "status": "active",
        "features": ["功能A", "功能B"],
        "options": [
            {
                "id": 1,
                "name": "3個月方案",
                "price": 2699,
                "duration_days": 90
            }
        ]
    }
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `404`  | 廣告方案不存在 |

---

## POST `/api/admin/ad-plans` — 建立廣告方案 🔒 Admin

**Content-Type：** `multipart/form-data` 或 `application/json`

**Request Body**

| 欄位            | 類型          | 必填 | 說明                                                                    |
| --------------- | ------------- | ---- | ----------------------------------------------------------------------- |
| `category`      | `string`      | ✅   | 分類 slug 或 ID（如 `meeting-management` 或 `1`）                       |
| `name`          | `string`      | ✅   | 方案名稱（max:255）                                                     |
| `description`   | `string`      | ❌   | 方案說明                                                                |
| `price`         | `number`      | ✅   | 價格（min:0）                                                           |
| `duration_days` | `integer`     | ✅   | 有效天數（min:1）                                                       |
| `ad_limit`      | `integer`     | ❌   | 廣告數量上限（min:1）                                                   |
| `status`        | `string`      | ❌   | `active`（預設）/ `inactive`                                            |
| `features`      | `array`       | ❌   | 方案特色列表（JSON 陣列）                                               |
| `image`         | `file/string` | ❌   | 方案圖片（支援上傳檔案或 base64，格式：jpeg/jpg/png/gif/webp，max:2MB） |
| `options`       | `array`       | ❌   | 方案選項列表（見下方 options 欄位說明）                                 |

**`options` 子欄位**

| 欄位               | 類型      | 必填 | 說明                                            |
| ------------------ | --------- | ---- | ----------------------------------------------- |
| `name`             | `string`  | ✅   | 選項名稱（max:255）                             |
| `description`      | `string`  | ❌   | 選項說明                                        |
| `duration_days`    | `integer` | ✅   | 有效天數（min:1）                               |
| `price`            | `integer` | ✅   | 價格（min:0）                                   |
| `valid_start_date` | `date`    | ❌   | 有效起始日（`Y-m-d`）                           |
| `valid_end_date`   | `date`    | ❌   | 有效結束日（`Y-m-d`，需 >= `valid_start_date`） |
| `sort_order`       | `integer` | ❌   | 排序                                            |

**請求範例（JSON）**

```json
{
    "category": "meeting-management",
    "name": "企業專業方案",
    "description": "適合中型企業使用",
    "price": 2999.0,
    "duration_days": 90,
    "ad_limit": 50,
    "status": "active",
    "features": ["無限下架", "優先顯示", "數據分析"],
    "options": [
        {
            "name": "3個月方案",
            "price": 2699,
            "duration_days": 90,
            "valid_start_date": "2026-01-01",
            "valid_end_date": "2026-12-31"
        }
    ]
}
```

**回應範例（201）**

```json
{
    "success": true,
    "message": "廣告方案建立成功",
    "data": {
        "id": 5,
        "name": "企業專業方案",
        "price": "2999.00",
        "options": [ ... ],
        "created_at": "2026-03-13 11:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明     |
| ------ | -------- |
| `401`  | 未授權   |
| `422`  | 驗證失敗 |

---

## PUT `/api/admin/ad-plans/{id}` — 更新廣告方案 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 方案 ID |

**Request Body** 同 `POST` 建立，所有欄位改為**選填**（至少提供一個）。

**`options` 子欄位（更新時額外支援）**

| 欄位 | 類型      | 說明                                      |
| ---- | --------- | ----------------------------------------- |
| `id` | `integer` | 已存在的選項 ID（傳入則更新，不傳則新增） |

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告方案更新成功",
    "data": {
        "id": 5,
        "name": "企業專業方案 Pro",
        "updated_at": "2026-03-13 12:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `401`  | 未授權         |
| `404`  | 廣告方案不存在 |
| `422`  | 驗證失敗       |

---

## DELETE `/api/admin/ad-plans/{id}` — 刪除廣告方案 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 方案 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "廣告方案刪除成功"
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `401`  | 未授權         |
| `404`  | 廣告方案不存在 |
