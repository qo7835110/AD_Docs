# Admin API - 廣告方案與計費控管 (Ad Plans)
**身份驗證:** `auth:admin`   
**所需模組權限:** 初核 `ad_plans` 或 子維護 `plan_options`

> 大型重構核心之一：分為母體 `AdPlans`（決定整體規格限制）與子實體 `PlanOptions`（決定實際售價與天數計算）。

## ── 母層：廣告方案管理 (Ad Plans) ──

## [POST] `/api/admin/ad-plans`
替特定分類建立母層廣告方案。
- **權限要求:** `admin.permission:ad_plans,create`
- **Payload:** 
  - `category_id` (integer, required)
  - `name` (string, required)
  - `ad_limit` (integer, optional)
  - 可同時在 Payload 內包含 `options` 陣列做 Batch Insert。

## [PUT] `/api/admin/ad-plans/{id}`
更新該方案資訊與狀態（啟動/停用）。
- **權限要求:** `admin.permission:ad_plans,update`

## [DELETE] `/api/admin/ad-plans/{id}`
整體刪除母層方案。
- **權限要求:** `admin.permission:ad_plans,delete`
> **邏輯提示：** 所有繼承該母層的 `PlanOptions` 預計會被一併軟殺或無法繼續販售。歷史訂單依然參照原有快照(Snapshot)，不影響財報功能。

---

## ── 子層：廣告計費選項管理 (Plan Options) ──

## [POST] `/api/admin/ad-plans/{planId}/options`
動態針對現有方案插入新的計費維度。例如「新年特惠 7天 - 300元」。
- **權限要求:** `admin.permission:plan_options,create`
- **Payload:** 
  - `name` (string, e.g., '14 Days Basic')
  - `duration_days` (integer, required, e.g., 14)
  - `price` (integer, required, e.g., 3000)
  - `valid_start_date` / `valid_end_date` (檔期操作用限制時間)

## [PUT] `/api/admin/ad-plans/options/{id}`
更新該計費選項內容。
- **權限要求:** `admin.permission:plan_options,update`

## [DELETE] `/api/admin/ad-plans/options/{id}`
標記停售 / 抹除該計費選項，避免前端繼續產出無效訂單。
- **權限要求:** `admin.permission:plan_options,delete`
