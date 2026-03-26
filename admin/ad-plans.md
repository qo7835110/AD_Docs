# Admin API - 廣告方案與計費控管 (Ad Plans)
**身份驗證:** `auth:admin`   
**所需模組權限:** 初核 `ad_plans` 或 子維護 `plan_options`

> 大型重構核心之一：分為母體 `AdPlans`（決定整體規格限制）與子實體 `PlanOptions`（決定實際售價與天數計算）。

---

## ── 母層：廣告方案管理 (Ad Plans) ──

## [POST] `/api/admin/ad-plans`
替特定分類建立母層廣告方案。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `category_id` | integer | exists:categories,id | 是 | 分類隸屬 ID |
| `name` | string | max:255 | 是 | 廣告方案前台顯示名稱 |
| `ad_limit` | integer | min:1 | 否 | 同案下可供上傳之最大數量 |
| `status` | string | in:active,inactive | 否 | 初始發布狀態（預設 active） |
| `options[]` | array | | 否 | [陣列] 順便一併批次建立的子計費選項 |
| `options.*.name` | string | max:255 | 是 | 選項名稱 (必填，若有給 options) |
| `options.*.duration_days` | integer | min:1 | 是 | 廣告有效刊登長度 (天數) |
| `options.*.price` | integer | min:0 | 是 | 實際金流收費價格 |

### Payload 範例 (JSON)
```json
{
  "category_id": 1,
  "name": "首頁黃金版面 Banner",
  "ad_limit": 5,
  "status": "active",
  "options": [
    {
      "name": "7天短效期曝光",
      "duration_days": 7,
      "price": 1500,
      "sort_order": 1
    },
    {
      "name": "30天超值長版",
      "duration_days": 30,
      "price": 5000,
      "sort_order": 2
    }
  ]
}
```

## [PUT] `/api/admin/ad-plans/{id}`
更新該方案資訊與狀態（啟動/停用）。Payload 型態同 `POST` 但全欄位幾乎可接受 `sometimes` 獨立更新。

## [DELETE] `/api/admin/ad-plans/{id}`
整體刪除母層方案。
> **邏輯提示：** 所有繼承該母層的 `PlanOptions` 會受到管控防呆（若有活躍訂單可能拒絕刪除或進行 Soft Delete，確保財源紀錄穩定）。

---

## ── 子層：廣告計費選項管理 (Plan Options) ──

## [POST] `/api/admin/ad-plans/{planId}/options`
動態針對現有方案插入新的計費維度。例如「暑期行銷專案 14天」。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `name` | string | max:255 | 是 | 選項名稱 |
| `duration_days` | integer | min:1 | 是 | 廣告有效刊登長度 (天數) |
| `price` | integer | min:0 | 是 | 實際收費，可為 0 |
| `valid_start_date` | date  | date | 否 | 該計費選項之促銷起始日 |
| `valid_end_date` | date  | after:start_date | 否 | 該計費選項之截止日 |
| `sort_order` | integer | numeric | 否 | 排序優先級 |

### Payload 範例 (JSON)
```json
{
  "name": "快閃特惠 3天",
  "duration_days": 3,
  "price": 500,
  "valid_start_date": "2026-04-01",
  "valid_end_date": "2026-04-05",
  "sort_order": 0
}
```

## [PUT] `/api/admin/ad-plans/options/{id}`
更新該計費選項內容。需考慮已生成訂單鎖定狀態，故重大變更系統將內部以版本控制轉換或軟刪除替換。

## [DELETE] `/api/admin/ad-plans/options/{id}`
標記停售此計費選項。
