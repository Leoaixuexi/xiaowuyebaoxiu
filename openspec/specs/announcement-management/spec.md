# announcement-management Specification

## Purpose
TBD - created by archiving change refactor-admin-console. Update Purpose after archive.
## Requirements
### Requirement: Announcement CRUD
系统 SHALL 支持公告的创建、编辑、删除操作，公告内容支持富文本格式。

#### Scenario: Create announcement
- **WHEN** 管理员填写公告表单并提交
- **THEN** 公告以草稿状态保存到数据库
- **AND** 返回公告列表页面

#### Scenario: Edit announcement with rich text
- **WHEN** 管理员使用富文本编辑器修改公告内容
- **THEN** 支持文字加粗、斜体、列表、链接等格式
- **AND** 公告内容以HTML格式保存

#### Scenario: Delete announcement
- **WHEN** 管理员确认删除公告
- **THEN** 公告从数据库中删除

### Requirement: Announcement Publish/Offline
系统 SHALL 支持公告的发布和下线操作。

#### Scenario: Publish announcement
- **WHEN** 管理员点击发布按钮
- **THEN** 公告状态变为已发布
- **AND** 公告对目标角色用户可见

#### Scenario: Offline announcement
- **WHEN** 管理员点击下线按钮
- **THEN** 公告状态变为已下线
- **AND** 公告对所有用户不可见

### Requirement: Announcement Visibility by Role
系统 SHALL 支持按角色配置公告可见范围。

#### Scenario: Configure visible roles
- **WHEN** 管理员在公告表单中选择可见角色（多选）
- **THEN** 仅选中角色的用户可以看到该公告

#### Scenario: All roles visible
- **WHEN** 管理员选择所有角色
- **THEN** 所有用户都可以看到该公告

### Requirement: Announcement Expiration
系统 SHALL 支持设置公告有效期。

#### Scenario: Set expiration time
- **WHEN** 管理员设置公告有效期
- **THEN** 公告在有效期结束后自动不再显示

#### Scenario: No expiration
- **WHEN** 管理员不设置有效期
- **THEN** 公告持续显示直到手动下线

### Requirement: User View Announcements
普通用户 SHALL 能在消息页面查看对其可见的公告。

#### Scenario: View announcement list
- **WHEN** 用户进入消息页面的"通知公告"
- **THEN** 显示对该用户角色可见的已发布公告列表

#### Scenario: View announcement detail
- **WHEN** 用户点击公告标题
- **THEN** 显示公告详情内容

### Requirement: Announcement Access Control
公告管理 SHALL 仅对系统管理员可访问。

#### Scenario: Admin access granted
- **WHEN** 系统管理员访问公告管理页面
- **THEN** 显示公告管理界面

#### Scenario: Non-admin access denied
- **WHEN** 非管理员尝试访问公告管理页面
- **THEN** 显示权限不足提示并导航返回

