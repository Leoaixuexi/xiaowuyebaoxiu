# 维修员功能开发 - 详细实施计划

## 当前状态
- ✅ 物业员工功能完整版本已保存（v1.0-property-staff-complete）
- ✅ 代码已上传到 GitHub
- 🎯 准备开始维修员功能开发

---

## 实施步骤

### 阶段 1：工作台页面改造 (pages/index/index)

#### Step 1.1：添加角色判断逻辑
- [ ] 在 `onShow` 方法中调用 `auth.getCurrentUser()` 获取用户信息
- [ ] 将 `userRole`（role_id）、`userDepartment`（department）、`userId` 存储到 data
- [ ] 添加 `isPropertyStaff` 和 `isMaintenanceWorker` 布尔标识

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/index/index.js`

**代码示例：**
```javascript
async onShow() {
  // 获取用户信息
  const userInfo = await auth.getCurrentUser();
  const isPropertyStaff = userInfo.role_id === 4;
  const isMaintenanceWorker = userInfo.role_id === 3;

  this.setData({
    userRole: userInfo.role_id,
    userDepartment: userInfo.department,
    userId: userInfo.id,
    isPropertyStaff,
    isMaintenanceWorker
  });

  // ... 其余逻辑
}
```

---

#### Step 1.2：动态生成状态筛选按钮
- [ ] 在 data 中创建 `statusButtonsConfig` 配置对象
- [ ] 包含两套配置：物业员工和维修员
- [ ] 在 `onShow` 中根据角色动态设置 `statusButtons` 数组
- [ ] 修改 WXML，使用 `wx:for` 遍历 `statusButtons` 渲染

**状态按钮配置：**
- 物业员工：[已提报, 维修中, 待复核, 已完成]
- 维修员：[待接单, 维修中, 已修复, 需重修, 已完成]

**状态映射：**
```javascript
// 维修员状态到实际状态的映射
const maintenanceStatusMap = {
  'pending_accept': 'Pending Repair',   // 待接单
  'maintenance': 'In Progress',         // 维修中
  'repaired': 'Repaired',               // 已修复
  'rework': 'Needs Rework',             // 需重修
  'completed': 'Completed'              // 已完成
};
```

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/index/index.js`
- `gongdanbaoxiu/miniprogram/pages/index/index.wxml`

---

#### Step 1.3：调整工单数据过滤逻辑
- [ ] 修改 `loadWorkOrders` 方法，在获取全部工单后，根据角色过滤
- [ ] 物业员工：只显示 `submitter.user_id === 当前用户ID`
- [ ] 维修员：只显示 `responsible_party === 用户部门`
- [ ] 修改 `filterByStatus` 方法，维修员的状态映射

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/index/index.js`

---

#### Step 1.4：隐藏维修员的新工单提交按钮
- [ ] 在 WXML 的浮动按钮（FAB）上添加 `wx:if="{{isPropertyStaff}}"`
- [ ] 确保维修员看不到"+"按钮

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/index/index.wxml`

---

### 阶段 2：工单详情页改造 (pages/work-order-detail/index)

#### Step 2.1：添加"接单"按钮显示逻辑
- [ ] 在 `loadWorkOrder` 方法的权限判断部分添加 `canAcceptOrder`
- [ ] 条件：维修员 && 工单状态为 Pending Repair && 责任方=用户部门

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.js`

---

#### Step 2.2：实现"接单"功能
- [ ] 添加 `handleAcceptOrder` 方法
- [ ] 调用 `workOrderService.updateWorkOrderStatus(orderId, 'In Progress', '维修员接单开始维修')`
- [ ] 成功后显示提示并刷新页面

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.js`

---

#### Step 2.3：更新 WXML 模板
- [ ] 在按钮区域添加"接单"按钮，条件显示 `wx:if="{{canAcceptOrder}}"`
- [ ] 使用绿色主题样式

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.wxml`
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.wxss`

---

### 阶段 3：数据页面改造 (pages/data/index)

#### Step 3.1：动态生成统计卡片
- [ ] 创建 `statsConfig` 配置对象，包含两套统计
- [ ] 在 `onShow` 中根据角色动态设置 `stats`

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/data/index.js`

---

#### Step 3.2：实现统计数据获取
- [ ] 添加 `loadStatistics` 方法
- [ ] 根据角色过滤工单后进行统计
- [ ] 物业员工：统计 `submitter.user_id === 当前用户ID`
- [ ] 维修员：统计 `responsible_party === 用户部门`

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/data/index.js`

---

#### Step 3.3：隐藏维修员的排名模块
- [ ] 在 WXML 的排名模块上添加 `wx:if="{{isPropertyStaff}}"`

**预期修改文件：**
- `gongdanbaoxiu/miniprogram/pages/data/index.wxml`

