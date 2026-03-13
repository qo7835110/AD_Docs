# ⚙️ Plan Options — 方案選項管理 API

> 方案選項（`PlanOption`）屬於某個廣告方案（`AdPlan`）的子項目，可設定不同時效、價格的選購方案。
> **讀取端點為公開 API**，不需要驗證。
> **建立、更新、刪除為管理員專屬**，需要 `auth:admin` Guard 🔒。

---

## 端點索引

### 公開端點（無需驗證）

| 方法  | 路徑                             | 說明                 |
| ----- | -------------------------------- | -------------------- |
| `GET` | `/api/ad-plans/{planId}/options` | 取得方案下的所有選項 |
| `GET` | `/api/plan-options/{id}`         | 取得單一選項詳情     |

### 管理員端點（auth:admin 🔒）

| 方法     | 路徑                                        | 說明           |
| -------- | ------------------------------------------- | -------------- |
| `POST`   | `/api/admin/ad-plans/{planId}/options`       | 為方案新增選項 |
| `PUT`    | `/api/admin/ad-plans/options/{id}`           | 更新方案選項   |
| `DELETE` | `/api/admin/ad-plans/options/{id}`           | 刪除方案選項   |

---

## GET `/api/ad-plans/{planId}/options` — 取得方案選項列表

**Path Parameters**

| 參數     | 類型      | 說明        |
| -------- | --------- | ----------- |
| `planId` | `integer` | 廣告方案 ID |

**回應範例（200）**

```json
{
    "success": true,
    "data": [
        {
            "id": 1,
            "ad_plan_id": 1,
            "name": "3個月方案",
            "description": "季度優惠方案",
            "duration_days": 90,
            "price": "2699.00",
            "valid_start_date": "2026-01-01",
            "valid_end_date": "2026-12-31",
            "sort_order": 1,
            "created_at": "2026-01-01 00:00:00",
            "updated_at": "2026-01-01 00:00:00"
        }
    ]
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `404`  | 廣告方案不存在 |

---

## POST `/api/admin/ad-plans/{planId}/options` — 新增方案選項 🔒 Admin

**Path Parameters**

| 參數     | 類型      | 說明        |
| -------- | --------- | ----------- |
| `planId` | `integer` | 廣告方案 ID |

**Request Body（JSON）**

| 欄位               | 類型      | 必填 | 說明                                            |
| ------------------ | --------- | ---- | ----------------------------------------------- |
| `name`             | `string`  | ✅   | 選項名稱（max:255）                             |
| `description`      | `string`  | ❌   | 選項說明                                        |
| `duration_days`    | `integer` | ✅   | 有效天數（min:1）                               |
| `price`            | `integer` | ✅   | 價格（min:0）                                   |
| `valid_start_date` | `date`    | ❌   | 有效起始日（`Y-m-d`）                           |
| `valid_end_date`   | `date`    | ❌   | 有效結束日（`Y-m-d`，需 >= `valid_start_date`） |
| `sort_order`       | `integer` | ❌   | 排序                                            |

**請求範例**

```json
{
    "name": "6個月方案",
    "description": "半年優惠方案",
    "duration_days": 180,
    "price": 4999,
    "valid_start_date": "2026-01-01",
    "valid_end_date": "2026-12-31",
    "sort_order": 2
}
```

**回應範例（201）**

```json
{
    "success": true,
    "message": "方案選項建立成功",
    "data": {
        "id": 3,
        "ad_plan_id": 1,
        "name": "6個月方案",
        "price": "4999.00",
        "duration_days": 180,
        "valid_start_date": "2026-01-01",
        "valid_end_date": "2026-12-31",
        "sort_order": 2,
        "created_at": "2026-03-13 11:00:00"
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

## GET `/api/plan-options/{id}` — 取得單一選項詳情

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 選項 ID |

**回應範例（200）**

```json
{
    "success": true,
    "data": {
        "id": 1,
        "ad_plan_id": 1,
        "name": "3個月方案",
        "description": "季度優惠方案",
        "duration_days": 90,
        "price": "2699.00",
        "valid_start_date": "2026-01-01",
        "valid_end_date": "2026-12-31",
        "sort_order": 1
    }
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `404`  | 方案選項不存在 |

---

## PUT `/api/admin/ad-plans/options/{id}` — 更新方案選項 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 選項 ID |

**Request Body（JSON）**（所有欄位選填）

| 欄位               | 類型      | 說明                  |
| ------------------ | --------- | --------------------- |
| `name`             | `string`  | 選項名稱（max:255）   |
| `description`      | `string`  | 選項說明              |
| `duration_days`    | `integer` | 有效天數（min:1）     |
| `price`            | `integer` | 價格（min:0）         |
| `valid_start_date` | `date`    | 有效起始日（`Y-m-d`） |
| `valid_end_date`   | `date`    | 有效結束日（`Y-m-d`） |
| `sort_order`       | `integer` | 排序                  |

**請求範例**

```json
{
    "price": 5299,
    "sort_order": 3
}
```

**回應範例（200）**

```json
{
    "success": true,
    "message": "方案選項更新成功",
    "data": {
        "id": 1,
        "price": "5299.00",
        "sort_order": 3,
        "updated_at": "2026-03-13 12:00:00"
    }
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `401`  | 未授權         |
| `404`  | 方案選項不存在 |
| `422`  | 驗證失敗       |

---

## DELETE `/api/admin/ad-plans/options/{id}` — 刪除方案選項 🔒 Admin

**Path Parameters**

| 參數 | 類型      | 說明    |
| ---- | --------- | ------- |
| `id` | `integer` | 選項 ID |

**回應範例（200）**

```json
{
    "success": true,
    "message": "方案選項刪除成功"
}
```

**錯誤回應**

| 狀態碼 | 說明           |
| ------ | -------------- |
| `401`  | 未授權         |
| `404`  | 方案選項不存在 |
