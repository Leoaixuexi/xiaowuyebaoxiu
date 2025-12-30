# Tasks: 字典管理功能

## 1. 云函数与数据层

- [ ] 1.1 创建 `dictionaryManager` 云函数
  - 实现 `list`, `get`, `getBatch`, `create`, `update`, `delete` actions
  - 添加管理员权限校验（role_id=1）
- [ ] 1.2 在 `initDatabase` 云函数中添加 `initDictionaries` action
  - 预置 4 个系统字典：floor, order_category, responsible_party, department
  - 设置 `is_system: true` 标记

## 2. 前端字典服务

- [ ] 2.1 创建 `services/dictionary.js`
  - 实现 `getDictionary(dictKey)` 方法
  - 实现 `getDictionaries(dictKeys)` 批量获取方法
  - 实现 `getOptions(dictKey)` 返回选项数组
  - 实现本地缓存（5分钟过期）
  - 实现硬编码兜底值 `FALLBACK_OPTIONS`
- [ ] 2.2 创建 `utils/dictFallback.js`
  - 定义各字典的默认值常量

## 3. 管理员页面

- [ ] 3.1 创建 `pages/admin/dict/index` 字典组列表页
  - 展示所有字典组（dict_key, dict_name, 项数）
  - 支持新增字典组（弹窗表单）
  - 点击进入字典项管理
- [ ] 3.2 创建 `pages/admin/dict/items` 字典项管理页
  - 展示当前字典的所有项
  - 支持新增/编辑/删除字典项（弹窗表单）
  - 支持拖拽排序或手动输入排序号
  - 支持启用/停用切换
- [ ] 3.3 在 `pages/admin/index` 添加"字典管理"入口
  - 新增卡片，跳转到字典列表页

## 4. 表单页面改造

- [ ] 4.1 改造 `pages/property/submit/index.js`
  - 从字典服务获取 floorOptions
  - 从字典服务获取 orderCategories
  - 从字典服务获取 responsibleParties
  - 加载失败时使用兜底值
- [ ] 4.2 改造 `pages/work-order-edit/index.js`
  - 从字典服务获取 categoryOptions
  - 从字典服务获取 responsibleOptions
- [ ] 4.3 改造 `pages/admin/users/add/index.js`
  - 从字典服务获取 departments
- [ ] 4.4 改造 `pages/admin/users/edit/index.js`
  - 从字典服务获取 departments

## 5. 验证与测试

- [ ] 5.1 测试字典管理 CRUD 功能
- [ ] 5.2 测试字典缓存和兜底逻辑
- [ ] 5.3 测试表单页面正常加载字典选项
- [ ] 5.4 测试云函数失败时的兜底表现

## Dependencies

- 1.1, 1.2 可并行开发
- 2.1 依赖 1.1 完成
- 3.x 依赖 1.1, 2.1 完成
- 4.x 依赖 2.1 完成
- 5.x 依赖 1-4 完成

## Notes

- 只有管理员（role_id=1）可以访问字典管理页面
- 系统字典（is_system=true）不可删除，但可以修改字典项
- 字典项只做逻辑删除（enabled=false），保证历史数据可识别
