# Admin API - 廣告審核統籌
**身份驗證:** `auth:admin`   
**所需模組權限:** `ads`

> 這個模組掌控廣告從「送審」(Pending) 到「結案上架」或是包含異常下架等全生命週期控制。

## [GET] `/api/admin/ads`
取得全平台建立的廣告列表，包含草稿 (Draft)、審核中 (Pending)、上架中 (Active) 的廣告。
- **權限要求:** `admin.permission:ads,view`
- **Query Params:** 支援 `status`, `category_id`, 以及 `user_id` 篩選過濾。可一次檢視待審批列隊。

## [GET] `/api/admin/ads/{id}`
取得單一廣告詳細內容（含方案資訊、會員提供的圖片、詳細文案與過往審核紀錄）。
- **權限要求:** `admin.permission:ads,view`
- **預留擴充:** 未來會加入審核紀錄檔 `ad_review_logs` 方便歷史追蹤。

## [POST] `/api/admin/ads/{id}/approve`
核准該廣告。審核通過！
- **權限要求:** `admin.permission:ads,update`
- **邏輯連動:** 若廣告綁定的訂單已付清，系統通常會在核准當下自動生效並轉為 `active` 等待聯播網排程。

## [POST] `/api/admin/ads/{id}/reject`
退回有問題的廣告，將狀態撥為 `rejected` 讓客戶重新修改重送。
- **權限要求:** `admin.permission:ads,update`
- **Payload Validation:** `reason` (string, required) - 必填退件原因供客戶前台檢視。

## [POST] `/api/admin/ads/{id}/activate`
直接以系統管理員身份強制讓廣告於前台播送。
- **權限要求:** `admin.permission:ads,update`

## [POST] `/api/admin/ads/{id}/deactivate`
強制將異常的廣告（如有違規情事）立即下架停止前台播送，狀態轉為 `inactive`。
- **權限要求:** `admin.permission:ads,update`
