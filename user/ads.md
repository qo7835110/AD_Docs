# User API - 會員廣告管理
**權限:** `auth:api` (需攜帶 Bearer Token)

## [GET] `/api/ads`
取得會員自己建立的所有廣告清單（草稿、審核中、上架中、下架等各種狀態）。無 Payload 需求。

---

## [POST] `/api/ads`
建立新的「廣告草稿」。

### Payload 說明
| Schema | 型別 | 驗證規則 | 必填 | 說明 |
|---|---|---|---|---|
| `ad_plan_id` | integer | exists:ad_plans,id | 是 | 想對應的廣告母方案 ID |
| `title` | string | max:255 | 是 | 廣告主打標題 |
| `content` | string | | 是 | 廣告詳細文字與排版內文 |

### Payload 範例 (JSON)
```json
{
  "ad_plan_id": 5,
  "title": "2026 春季大促銷 - 全館滿千送百",
  "content": "年度最強檔期開始了，結帳輸入優惠碼即可享有八折！"
}
```
> **重點商務邏輯：** 建立廣告前，Server 會利用 `CategoryPermission` 檢查該會員是否具備該廣告方案（所屬分類）的合法購買權限。若不具備，將拋出 `403 Forbidden`。新建立的廣告預設狀態一律為草稿 (Draft)。

---

## [GET] `/api/ads/{id}`
取得會員擁有的單一廣告詳細內容（含審核回饋留言）。

---

## [PUT] `/api/ads/{id}`
更新廣告內容。格式與建立 `[POST] /api/ads` 完全相同。
> **業務邏輯限制：** 僅限狀態為「草稿 (`draft`)」或「被退回 (`rejected`)」的廣告可以覆蓋更新。

---

## [DELETE] `/api/ads/{id}`
刪除尚未發布且未鎖定的廣告草稿。

---

## [POST] `/api/ads/{id}/image`
上傳廣告附屬圖片（如主視覺 Banner）。

### Payload 說明
此 Request 必須以 `multipart/form-data` 送出：
- `image` (file, required, valid image formats: jpeg/png, size < 2048KB)

---

## [POST] `/api/ads/{id}/submit`
將草稿提交送審。
- **Payload:** 此為單純動作驅發的 Endpoint，無須夾帶 Payload 資料。
- **邏輯：** 廣告的狀態將變更為 `pending`，鎖定更新權限並移交管理員列隊。
