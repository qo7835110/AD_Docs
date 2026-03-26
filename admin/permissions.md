# Admin API - 權限治理核心 (Permissions)
**身份驗證:** `auth:admin`   
**所需模組權限:** 極機密 `category_permissions` 或 `admin_permissions`

> 系統內最高層級的安全管理。分為「使用者私有白名單設定」與「後台子帳號的模組分發指派」。

---

## ── 會員端的私有分類權限 (Category Permissions) ──

**權限要求:** 操作者必須具有 `admin.permission:category_permissions` 系列全限。

## [GET] `/api/admin/users/{userId}/category-permissions`
盤點該會員特例能購買哪些受到管制、非公開的廣告分類。
- **Response:** 列舉 Array of Category Objects。

## [PUT] `/api/admin/users/{userId}/category-permissions`
單筆加入一筆分類的開通權或封鎖。
- **Payload:** `category_id`, `status`

## [POST] `/api/admin/users/{userId}/category-permissions/batch`
批次一次性開啟極多個私有分類。特別適用於合約客戶。

## [DELETE] `/api/admin/users/{userId}/category-permissions/{categoryId}`
強硬拔除該使用者的白名單資格。拔除後任何該類別的草稿建立皆會 `403 Forbidden`。

---

## ── 後台管理員子權限指派 (Admin Permissions) ──

**權限要求:** 操作者通常為 SuperAdmin，需具備 `admin.permission:admin_permissions`。

## [GET] `/api/admin/permissions/modules`
列出系統所有設計為可權限化的架構元件表單（包含：ads, categories, ad_plans 等）。
- 供前端動態渲染出 Matrix-checked 開關列表供主管理打勾。

## [POST] `/api/admin/admins/{adminId}/permissions`
新增一項細部權限給特定員工。
- **Payload:** `module` (e.g. ads), `action` (e.g., update)

## [DELETE] `/api/admin/admins/{adminId}/permissions`
拔除該員工的一項功能入口與後端攔截。

## [PUT] `/api/admin/admins/{adminId}/permissions/batch`
用前端送入的新 Array，覆蓋清理舊有的權限設定（最常見的全重設做法）。
