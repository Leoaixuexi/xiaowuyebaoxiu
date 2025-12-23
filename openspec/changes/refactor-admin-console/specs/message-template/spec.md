## ADDED Requirements

### Requirement: Message Template CRUD
系统 SHALL 支持消息模板的创建、编辑、删除操作。

#### Scenario: Create template
- **WHEN** 管理员填写消息模板表单并提交
- **THEN** 模板保存到数据库

#### Scenario: Edit template
- **WHEN** 管理员修改模板内容并保存
- **THEN** 模板内容更新成功

#### Scenario: Delete template
- **WHEN** 管理员确认删除模板
- **THEN** 模板从数据库中删除

### Requirement: Message Template by Scene
系统 SHALL 按场景组织消息模板。

#### Scenario: Scene-based template list
- **WHEN** 管理员进入消息模板列表
- **THEN** 显示按场景分组的模板列表

#### Scenario: Predefined scenes
- **WHEN** 系统初始化
- **THEN** 预设以下场景的默认模板：
  - 工单创建通知
  - 工单状态更新通知
  - 工单催办提醒
  - 公告发布通知

### Requirement: Message Template Variables
系统 SHALL 支持模板中的变量占位符，变量替换在云端执行。

#### Scenario: Variable placeholder
- **WHEN** 管理员在模板内容中使用 `{{变量名}}` 格式
- **THEN** 云函数在发送消息时自动替换为实际值

#### Scenario: Cloud-side replacement
- **WHEN** 系统触发消息发送
- **THEN** 云端读取对应场景模板
- **AND** 云端替换所有变量占位符为实际数据
- **AND** 前端只接收最终渲染后的消息内容

#### Scenario: Variable documentation
- **WHEN** 管理员编辑模板
- **THEN** 显示该场景可用的变量列表及说明

### Requirement: Message Template Enable/Disable
系统 SHALL 支持启用或停用消息模板。

#### Scenario: Disable template
- **WHEN** 管理员停用某个模板
- **THEN** 该场景的消息不再自动发送

#### Scenario: Enable template
- **WHEN** 管理员启用某个模板
- **THEN** 该场景的消息按模板内容发送

### Requirement: Message Template Preview
系统 SHALL 支持预览模板效果。

#### Scenario: Preview with sample data
- **WHEN** 管理员点击预览按钮
- **THEN** 使用示例数据渲染模板并显示效果

### Requirement: Message Template Access Control
消息模板管理 SHALL 仅对系统管理员可访问。

#### Scenario: Admin access granted
- **WHEN** 系统管理员访问消息模板管理页面
- **THEN** 显示消息模板管理界面

#### Scenario: Non-admin access denied
- **WHEN** 非管理员尝试访问消息模板管理页面
- **THEN** 显示权限不足提示并导航返回

#### Scenario: View only for users
- **WHEN** 普通用户收到系统消息
- **THEN** 仅能查看消息内容，无法编辑模板
