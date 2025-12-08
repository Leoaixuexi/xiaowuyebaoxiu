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
- 待补充...
