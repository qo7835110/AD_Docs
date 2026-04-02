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
- **邏輯連動:** 若廣告綁定的訂單已付清，系統通常會在核准當下自動生效並轉為 `active` 等待聯播網排程。無須 Payload。

## [POST] `/api/admin/ads/{id}/reject`
退回有問題的廣告，將狀態撥為 `rejected` 讓客戶重新修改重送。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `reason` | string | required | 是 | 必填退件原因，供客戶前台檢視 |

### Payload 範例 (JSON)
```json
{
  "reason": "廣告主視覺圖片包含違規文字，請修改後重新送審。"
}
```

## [POST] `/api/admin/ads/{id}/activate`
直接以系統管理員身份強制讓廣告於前台播送。無 Payload。
- **權限要求:** `admin.permission:ads,update`

## [POST] `/api/admin/ads/{id}/deactivate`
強制將異常的廣告（如有違規情事）立即下架停止前台播送，狀態轉為 `inactive`。無 Payload。
- **權限要求:** `admin.permission:ads,update`

## [GET] `/api/admin/ads/{id}/stats`
取得指定廣告在特定日期區間內的點閱成效與相關統計數據（含曝光數、點擊數、點擊率 CTR 以及每日趨勢對照）。
- **權限要求:** `admin.permission:ads,view`
- **Query Params:**
  - `from` (string, optional) - 查詢起始日期 (Y-m-d)。預設為 30 天前。
  - `to` (string, optional) - 查詢結束日期 (Y-m-d)。預設為今天。
- **限制與校驗:** 最大查詢區間不可超過 365 天，若日期格式錯誤會回傳 `422 Unprocessable Entity`。
- **Response 結構 (JSON):**
```json
{
  "success": true,
  "message": "成功取得廣告統計",
  "data": {
    "ad_id": 1,
    "period": {
      "from": "2026-02-25",
      "to": "2026-03-26"
    },
    "total_impressions": 1500,
    "total_clicks": 45,
    "ctr": 0.03,
    "daily": [
      {
        "date": "2026-02-25",
        "impressions": 50,
        "clicks": 2,
        "ctr": 0.04
      }
    ]
  }
}
```
