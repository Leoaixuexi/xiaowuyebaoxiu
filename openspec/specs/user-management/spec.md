# user-management Specification

## Purpose
TBD - created by archiving change refactor-admin-console. Update Purpose after archive.
## Requirements
### Requirement: User Enable/Disable
系统 SHALL 支持启用或停用用户账号。

#### Scenario: Disable user
- **WHEN** 管理员点击停用按钮
- **THEN** 用户账号被标记为停用状态
- **AND** 该用户无法登录系统

#### Scenario: Enable user
- **WHEN** 管理员点击启用按钮
- **THEN** 用户账号被标记为启用状态
- **AND** 该用户可以正常登录系统

#### Scenario: Prevent self-disable
- **WHEN** 管理员尝试停用自己的账号
- **THEN** 系统拒绝操作并提示不能停用自己

### Requirement: Reset User Password
系统 SHALL 支持重置用户密码。

#### Scenario: Reset password success
- **WHEN** 管理员点击重置密码按钮并确认
- **THEN** 用户密码被重置为默认密码
- **AND** 显示重置成功提示

#### Scenario: Reset password with notification
- **WHEN** 用户有绑定手机号且重置密码成功
- **THEN** 系统发送短信通知用户（可选功能）

### Requirement: Prevent Self Demotion
系统 SHALL 阻止管理员降低自己的角色权限。

#### Scenario: Self role change blocked
- **WHEN** 管理员尝试修改自己的角色为非管理员
- **THEN** 系统拒绝操作并提示不能修改自己的角色

#### Scenario: Cloud validation
- **WHEN** 通过API绕过前端尝试修改自己的角色
- **THEN** 云端校验拒绝该操作

