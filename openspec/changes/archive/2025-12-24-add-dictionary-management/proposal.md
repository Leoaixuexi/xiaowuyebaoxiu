# Change: 新增字典管理功能

## Why

当前项目中存在多处硬编码的下拉选项（楼层、工单类别、责任方、优先级等），分散在前端、云函数、后端多个位置，维护困难且容易不一致。需要提供一个统一的字典管理功能，让管理员可以动态维护这些选项。

## What Changes

### 新增功能
- 新增 `dictionaries` 云数据库集合存储字典数据
- 新增 `dictionaryManager` 云函数提供字典 CRUD 接口
- 新增管理员"字典管理"页面模块（`pages/admin/dict/`）
- 新增前端字典服务（`services/dictionary.js`）支持缓存和兜底
- 修改现有表单页面，从字典服务获取选项数据

### 需要字典化的选项清单

| 字典键 (dict_key) | 字典名称 | 当前选项值 | 使用位置 |
|------------------|---------|-----------|---------|
| `floor` | 楼层 | 1楼, 2楼, 3楼, 4楼, 5楼, B1, B2 | 提交工单、工单筛选 |
| `order_category` | 工单类别 | 电梯维修, 水电维修, 消防维修, 空调维修, 其他 | 提交工单、编辑工单、云函数验证 |
| `responsible_party` | 责任方 | 信泰物业, 业主, 第三方 | 提交工单、编辑工单、通知逻辑 |
| `department` | 部门 | 行政部, 信泰物业, 工程总包, 供应商 | 用户管理 |

### 不纳入字典管理的选项（保持硬编码）
- **优先级** (`priority`): 与业务逻辑强绑定（SLA规则、颜色、排序）
- **工单状态** (`status`): 与状态机流转强绑定
- **角色** (`role`): 与权限系统强绑定
- **公告状态**: 系统内置状态

## Impact

- 新增 specs: `dictionary`
- 新增云函数: `dictionaryManager`
- 新增集合: `dictionaries`
- 修改页面:
  - `pages/property/submit/index.js` - 楼层、类别、责任方
  - `pages/work-order-edit/index.js` - 类别、责任方
  - `pages/admin/users/add/index.js` - 部门
  - `pages/admin/users/edit/index.js` - 部门
- 新增页面:
  - `pages/admin/dict/index` - 字典组列表
  - `pages/admin/dict/items` - 字典项管理
