# data-page Specification

## Purpose
TBD - created by archiving change add-manager-data-analytics. Update Purpose after archive.
## Requirements
### Requirement: 基于角色的数据页面视图

数据页面 SHALL 基于已认证用户的角色渲染不同的视图，以提供适当的统计和分析数据。

#### Scenario: 物业员工查看个人统计

- **WHEN** role_id = 4（物业员工）的用户访问 /pages/data/index
- **THEN** 页面必须显示按 submitter_id 过滤的个人工单统计
- **AND** 统计必须包括：今日提报、维修中、待复核、已完成

#### Scenario: 维修员查看个人统计

- **WHEN** role_id = 3（维修员）的用户访问 /pages/data/index
- **THEN** 页面必须显示按 assigned_technician_id 过滤的个人工单统计
- **AND** 统计必须包括：今日维修、已修复、需重修、已完成

#### Scenario: 物业经理查看全局分析

- **WHEN** role_id = 2（物业经理）的用户访问 /pages/data/index
- **THEN** 页面必须显示全局工单分析，涵盖所有工单且无用户过滤
- **AND** 视图必须包括多维度时间过滤器和可视化图表

### Requirement: 数据页面认证和授权

数据页面 SHALL 在渲染任何统计数据之前验证用户认证和角色。

#### Scenario: 未认证访问尝试

- **WHEN** 未认证用户尝试访问 /pages/data/index
- **THEN** 系统必须重定向到登录页面
- **AND** 不得获取或显示任何统计数据

#### Scenario: 未授权角色访问

- **WHEN** role_id 不在 [2, 3, 4] 范围内的用户访问 /pages/data/index
- **THEN** 系统必须显示权限不足的错误消息
- **AND** 不得获取或显示任何统计数据