---

### 阶段 4：消息页面改造（可选，暂时跳过）

> 注：消息推送逻辑需要后端/云函数支持，暂时先实现前端基础功能

---

### 阶段 5：测试和验证

#### 测试清单
- [ ] 使用物业员工账号测试所有功能
  - [ ] 工作台只显示自己的工单
  - [ ] 状态筛选正常
  - [ ] 提交新工单正常
  - [ ] 数据统计和排名正常
- [ ] 使用维修员账号测试所有功能
  - [ ] 工作台只显示责任方=自己部门的工单
  - [ ] 状态筛选（待接单、维修中、已修复等）正常
  - [ ] 接单功能正常
  - [ ] 数据统计正常（无排名）
  - [ ] 无新工单提交按钮

---

## 关键技术点

### 1. 用户信息结构
```javascript
{
  id: 1,
  username: "zhangsan",
  name: "张三",
  role_id: 3,          // 3=维修员, 4=物业员工
  department: "维修部",
  phone: "13800138000",
  ...
}
```

### 2. 工单状态枚举
```javascript
'Pending Repair'   // 已提报/待接单
'In Progress'      // 维修中
'Repaired'         // 已修复
'Needs Rework'     // 需返工
'Completed'        // 已完成
```

### 3. 数据过滤关键字段
- `submitter.user_id` - 工单提报人ID
- `responsible_party` - 责任方（部门）
- `status` - 工单状态

---

## 实现原则（再次强调）
1. **简单至上**：只改必要的代码
2. **复用优先**：共用页面和组件
3. **数据驱动**：配置对象驱动UI
4. **向后兼容**：不影响现有功能
5. **找到根因**：不做临时修复

---

## 审查检查点
*每完成一个阶段后填写*

### 阶段 1 完成检查
- [ ] 代码是否简洁清晰
- [ ] 是否影响物业员工功能
- [ ] 是否有硬编码
- [ ] 是否需要重构

### 阶段 2 完成检查
- [ ] 接单按钮是否正确显示
- [ ] 接单功能是否正常
- [ ] 状态变更是否记录

### 阶段 3 完成检查
- [ ] 统计数据是否准确
- [ ] 排名模块是否正确隐藏
- [ ] 性能是否正常

---

## 变更日志
- 2025-12-08：创建详细实施计划
- 2025-12-08：完成阶段 1 - 工作台页面改造
- 2025-12-08：完成阶段 2 - 工单详情页改造
- 2025-12-08：完成阶段 3 - 数据页面改造
- 2025-12-08：修复用户反馈问题（提交按钮、待接单统计）
- 2025-12-08：修复数据页面统计 - 用今日维修替换维修中

---

## 开发完成审查报告

### 一、总体完成情况
✅ **所有核心功能已完成并提交到 GitHub**

**Git 提交记录：**
1. `feat: 实现维修员角色功能` - 核心功能实现
2. `fix: 修复维修员界面问题` - 用户反馈修复
3. `fix: 维修员数据页面统计 - 用今日维修替换维修中` - 最新优化

---

### 二、已完成功能列表

#### 1. 工作台页面 (pages/index/index)
**已实现：**
- ✅ 角色判断逻辑（维修员 role_id=3，物业员工 role_id=4）
- ✅ 动态状态筛选按钮
  - 物业员工：[已提报, 维修中, 待复核, 已完成]
  - 维修员：[待接单, 维修中, 已修复, 需重修, 已完成]
- ✅ 基于角色的工单过滤
  - 物业员工：只看自己提报的工单 (`submitter.user_id`)
  - 维修员：只看责任方为自己部门的工单 (`responsible_party`)
- ✅ 隐藏维修员的新工单提交按钮（FAB 和空状态按钮）
- ✅ 状态文本映射（同一状态对不同角色显示不同文本）

**修改文件：**
- `gongdanbaoxiu/miniprogram/pages/index/index.js` - 核心逻辑
- `gongdanbaoxiu/miniprogram/pages/index/index.wxml` - 模板调整
- `gongdanbaoxiu/miniprogram/pages/index/index.wxss` - 样式补充

---

#### 2. 工单详情页 (pages/work-order-detail/index)
**已实现：**
- ✅ "接单"按钮显示逻辑
  - 仅维修员可见
  - 仅当工单状态为"待接单"（Pending Repair）
  - 仅当责任方为当前用户部门
- ✅ "接单"功能实现
  - 调用 `workOrderService.updateWorkOrderStatus`
  - 状态从 Pending Repair → In Progress
  - 添加操作日志："维修员接单开始维修"
- ✅ 接单按钮样式（蓝色渐变）

