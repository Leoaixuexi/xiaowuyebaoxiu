# 维修员账号功能开发计划

## 项目概述
基于现有物业员工账号的页面，适配维修员账号的差异化需求。**核心策略：共用页面，根据角色动态调整UI和数据**。

## 角色定义
- 物业员工：`role_id = 4` (PROPERTY_STAFF)
- 维修员：`role_id = 3` (MAINTENANCE_STAFF)

---

## 任务清单

### 1. 工作台页面改造 (pages/index/index)
**目标**：根据角色显示不同的状态筛选按钮和工单数据

#### 1.1 添加角色判断逻辑
- [ ] 在 `onLoad` 或 `onShow` 中获取当前用户角色和部门信息
- [ ] 将 `userRole` 和 `userDepartment` 存储到 data 中

#### 1.2 动态生成状态筛选按钮
- [ ] 创建两套状态按钮配置：
  - 物业员工：已提报、维修中、待复核、已完成
  - 维修员：待接单、维修中、已修复、需重修、已完成
- [ ] 根据角色动态设置状态按钮列表
- [ ] 更新 WXML 模板，使用动态数据渲染状态按钮

#### 1.3 调整工单数据过滤逻辑
- [ ] 修改 `loadWorkOrders` 方法：
  - 物业员工：过滤 `submitter.user_id === 当前用户ID` 的工单
  - 维修员：过滤 `responsible_party === 用户部门` 的工单
- [ ] 修改 `filterByStatus` 方法：
  - 维修员的"待接单" → `Pending Repair`
  - 维修员的"已修复" → `Repaired`
  - 维修员的"需重修" → `Needs Rework`

#### 1.4 隐藏维修员的新工单提交按钮
- [ ] 在 WXML 中添加条件判断 `wx:if="{{userRole === 4}}"`
- [ ] 只对物业员工显示浮动的"+"按钮

---

### 2. 工单详情页改造 (pages/work-order-detail/index)
**目标**：为维修员添加"接单"按钮

#### 2.1 添加"接单"按钮显示逻辑
- [ ] 在权限判断中添加 `canAcceptOrder` 变量
- [ ] 条件：`isMaintenanceWorker && workOrder.status === 'Pending Repair' && workOrder.responsible_party === userDepartment`

#### 2.2 实现"接单"功能
- [ ] 添加 `handleAcceptOrder` 方法
- [ ] 调用 `workOrderService.updateWorkOrderStatus` 将状态更新为 `In Progress`
- [ ] 添加状态变更记录：`维修员接单开始维修`
- [ ] 成功后刷新页面数据

#### 2.3 更新 WXML 模板
- [ ] 在按钮区域添加"接单"按钮
- [ ] 按钮样式使用绿色主题，与"开始维修"按钮区分

---

### 3. 数据页面改造 (pages/data/index)
**目标**：根据角色显示不同的统计模块

#### 3.1 动态生成统计卡片
- [ ] 创建两套统计配置：
  - 物业员工：今日提报、维修中、待复核、已完成
  - 维修员：待接单、维修中、已修复、需重修、已完成
- [ ] 根据角色设置 `stats` 数据

#### 3.2 实现统计数据获取
- [ ] 添加 `loadStatistics` 方法
- [ ] 根据角色调用不同的数据筛选逻辑：
  - 物业员工：统计自己提报的工单
  - 维修员：统计责任方=自己部门的工单

#### 3.3 隐藏维修员的排名模块
- [ ] 在 WXML 中添加条件判断 `wx:if="{{userRole === 4}}"`
- [ ] 只对物业员工显示"月度排名"模块

---

### 4. 消息页面改造 (pages/notifications/index)
**目标**：根据角色过滤和显示不同的消息

#### 4.1 扩展消息映射逻辑 (utils/message-mapper.js)
- [ ] 在 `EVENT_TO_MODULE` 中添加新的事件类型：
  - `work_order_pending_accept`: 新工单待接单（维修员）
  - `work_order_repaired`: 工单已修复待复核（物业员工）
