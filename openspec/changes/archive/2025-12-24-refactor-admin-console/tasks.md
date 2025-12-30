# 管理员后台重构 - 实现任务清单

## 1. 基础设施

- [x] 1.1 扩展云数据库集合定义
  - 新增 `announcements` 集合
  - 新增 `message_templates` 集合
  - 添加相应索引

- [x] 1.2 扩展云函数 `userAuth`
  - 添加公告管理相关操作 (create/update/delete/list/publish/offline)
  - 添加消息模板管理相关操作 (create/update/delete/list/toggle)
  - 添加用户管理增强操作 (enable/disable/resetPassword)

- [x] 1.3 扩展 `services/cloudDatabase.js`
  - 添加 announcements 相关方法
  - 添加 messageTemplates 相关方法
  - 添加 users 增强方法 (enable/disable/resetPassword)
  - **已修复**: 参数传递格式与云函数匹配

- [x] 1.4 扩展 `utils/constants.js`
  - 添加管理模块权限常量
  - 添加公告状态常量
  - 添加消息模板场景常量

## 2. 管理台首页 (P0)

- [x] 2.1 创建管理台首页页面
  - 新建 `pages/admin/dashboard/index.wxml|wxss|js|json`
  - 卡片入口：账号管理、角色与权限、系统配置、公告管理、消息模板、审计日志

- [x] 2.2 修改管理员登录跳转逻辑
  - 在 `login.js` 中判断管理员角色(role_id=1)
  - 管理员登录后跳转到管理台首页

- [x] 2.3 注册页面路由
  - 在 `app.json` 中添加管理台首页路由

## 3. 账号管理增强 (P0)

- [x] 3.1 用户列表页面增强
  - 添加筛选功能（角色、部门、启用状态、关键字）
  - 添加启用/停用按钮
  - 添加重置密码按钮

- [x] 3.2 新增/编辑用户表单完善
  - 完整字段：用户名、密码、姓名、角色、部门、手机号、启用状态
  - 表单验证

- [x] 3.3 防自我降级/禁用逻辑
  - 前端禁用当前用户的角色修改/禁用按钮
  - 云端校验防止绕过

## 4. 公告管理 (P0)

- [x] 4.1 创建公告列表页面
  - 新建 `pages/admin/announcements/index.wxml|wxss|js|json`
  - 列表展示：标题、状态、可见角色、创建时间
  - 发布/下线操作

- [x] 4.2 创建公告编辑页面
  - 新建 `pages/admin/announcements/edit/index.wxml|wxss|js|json`
  - 集成富文本编辑器（使用微信小程序 editor 组件）
  - 表单：标题、内容（富文本）、可见角色（多选）、有效期

- [x] 4.3 富文本展示组件
  - 使用 rich-text 组件渲染公告内容
  - 处理HTML内容安全性

- [x] 4.4 普通用户查看公告
  - 修改 `pages/message-list/index.js` 支持公告类型
  - 从云端获取当前用户可见的公告列表
  - **已修复**: 从 mock 数据改为真实云端数据

## 5. 消息模板管理 (P0)

- [x] 5.1 创建消息模板列表页面
  - 新建 `pages/admin/message-templates/index.wxml|wxss|js|json`
  - 列表展示：场景、标题、状态、更新时间
  - 启用/停用操作

- [x] 5.2 创建消息模板编辑页面
  - 新建 `pages/admin/message-templates/edit/index.wxml|wxss|js|json`
  - 表单：场景选择、标题、正文模板、变量说明
  - 预览功能

- [x] 5.3 预设场景模板
  - 工单创建通知
  - 工单状态更新通知
  - 工单催办提醒
  - 公告发布通知

## 6. 测试与验证

- [ ] 6.1 管理员角色登录跳转测试
- [ ] 6.2 账号管理CRUD测试
- [ ] 6.3 公告发布/下线/可见范围测试
- [ ] 6.4 消息模板启用/停用测试
- [ ] 6.5 权限校验测试（非管理员无法访问管理页面）

## 依赖关系

```
1.基础设施 ──┬── 2.管理台首页
             ├── 3.账号管理增强
             ├── 4.公告管理
             └── 5.消息模板管理
                     │
                     └── 6.测试与验证
```

## 并行化建议

- 2.管理台首页、3.账号管理增强 可并行开发
- 4.公告管理、5.消息模板管理 可并行开发（依赖1完成后）

## 实施记录

### 2025-12-18 修复记录

修复了服务层与云函数之间的参数传递不一致问题：

1. **cloudDatabase.js - users 服务**
   - `enable/disable/resetPassword` 参数格式从 `{ action, userId }` 改为 `{ action, data: { user_id } }`
   - 返回值判断从 `code !== 0` 改为 `!result.result.success`

2. **cloudDatabase.js - announcements 服务**
   - 所有方法参数统一为 `data: { ... }` 格式
   - 返回值统一为 `{ list: result.result.announcements }`

3. **cloudDatabase.js - messageTemplates 服务**
   - 所有方法参数统一为 `data: { ... }` 格式
   - 返回值统一为 `{ list: result.result.templates }`

4. **message-list/index.js**
   - 公告模块从 mock 数据改为调用 `cloudDB.announcements.listForUser(roleId)`
   - 新增 `formatDate()` 和 `stripHtml()` 辅助方法
