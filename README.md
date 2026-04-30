# Ad Platform API 文件

## 基本資訊

- **Base URL**: `/api`
- **Response 格式**: JSON
- **認證機制**: JWT Bearer Token

---

## 認證說明

本平台分為兩套獨立的認證系統：

| 系統 | Guard | Header |
|------|-------|--------|
| 會員 (User) | `auth:api` | `Authorization: Bearer {token}` |
| 管理員 (Admin) | `auth:admin` | `Authorization: Bearer {token}` |

Token 由登入 API 取得，回傳的 `token` 欄位即為 Bearer Token。Token 為 JWT，預設有效期請參照系統設定（預設 60 分鐘）。

---

## 通用 Response 格式

所有 API 均採用統一的回應結構：

```json
{
  "success": true,
  "message": "操作成功",
  "data": {}
}
```

錯誤回應範例：

```json
{
  "success": false,
  "message": "錯誤說明",
  "data": null
}
```

---

## HTTP 狀態碼

| 狀態碼 | 說明 |
|--------|------|
| 200 | 成功 |
| 201 | 建立成功 |
| 302 | 重新導向（用於廣告點擊追蹤） |
| 400 | 業務邏輯錯誤 |
| 401 | 未授權（Token 無效或未提供） |
| 403 | 禁止存取（權限不足） |
| 404 | 資源不存在 |
| 422 | 驗證失敗 |
| 500 | 伺服器錯誤 |

---

## API 分類總覽

### 公開 API（無需認證）

| 路徑 | 方法 | 說明 |
|------|------|------|
| `/api/auth/register` | POST | 會員註冊 |
| `/api/auth/login` | POST | 會員登入 |
| `/api/admin/login` | POST | 管理員登入 |
| `/api/ad-plans` | GET | 瀏覽廣告方案列表 |
| `/api/ad-plans/{id}` | GET | 取得廣告方案詳情 |
| `/api/ad-plans/{planId}/options` | GET | 取得方案選項列表 |
| `/api/plan-options/{id}` | GET | 取得方案選項詳情 |
| `/api/public/ads` | GET | 瀏覽上架廣告（支援搜尋/分頁） |
| `/api/public/ads/{id}` | GET | 取得廣告詳情 |
| `/api/public/ads/{id}/impression` | POST | 記錄廣告曝光 |
| `/api/public/ads/{id}/click` | GET | 記錄廣告點擊（302 導向） |

### 會員 API（需 User JWT）

| 路徑 | 方法 | 說明 |
|------|------|------|
| `/api/auth/me` | GET | 取得目前登入會員資料 |
| `/api/auth/logout` | POST | 會員登出 |
| `/api/auth/refresh` | POST | 刷新 Token |
| `/api/auth/change-password` | PUT | 修改密碼 |
| `/api/auth/profile` | PUT | 更新個人資料 |
| `/api/categories` | GET | 取得分類列表 |
| `/api/categories/{id}` | GET | 取得分類詳情 |
| `/api/orders` | GET | 取得我的訂單列表 |
| `/api/orders` | POST | 建立訂單 |
| `/api/orders/with-ads` | POST | 建立訂單並同時建立廣告草稿 |
| `/api/orders/{orderNumber}` | GET | 取得訂單詳情 |
| `/api/orders/{orderNumber}/cancel` | POST | 取消訂單 |
| `/api/orders/{orderNumber}/pay` | POST | 支付訂單 |
| `/api/orders/{orderNumber}/refund` | POST | 申請退款 |
| `/api/orders/{orderNumber}/payments` | GET | 取得訂單付款記錄 |
| `/api/ads` | GET | 取得我的廣告列表 |
| `/api/ads` | POST | 建立廣告草稿 |
| `/api/ads/{id}` | GET | 取得廣告詳情 |
| `/api/ads/{id}` | PUT | 更新廣告內容 |
| `/api/ads/{id}` | DELETE | 刪除廣告 |
| `/api/ads/{id}/image` | POST | 上傳廣告圖片 |
| `/api/ads/{id}/submit` | POST | 提交廣告審核 |

### 管理員 API（需 Admin JWT）

請參閱 [admin/](./admin/) 目錄下的各功能分類文件。

---

## 文件目錄

- [public/auth.md](./public/auth.md) - 公開認證 API
- [public/ad-plans.md](./public/ad-plans.md) - 公開廣告方案 API
- [public/ads.md](./public/ads.md) - 公開廣告瀏覽與追蹤 API
- [user/auth.md](./user/auth.md) - 會員認證 API
- [user/categories.md](./user/categories.md) - 分類查詢 API
- [user/orders.md](./user/orders.md) - 訂單管理 API
- [user/ads.md](./user/ads.md) - 廣告管理 API
- [admin/auth.md](./admin/auth.md) - 管理員認證 API
- [admin/admins.md](./admin/admins.md) - 管理員帳號管理 API
- [admin/ads.md](./admin/ads.md) - 管理員廣告審核 API
- [admin/ad-plans.md](./admin/ad-plans.md) - 管理員廣告方案管理 API
- [admin/categories.md](./admin/categories.md) - 管理員分類管理 API
- [admin/orders.md](./admin/orders.md) - 管理員訂單管理 API
- [admin/permissions.md](./admin/permissions.md) - 管理員權限管理 API
- [admin/users.md](./admin/users.md) - 管理員使用者帳號管理 API