**修改文件：**
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.js` - 接单逻辑
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.wxml` - 按钮添加
- `gongdanbaoxiu/miniprogram/pages/work-order-detail/index.wxss` - 按钮样式

---

#### 3. 数据页面 (pages/data/index)
**已实现：**
- ✅ 动态统计卡片配置
  - 物业员工：[今日提报, 维修中, 待复核, 已完成]
  - 维修员：[今日维修, 已修复, 需重修, 已完成]
- ✅ 实时统计数据计算
  - 基于角色过滤工单后统计
  - 今日提报：统计今天创建的工单
  - **今日维修**：统计今天接单或正在维修的工单（状态为 In Progress 且今天更新）
- ✅ 隐藏维修员的排名模块

**修改文件：**
- `gongdanbaoxiu/miniprogram/pages/data/index.js` - 统计逻辑
- `gongdanbaoxiu/miniprogram/pages/data/index.wxml` - 条件渲染

---

### 三、用户反馈问题修复

#### 问题 1：维修员看到"提交工单"按钮
**现象：** 维修员在空状态下看到"提交工单"按钮
**根因：** 空状态按钮未添加角色判断条件
**修复：** 在 WXML 中添加 `wx:if="{{isPropertyStaff}}"` 条件
**状态：** ✅ 已修复

#### 问题 2：数据页面显示"待接单"统计
**现象：** 维修员数据页面工单汇总显示"待接单"统计
**根因：** 维修员统计配置包含 `pending_accept` 项
**修复：** 从统计配置中移除该项
**状态：** ✅ 已修复

#### 问题 3：需要"今日维修"而非"维修中"
**现象：** 维修员数据页面显示"维修中"统计，用户需要"今日维修"
**根因：** 统计配置未区分"维修中"和"今日维修"
**修复：**
- 将 `in_progress` 替换为 `today_maintenance`
- 添加特殊计算逻辑：统计状态为 In Progress 且今天更新的工单
**状态：** ✅ 已修复

---

### 四、技术实现要点

#### 1. 共享页面架构
- 同一页面根据 `role_id` 动态展示不同 UI
- 优点：代码复用、维护简单、避免重复逻辑
- 实现方式：`wx:if` 条件渲染 + 动态数据配置

#### 2. 数据过滤策略
```javascript
// 物业员工：看自己的工单
myOrders = allOrders.filter(order =>
  order.submitter && order.submitter.user_id === userId
);

// 维修员：看责任方为自己部门的工单
myOrders = allOrders.filter(order =>
  order.responsible_party === userDepartment
);
```

#### 3. 状态映射
- 后端统一状态：Pending Repair, In Progress, Repaired, Needs Rework, Completed
- 前端根据角色显示不同文本：
  - Pending Repair → 物业员工："已提报" / 维修员："待接单"
  - Repaired → 物业员工："待复核" / 维修员："已修复"

#### 4. 统计算法
- 今日提报：`created_at >= today 00:00:00`
- 今日维修：`status === 'In Progress' && (updated_at >= today || created_at >= today)`

---

### 五、代码质量评估

#### 优点
✅ 代码简洁，只修改必要部分
✅ 使用配置对象驱动，易于维护
✅ 没有破坏现有物业员工功能
✅ 遵循 DRY 原则，避免重复代码
✅ 所有修复都找到根本原因，无临时方案

#### 改进建议
- 如需扩展更多角色，可考虑将角色配置抽取到独立文件
- 统计数据可考虑加缓存优化性能
- 消息推送功能需要后端/云函数支持（后续开发）

---

### 六、测试建议

#### 物业员工账号测试
- [ ] 工作台只显示自己提报的工单
- [ ] 状态筛选：已提报、维修中、待复核、已完成
- [ ] 可以提交新工单（FAB 按钮）
- [ ] 数据页面显示：今日提报、维修中、待复核、已完成
- [ ] 数据页面显示排名模块

#### 维修员账号测试
- [ ] 工作台只显示责任方为自己部门的工单
- [ ] 状态筛选：待接单、维修中、已修复、需重修、已完成
- [ ] 无提交工单按钮（FAB 和空状态按钮）
- [ ] 详情页可以接单（待接单状态）
- [ ] 数据页面显示：今日维修、已修复、需重修、已完成
- [ ] 数据页面不显示排名模块

---

### 七、总结

本次开发严格遵循"简单至上、复用优先"的原则，成功实现了维修员角色的所有核心功能。通过共享页面架构和动态配置的方式，在不增加系统复杂度的前提下，为两种角色提供了差异化的用户体验。

所有代码已提交到 GitHub，建议进行充分测试后再部署到生产环境。

**开发完成时间：** 2025-12-08
**GitHub 分支：** 001-work-order-system
**状态：** ✅ 开发完成，待测试验证