- [ ] 在 `filterMessagesByModule` 中添加角色过滤

#### 4.2 修改消息加载逻辑
- [ ] 在 `loadData` 方法中获取当前用户角色
- [ ] 根据角色过滤消息：
  - 物业员工：只显示自己提报的工单相关消息
  - 维修员：只显示责任方=自己部门的工单消息

#### 4.3 调整"待办工单"模块内容
- [ ] 物业员工：显示状态为 `Repaired` 需要复核的工单消息
- [ ] 维修员：显示状态为 `Pending Repair` 待接单的工单消息

---

### 5. 消息推送逻辑实现（后端/云函数）
**目标**：在工单状态变更时推送消息给相关用户

#### 5.1 新工单提交时推送给维修员
- [ ] 物业员工提交新工单 → 状态为 `Pending Repair`
- [ ] 查询该工单的 `responsible_party`（责任方/部门）
- [ ] 查找所有该部门的维修员
- [ ] 发送消息：`有新的工单待接单：[工单编号]`

#### 5.2 工单已修复时推送给提报人
- [ ] 维修员提交修复 → 状态变为 `Repaired`
- [ ] 获取该工单的 `submitter.user_id`（提报人）
- [ ] 发送消息给提报人：`您提报的工单 [工单编号] 已修复，请复核`

---

### 6. 测试和验证
**目标**：确保两种角色的功能正常运行

#### 6.1 物业员工角色测试
- [ ] 测试工作台：只显示自己提报的工单
- [ ] 测试状态筛选：已提报、维修中、待复核、已完成
- [ ] 测试提交新工单功能
- [ ] 测试数据页面：统计和排名显示正常
- [ ] 测试消息：接收工单已修复的提醒

#### 6.2 维修员角色测试
- [ ] 测试工作台：只显示责任方=自己部门的工单
- [ ] 测试状态筛选：待接单、维修中、已修复、需重修、已完成
- [ ] 测试接单功能：将待接单工单变为维修中
- [ ] 测试数据页面：统计显示正常，无排名模块
- [ ] 测试消息：接收新工单待接单的提醒

#### 6.3 交互逻辑测试
- [ ] 物业员工提交工单 → 维修员收到待接单消息
- [ ] 维修员接单 → 工单状态变为维修中
- [ ] 维修员提交修复 → 物业员工收到待复核消息
- [ ] 物业员工复核通过 → 工单状态变为已完成

---

## 技术要点

### 角色判断
```javascript
const userInfo = await auth.getCurrentUser();
const isPropertyStaff = userInfo.role_id === 4;
const isMaintenanceWorker = userInfo.role_id === 3;
const userDepartment = userInfo.department;
```

### 状态映射（维修员视角）
- "待接单" → `Pending Repair` (物业员工提报后的初始状态)
- "维修中" → `In Progress` (接单后的状态)
- "已修复" → `Repaired` (提交修复完成)
- "需重修" → `Needs Rework` (复核不通过)
- "已完成" → `Completed` (复核通过)

### 数据过滤（共用一套数据，不同角色看到不同工单）
```javascript
// 物业员工
filteredOrders = allOrders.filter(order =>
  order.submitter && order.submitter.user_id === userInfo.id
);

// 维修员
filteredOrders = allOrders.filter(order =>
  order.responsible_party === userDepartment
);
```

---

## 实现原则
1. **简单至上**：只修改必要的代码，不做过度设计
2. **复用优先**：共用页面和组件，通过角色判断动态调整
3. **数据驱动**：通过配置对象驱动UI渲染，减少硬编码
4. **向后兼容**：确保现有物业员工功能不受影响
5. **找到根因**：如有bug，必须找到根本原因并修复，不做临时方案

---

## 审查项（Review）
*待实现后填写*
