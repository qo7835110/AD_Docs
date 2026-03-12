# AD Platform API 文件

> **Base URL：** `http://localhost:8000/api`
> **版本：** v1.0.0
> **日期時間格式：** `Y-m-d H:i:s`（所有 datetime 欄位）

## 驗證方式

API 使用 **JWT Bearer Token** 進行身份驗證。

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
  "data": null
}
```

## API 分類目錄

| 分類            | 說明                                   | 文件                                 |
| --------------- | -------------------------------------- | ------------------------------------ |
| 🔐 Auth         | 會員認證（註冊、登入、登出、個人資料） | [auth.md](./auth.md)                 |
| 👤 Guest Auth   | 訪客認證（無需正式會員）               | [guest-auth.md](./guest-auth.md)     |
| 📁 Categories   | 廣告分類管理（CRUD + 軟刪除還原）      | [categories.md](./categories.md)     |
| 📋 Ad Plans     | 廣告方案管理（CRUD）                   | [ad-plans.md](./ad-plans.md)         |
| ⚙️ Plan Options | 方案選項管理（CRUD）                   | [plan-options.md](./plan-options.md) |
| 🛒 Orders       | 訂單管理（建立、付款、退款）           | [orders.md](./orders.md)             |

## HTTP 狀態碼

| 狀態碼 | 說明                        |
| ------ | --------------------------- |
| `200`  | 成功                        |
| `201`  | 建立成功                    |
| `400`  | 請求錯誤（業務邏輯失敗）    |
| `401`  | 未授權 / Token 無效或已過期 |
| `404`  | 資源不存在                  |
| `422`  | 驗證失敗（欄位格式錯誤）    |
| `500`  | 伺服器內部錯誤              |

## 路由前綴總覽

```
/api/auth/...
/api/guest/...
/api/categories/...
/api/ad-plans/...
/api/plan-options/...
/api/orders/...
```
