# Supplier Module (Suppliers + Supplier Products) Design

**Goal:** 将供应商独立为 `suppliers` 模块，提供两个菜单：供应商列表（纯列表）与供应商产品列表（供应商行 + 产品关系子表分页），并支持供应商类型多选、联系人/账户/标签子表，以及“产品唯一默认供应商”的强校验规则。

**Scope:** 后端（am-erp-go）+ 前端（am-erp-vue）+ 初始化测试数据。

---

## Architecture

- 新增 `suppliers` 独立模块（domain/usecase/repository/delivery）与数据库表。
- 供应商与产品通过 `product_suppliers` 关系表关联，包含供应价、账期、MOQ、交期、币种等业务字段。
- `product.default_supplier_id` 作为强校验字段：每个产品必须且仅一个默认供应商；设置默认时联动更新关系表与产品表。

---

## Data Model (MySQL)

### supplier
- id (bigint unsigned, PK)
- name (varchar)
- status (varchar, ACTIVE/DISABLED)
- remark (text)
- gmt_create, gmt_modified (datetime)
- 规则：名称唯一或同名提示

### supplier_type
- id
- supplier_id
- type (PRODUCT/PACKAGING/LOGISTICS)

### supplier_contact
- id
- supplier_id
- name
- phone
- email
- position
- is_primary (tinyint)

### supplier_account
- id
- supplier_id
- bank_name
- bank_account
- currency
- tax_no
- payment_terms

### supplier_tag
- id
- supplier_id
- tag

### product_suppliers
- id
- product_id
- supplier_id
- is_default
- purchase_price (decimal)
- payment_terms (int days or enum)
- moq (decimal)
- lead_time_days (int)
- currency
- status

### product
- default_supplier_id (required)

---

## API

### 供应商列表
- GET `/api/v1/suppliers` (分页 + keyword + status + type)

### 供应商详情
- GET `/api/v1/suppliers/:id` (含类型/联系人/账户/标签)

### 供应商产品列表
- GET `/api/v1/suppliers/with-products` (供应商分页 + 产品关系分页摘要)
- GET `/api/v1/suppliers/:id/products` (单供应商产品关系分页)

### 关系维护
- POST `/api/v1/product-suppliers`
- PUT `/api/v1/product-suppliers/:id`
- DELETE `/api/v1/product-suppliers/:id`
- 设为默认时：强制替换旧默认并同步 `product.default_supplier_id`

---

## UI/UX

### 菜单 1：供应商列表（纯列表）
- 列：供应商信息（名称/类型）、联系人摘要、状态、标签摘要、操作
- 支持筛选：keyword/status/type

### 菜单 2：供应商产品列表（合成一行）
- 供应商为主行分页
- 行内子表分页显示产品关系：SKU/名称/供应价/账期/MOQ/交期/币种/默认标记
- 右侧操作：编辑供应商、绑定产品、设置默认

---

## Error Handling

- 参数校验与唯一性提示
- 默认供应商强校验冲突提示

---

## Testing

- 后端 domain/usecase 单测（默认供应商规则、类型多选）
- 前端交互测试（列表渲染/展开/分页/表单校验）

---

## Seed Data

- 3-5 家供应商
- 每家至少 2 个联系人
- 1-2 个账户
- 2-3 个标签
- 若干产品关系（含默认与非默认）
