# admin-dashboard Specification

## Purpose
TBD - created by archiving change refactor-admin-console. Update Purpose after archive.
## Requirements
### Requirement: Admin Dashboard Entry
系统管理员登录后 SHALL 默认跳转到管理台首页，而非工单工作台。

#### Scenario: Admin login redirect
- **WHEN** 用户以系统管理员角色(role_id=1)成功登录
- **THEN** 系统跳转到管理台首页 `/pages/admin/dashboard/index`

#### Scenario: Non-admin login unchanged
- **WHEN** 用户以非管理员角色登录
- **THEN** 系统保持原有跳转逻辑到工单工作台

### Requirement: Admin Dashboard Layout
管理台首页 SHALL 以卡片形式展示所有管理功能入口。

#### Scenario: Dashboard cards display
- **WHEN** 管理员进入管理台首页
- **THEN** 显示以下功能卡片入口：
  - 账号管理
  - 角色与权限
  - 系统配置
  - 公告管理
  - 消息模板
  - 审计日志

#### Scenario: Card navigation
- **WHEN** 管理员点击功能卡片
- **THEN** 跳转到对应的管理页面

### Requirement: Admin Dashboard Access Control
管理台首页 SHALL 仅对系统管理员可访问。

#### Scenario: Non-admin access denied
- **WHEN** 非管理员角色尝试访问管理台首页
- **THEN** 显示权限不足提示并导航返回

