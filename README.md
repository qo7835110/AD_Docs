# AD Platform API 文件

> **Base URL：** `http://localhost:8000/api`
> **版本：** v1.0.0
> **最後更新：** 2026-03-13
> **日期時間格式：** `Y-m-d H:i:s`（所有 datetime 欄位）

## 驗證方式

API 使用 **JWT Bearer Token** 進行身份驗證，系統提供兩組 Guard：

| Guard   | 對象     | 說明                        |
| ------- | -------- | --------------------------- |
| `api`   | 會員     | 註冊會員，使用 `users` 表   |
| `admin` | 管理員   | 後台管理員，使用 `admins` 表 |

請求時需在 Header 帶入：

```http
Authorization: Bearer {your_access_token}
```

## 回應格式

所有 API 回應均為 JSON，結構如下：

```json
// 成功
{
  "success": true,
  "message": "操作說明",
  "data": { ... }
}

// 失敗
{
  "success": false,
  "message": "錯誤說明",
  "errors": { ... }
}
```

## API 分類目錄

| 分類            | 說明                                       | 文件                                 |
| --------------- | ------------------------------------------ | ------------------------------------ |
| 🔐 Auth         | 會員認證（註冊、登入、登出、JOB 第三方）   | [auth.md](./auth.md)                 |
| ️ Admin Auth   | 管理員認證（後台登入、密碼管理）           | [admin-auth.md](./admin-auth.md)     |
| 📁 Categories   | 廣告分類管理（會員瀏覽 + 管理員 CRUD）    | [categories.md](./categories.md)     |
| 📋 Ad Plans     | 廣告方案（公開瀏覽 + 管理員 CRUD）        | [ad-plans.md](./ad-plans.md)         |
| ⚙️ Plan Options | 方案選項管理（公開瀏覽 + 管理員 CRUD）    | [plan-options.md](./plan-options.md) |
| 🛒 Orders       | 訂單管理（建立、付款、退款、含廣告下單）   | [orders.md](./orders.md)             |
| 📢 Ads          | 廣告管理（會員建立/送審 + 管理員審核上架） | [ads.md](./ads.md)                   |
| 🔒 Category Perms | 分類購買權限管理（管理員 CRUD）          | [category-permissions.md](./category-permissions.md) |
| 🔑 Admin Perms  | 管理員 API 權限管理（模組式 CRUD 控制）    | [admin-permissions.md](./admin-permissions.md) |

## HTTP 狀態碼

| 狀態碼 | 說明                        |
| ------ | --------------------------- |
| `200`  | 成功                        |
| `201`  | 建立成功                    |
| `400`  | 請求錯誤（業務邏輯失敗）    |
| `401`  | 未授權 / Token 無效或已過期 |
| `403`  | 禁止存取（權限不足）        |
| `404`  | 資源不存在                  |
| `422`  | 驗證失敗（欄位格式錯誤）    |
| `500`  | 伺服器內部錯誤              |

## 路由前綴總覽

```
公開路由
  POST   /api/auth/register
  POST   /api/auth/login
  POST   /api/admin/login
  GET    /api/ad-plans
  GET    /api/ad-plans/{id}
  GET    /api/ad-plans/{planId}/options
  GET    /api/plan-options/{id}
  GET    /api/public/ads
  GET    /api/public/ads/{id}

會員路由（auth:api）
  /api/auth/...          會員帳號管理
  /api/categories/...    分類瀏覽
  /api/orders/...        訂單管理
  /api/ads/...           廣告管理（使用者端）

管理員路由（auth:admin）
  /api/admin/...         管理員帳號管理
  /api/admin/ads/...     廣告審核管理
  /api/admin/categories/...  分類 CRUD
  /api/admin/ad-plans/...    廣告方案 CRUD
  /api/admin/users/*/category-permissions/...  分類權限管理
  /api/admin/admins/*/permissions/...  管理員權限管理
  /api/admin/permissions/modules       可用模組列表
```
