# 设计文档: 管理员后台重构

## Context

当前管理员功能分散在 `pages/admin/` 下的多个页面：
- `users/` - 用户管理（基础CRUD）
- `roles/` - 角色权限配置
- `config/` - 系统配置
- `audit-logs/` - 审计日志
- `duplicates/` - 重复检测

存在问题：
1. 无统一入口，管理员登录后进入工单工作台
2. 用户管理功能不完整（缺少启停用、重置密码）
3. 缺少公告管理和消息模板管理能力
4. 数据源混用（后端API + 云函数）

## Goals / Non-Goals

### Goals
- 提供统一的管理台入口
- 完善账号管理全生命周期功能
- 新增公告管理能力
- 新增消息模板管理能力
- 统一使用云函数/云数据库作为数据源

### Non-Goals
- 不实现字段级权限（仅页面/按钮级）
- 不实现数据范围权限
- 不实现系统工具（P2）
- 不重构后端API

## Decisions

### Decision 1: 数据源统一
- **选择**: 统一使用云函数/云数据库
- **理由**:
  - 保持与现有架构一致
  - 避免权限和数据不一致问题
  - 简化部署和维护

### Decision 2: 权限模型
- **选择**: 页面/按钮级权限
- **实现方式**:
  - 前端：根据 `userInfo.role_id` 控制页面/按钮可见性
  - 云端：每个云函数操作校验 `role_id === 1`（系统管理员）
- **理由**: 最易落地，满足当前需求

### Decision 3: 公告数据模型
```javascript
// announcements 集合
{
  _id: string,              // 自动生成
  title: string,            // 公告标题
  content: string,          // 公告内容（支持富文本HTML）
  status: 'draft' | 'published' | 'offline',  // 状态
  visible_roles: number[],  // 可见角色ID数组
  publish_time: Date,       // 发布时间
  expire_time: Date | null, // 过期时间（可选）
  created_by: number,       // 创建人ID
  created_at: Date,
  updated_at: Date
}
```

### Decision 4: 消息模板数据模型
```javascript
// message_templates 集合
{
  _id: string,
  scene: string,            // 场景标识
  scene_name: string,       // 场景名称
  title: string,            // 模板标题
  content: string,          // 模板内容（支持变量占位）
  variables: [              // 可用变量说明
    { key: 'order_id', name: '工单编号' },
    { key: 'status', name: '工单状态' }
  ],
  enabled: boolean,         // 是否启用
  created_at: Date,
  updated_at: Date
}

// 预设场景
const MESSAGE_SCENES = {
  ORDER_CREATED: 'order_created',       // 工单创建
  ORDER_STATUS_CHANGED: 'order_status_changed', // 状态变更
  ORDER_REMINDER: 'order_reminder',     // 催办提醒
  ANNOUNCEMENT: 'announcement'          // 公告通知
};
```

### Decision 5: 管理员登录跳转
- 在 `app.js` 的 `onLaunch` 或登录成功后判断
- `role_id === 1` 跳转到 `/pages/admin/dashboard/index`
- 其他角色跳转到原有工作台

## Risks / Trade-offs

### Risk 1: 云函数性能
- **风险**: 公告/模板列表查询可能较慢
- **缓解**: 添加适当索引，使用分页

### Risk 2: 权限绕过
- **风险**: 仅前端隐藏可能被绕过
- **缓解**: 云端每个操作都校验 role_id

### Risk 3: 现有功能影响
- **风险**: 修改登录跳转可能影响现有用户
- **缓解**: 仅影响 role_id=1 的管理员账号

## Migration Plan

1. 先部署云函数和数据库变更
2. 部署管理台页面（不影响现有功能）
3. 最后修改登录跳转逻辑

## Open Questions

~~1. 公告是否需要支持富文本？~~ **已确认：支持富文本HTML**
~~2. 消息模板变量替换是在前端还是云端执行？~~ **已确认：云端执行**
~~3. 是否需要管理台首页的统计卡片？~~ **已确认：不需要**

### Decision 6: 消息模板变量替换
- **选择**: 在云端执行变量替换
- **理由**:
  - 统一处理逻辑，避免前后端不一致
  - 云端可访问完整数据，变量值更准确
  - 前端只负责展示最终消息内容
- **实现**: 云函数在发送消息时，读取模板并替换 `{{变量名}}` 为实际值
