# 🔒 Category Permissions — 分類購買權限管理 API

> **Prefix：** `/api/admin/users/{userId}/category-permissions`
> **Guard：** `admin`（`admins` 表）
> **認證：** 所有端點均需要 🔒 `Authorization: Bearer {admin_token}`

---

## 概述
提供管理員針對單一使用者（User）設定「分類購買權限（allow / deny）」。當使用者被設定為 `deny` 的分類時，將無法購買該分類下的任何廣告方案與選項。

---

## 端點索引

| 方法     | 路徑                                   | 說明                   |
| -------- | -------------------------------------- | ---------------------- |
| `GET`    | `/`                                    | 取得使用者的分類權限列表 |
| `PUT`    | `/`                                    | 設定單一分類權限         |
| `POST`   | `/batch`                               | 批次設定分類權限         |
| `DELETE` | `/{categoryId}`                        | 移除權限（恢復預設允許）   |

---

## GET `/` — 取得使用者的分類權限列表 🔒

**Path Parameters**

| 參數     | 類型      | 說明      |
| -------- | --------- | --------- |
| `userId` | `integer` | 使用者 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "取得成功",
    "data": {
        "permissions": [
            {
                "id": 1,
                "user_id": 2,
                "category_id": 5,
                "permission": "deny",
                "reason": "違反使用條款",
                "category": {
                    "id": 5,
                    "name": "首頁橫幅",
                    "slug": "home-banner"
                }
            }
        ]
    }
}
```

---

## PUT `/` — 設定使用者的單一分類權限 🔒

**Path Parameters**

| 參數     | 類型      | 說明      |
| -------- | --------- | --------- |
| `userId` | `integer` | 使用者 ID |

**Request Body（JSON）**

| 欄位          | 類型      | 必填 | 說明                                |
| ------------- | --------- | ---- | ----------------------------------- |
| `category_id` | `integer` | ✅   | 分類 ID                             |
| `permission`  | `string`  | ✅   | 權限設定（`allow` 或 `deny`）       |
| `reason`      | `string`  | ❌   | 設定原因（max:500）                 |

**請求範例**

```json
{
    "category_id": 5,
    "permission": "deny",
    "reason": "多次發布違規廣告"
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "權限設定成功",
    "data": {
        "permission": {
            "id": 1,
            "user_id": 2,
            "category_id": 5,
            "permission": "deny",
            "reason": "多次發布違規廣告",
            "category": {
                "id": 5,
                "name": "首頁橫幅",
                "slug": "home-banner"
            }
        }
    }
}
```

---

## POST `/batch` — 批次設定使用者的分類權限 🔒

一次性寫入多筆分類權限，可混合 `allow` 與 `deny`。

**Path Parameters**

| 參數     | 類型      | 說明      |
| -------- | --------- | --------- |
| `userId` | `integer` | 使用者 ID |

**Request Body（JSON）**

| 欄位                       | 類型      | 必填 | 說明                          |
| -------------------------- | --------- | ---- | ----------------------------- |
| `permissions`              | `array`   | ✅   | 權限設定陣列（至少 1 筆）     |
| `permissions[].category_id`| `integer` | ✅   | 分類 ID                       |
| `permissions[].permission` | `string`  | ✅   | `allow` 或 `deny`             |
| `permissions[].reason`     | `string`  | ❌   | 設定原因                      |

**請求範例**

```json
{
    "permissions": [
        {
            "category_id": 5,
            "permission": "deny",
            "reason": "違反規範"
        },
        {
            "category_id": 8,
            "permission": "allow"
        }
    ]
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "批次權限設定成功",
    "data": {
        "permissions": [
            {
                "id": 10,
                "category_id": 5,
                "permission": "deny"
            },
            {
                "id": 11,
                "category_id": 8,
                "permission": "allow"
            }
        ]
    }
}
```

---

## DELETE `/{categoryId}` — 移除使用者的分類權限 🔒

刪除特定權限紀錄後，該使用者對該分類的權限將恢復為系統預設（允許購買）。

**Path Parameters**

| 參數         | 類型      | 說明      |
| ------------ | --------- | --------- |
| `userId`     | `integer` | 使用者 ID |
| `categoryId` | `integer` | 分類 ID   |

**回應範例（200）**

```json
{
    "success": true,
    "message": "權限已移除"
}
```
