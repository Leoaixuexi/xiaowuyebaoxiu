# 安全修复与上线准备

## 已完成的安全修复 (2025-12-19)

### 1. 禁止"清库按钮"裸奔
- [x] `cloudfunctions/initDatabase/index.js:362` - 必须传 adminToken 且匹配 `process.env.DB_ADMIN_TOKEN`

### 2. 禁止伪造身份
- [x] `cloudfunctions/userAuth/index.js:211` - test_openid 仅在 `ALLOW_TEST_OPENID=true` 且 ADMIN_TOKEN 校验通过时可用
- [x] 不再信任 current_user_id,密码登录时把当前 openid 绑定到账号

### 3. 禁止越权查工单/测试后门
- [x] `cloudfunctions/workOrderManager/index.js:791` - list 不再接受客户端 user_id
- [x] `cloudfunctions/workOrderManager/index.js:825` - getById 加了访问控制
- [x] 移除了 addTestPhotos 测试后门

### 4. 统一工单状态
- [x] `cloudfunctions/workOrderManager/index.js:15` - 归一化后对外永远返回英文状态,老数据照样能查/能统计

### 5. 修复统计函数读错集合/字段
- [x] `cloudfunctions/getAnalyticsOverview/index.js:18` - 改为读 work_orders
- [x] 把"类别统计"改为 order_category

### 6. 小程序端去后端依赖 + 修复点不动工单
- [x] `miniprogram/pages/maintenance/pending/index.js:57` - 改为走云函数
- [x] `miniprogram/pages/property/review/index.js:1` - 改为走云函数
- [x] `miniprogram/components/work-order-card/index.js:197` - 修复工单卡片点击拿不到 id

---

## 上线前必做事项

### 1. 云函数环境变量配置
- [ ] 设置 `DB_ADMIN_TOKEN`(给 initDatabase 用)
- [ ] 设置 `ADMIN_TOKEN`
- [ ] 确保 `ALLOW_TEST_OPENID` 不为 true(生产环境)

### 2. 重新部署云函数
- [ ] userAuth
- [ ] workOrderManager
- [ ] getAnalyticsOverview
- [ ] 其他 getAnalytics* 相关云函数
- [ ] initDatabase

### 3. 数据清理
- [ ] 生产环境删除/重置默认测试账号密码(即使前端隐藏了入口,账号本身还在库里就是雷)

---

## 物业员工复核弹窗简化 - 已完成

### 需求描述
物业员工在工单"待复核"状态时的交互优化：

**1. 已复核弹窗简化**
- 点击"已复核"按钮后弹出确认弹窗
- 只显示提示文字："是否确认该故障已修复？"
- 删除"审核结果"显示区域
- 删除"审核意见"输入框
- 只保留"确认"和"取消"两个按钮

**2. 需返工弹窗简化**
- 点击三个点菜单中的"需重修"按钮后弹出表单弹窗
- 只显示"需返工原因"输入框(改为非必填)
- 删除"审核结果"显示区域
- 删除"拒绝后工单将退回给维修人员返工"提示文字
- 保留"确认"和"取消"两个按钮

**3. 需返工原因显示到进度中**
- 如果需返工原因有内容,将其显示在工单进度时间线的"需返工"状态卡片内
- 需返工原因通过 status_history.notes 保存到状态历史

### 待办事项

#### 1. 修改 handleApprove 方法 - 已复核直接弹出确认框
- [x] 修改 `index.js:1018` handleApprove() 方法
- [x] 使用 wx.showModal 替代自定义表单
- [x] 确认对话框内容:"是否确认该故障已修复?"
- [x] 确认后调用 workOrderService.reviewWorkOrder()

#### 2. 修改审核表单 WXML - 简化需返工弹窗
- [x] 修改弹窗标题为"需返工"
- [x] 删除"审核结果"显示区域 (wxml:239-245)
- [x] 修改"审核意见"标签为"需返工原因"
- [x] 删除 required 类名,改为非必填
- [x] 删除"拒绝后工单将退回..."警告提示 (wxml:263-267)
- [x] 修改确认按钮文字为"确认"

#### 3. 修改 submitReview 方法 - 移除必填验证
- [x] 修改 `index.js:1211` submitReview() 方法
- [x] 移除需返工时审核意见的必填验证
- [x] 允许空内容提交

#### 4. 验证需返工原因显示到进度中
- [x] 验证 workOrderService.reviewWorkOrder() 将 reviewNotes 保存到 status_history.notes
- [x] 验证工单进度时间线正确显示 status_history.notes 内容
- [x] 优化 processWorkOrder 方法,将 '无' 改为空字符串

### 修改的文件
- `miniprogram/pages/work-order-detail/index.js`
- `miniprogram/pages/work-order-detail/index.wxml`

### Review 总结 (2025-12-23)

**修改的文件:**
1. `miniprogram/pages/work-order-detail/index.js`
2. `miniprogram/pages/work-order-detail/index.wxml`

**核心改动:**

1. **已复核交互简化**(index.js 第1018-1083行)
   - 从自定义表单改为系统 wx.showModal 对话框
   - 标题:"确认复核"
   - 内容:"是否确认该故障已修复？"
   - 确认后直接调用 reviewWorkOrder(),传入空字符串作为审核意见
   - 代码从80+行简化为60+行

2. **需返工弹窗简化**(index.wxml 第230-265行)
   - 标题固定为"需返工"
   - 删除"审核结果"显示区域(原242-245行)
   - 标签从"审核意见"改为"需返工原因"
   - placeholder 改为"请填写需返工原因(选填)"
   - 删除"拒绝后工单将退回..."警告提示(原263-267行)
   - 确认按钮文字改为"确认"

3. **submitReview 方法简化**(index.js 第1211-1282行)
   - 删除必填验证逻辑(原1153-1160行)
   - 固定 reviewDecision 为 'Needs Rework'
   - 允许空字符串的需返工原因
   - 确认对话框标题:"确认需返工"
   - 成功提示改为"已退回返工"

4. **时间线显示优化**(index.js 第440行)
   - 将 `item.notes || '无'` 改为 `item.notes || ''`
   - 避免在没有描述时显示"无"字样
   - 如果填写了需返工原因,会自动显示在"需返工"状态卡片

**交互流程:**

**已复核流程:**
```
点击"已复核"按钮
  ↓
wx.showModal 系统对话框
  ↓
用户点击"确认"
  ↓
调用 reviewWorkOrder(orderId, 'Completed', '')
  ↓
显示"复核成功" Toast
  ↓
2秒后返回上一页
```

**需返工流程:**
```
点击三个点菜单 → "需重修"
  ↓
显示自定义表单弹窗(只有"需返工原因"输入框)
  ↓
用户填写原因(可选)并点击"确认"
  ↓
wx.showModal 二次确认:"确认将此工单退回给维修人员返工吗？"
  ↓
用户点击"确认"
  ↓
调用 reviewWorkOrder(orderId, 'Needs Rework', reviewNotes)
  ↓
显示"已退回返工" Toast
  ↓
2秒后返回上一页
```

**技术实现:**
- 云函数 workOrderManager 的 reviewOrder 函数已支持将 reviewNotes 保存到 status_history.notes(第886行)
- 前端 processWorkOrder 方法从 status_history.notes 读取并生成时间线数据(第440行)
- timeline-item 组件自动显示 description 字段

**影响范围:**
- 只修改2个文件,共3处逻辑
- 代码量减少约50行
- 交互流程更简洁,用户体验更好
- 完全向后兼容,不影响已有工单数据

---

## 优先级选项简化 (普通/紧急) - 已完成

### 需求
- 将优先级从4个选项(低/普通/高/紧急)改为2个选项(普通/紧急)
- 卡片上显示两种标签:普通(绿色)、紧急(红色)

### 待办事项

#### 1. 前端常量修改
- [x] `miniprogram/utils/constants.js` - 修改 PRIORITIES 数组,只保留 Normal 和 Emergency
- [x] 修改 PRIORITY_DISPLAY_NAMES,只保留普通和紧急
- [x] 修改 PRIORITY_COLORS,只保留普通和紧急(普通改为绿色)

#### 2. 表单页面修改
- [x] `miniprogram/pages/property/submit/index.js` - priorityOptions 改为2个选项
- [x] `miniprogram/pages/property/submit/index.wxss` - 按钮间距从 12rpx 增大到 32rpx
- [x] `miniprogram/pages/work-order-edit/index.js` - priorityOptions 改为2个选项

#### 3. 卡片/列表显示修改
- [x] `miniprogram/pages/index/index.wxml` - 修改为只显示普通(绿色)和紧急(红色)两种标签
- [x] `miniprogram/pages/index/index.wxss` - 普通标签颜色改为绿色,删除高/低样式

#### 4. 后端常量修改(保持兼容)
- [x] `backend/src/utils/constants.js` - 修改 PRIORITIES 数组

#### 5. 数据库兼容性(暂不修改)
- 数据库 ENUM 保留原有值,以兼容历史数据
- 旧的 Low/High 数据在列表显示时会显示为普通(绿色标签)

### Review 总结 (2025-12-21)

**修改的文件:**
1. `miniprogram/utils/constants.js` - 前端常量,PRIORITIES/PRIORITY_DISPLAY_NAMES/PRIORITY_COLORS 都改为只有 Normal 和 Emergency
2. `miniprogram/pages/property/submit/index.js` - 提交表单优先级选项改为普通(绿色)和紧急(红色)
3. `miniprogram/pages/property/submit/index.wxss` - 优先级按钮间距增大
4. `miniprogram/pages/work-order-edit/index.js` - 编辑表单优先级选项同步修改
5. `miniprogram/pages/index/index.wxml` - 列表标签只显示普通和紧急
6. `miniprogram/pages/index/index.wxss` - 普通标签改为绿色(#22C55E),删除高/低样式
7. `backend/src/utils/constants.js` - 后端常量同步修改

**兼容性说明:**
- 数据库 ENUM 未修改,历史数据(Low/High)仍可存储
- 前端显示时,非 Emergency 的都显示为"普通"绿色标签

---

## 经理数据页面日期筛选增加"全部"选项 - 已完成

### 需求
- KPI统计和可视化图表模块日期筛选增加"全部"选项
- 页面刷新或页面切换时默认选中"全部"选项

### 修改的文件

1. **`miniprogram/utils/dateUtils.js`**
   - 添加 `'all'` case,返回 null 日期范围

2. **`miniprogram/pages/data/index.js`**
   - `timeFilter` 默认值改为 `'all'`
   - `initManagerView()` 初始化为"全部"
   - `switchTab()` 切换时重置为"全部"
   - `onTimeFilterChange()` 添加 `'all'` 处理

3. **`miniprogram/pages/data/index.wxml`**
   - Tab1 和 Tab2 的日期筛选器都添加了"全部"按钮

4. **`miniprogram/pages/admin-manager/analytics/index.js`**
   - 添加 `timeFilter` 状态
   - 添加 `dateUtils` 引用
   - `initializeDateRange()` 默认为"全部"
   - 添加 `onTimeFilterChange()` 方法
   - 添加自定义日期相关方法

5. **`miniprogram/pages/admin-manager/analytics/index.wxml`**
   - 替换原有日期选择器为快捷按钮组
   - 添加自定义日期选择弹窗

6. **`miniprogram/pages/admin-manager/analytics/index.wxss`**
   - 添加 `.manager-date-filter` 样式
   - 添加 `.date-chip` 按钮样式
   - 添加日期选择弹窗样式

### Review 总结 (2025-12-21)

两个经理数据页面现在都有统一的日期筛选按钮:**全部 | 昨日 | 今日 | 本周 | 本月 | 📅自定义**

默认选中"全部",查询所有历史数据。

---

## 维修员确认维修弹窗简化 - 已完成

### 需求
1. 维修员在"维修中"状态点击"确认维修"后的底部弹窗:
   - 删除"维修结果"选择器
   - 删除"维修照片"上传区域
   - 只保留"完成说明"字段,改名为"完成描述"
   - "完成描述"改为非必填字段
2. 如果完成描述有内容,显示在工单进度时间线的"已修复"状态卡片内

### 修改的文件

1. **`miniprogram/pages/work-order-detail/index.wxml`** (第194-229行)
   - 删除维修结果 picker
   - 删除维修照片上传区域
   - 将"完成说明"改为"完成描述"
   - 简化弹窗为只有一个文本输入框

2. **`miniprogram/pages/work-order-detail/index.js`**
   - Data 属性从6个简化为3个(showRepairForm, completionNotes, submittingRepair)
   - 删除方法:onRepairStatusChange, chooseRepairPhotos, removeRepairPhoto
   - 简化 submitRepairCompletion() - 移除必填验证,固定状态为 'Repaired'
   - 简化 cancelRepairForm()

3. **`miniprogram/services/workOrder.js`** (第282-322行)
   - completeRepair 函数从4个参数简化为2个:orderId, completionNotes

4. **`cloudfunctions/workOrderManager/index.js`**
   - completeRepair 函数:
     - 参数从 (openid, orderId, status, completionNotes, repairPhotos) 简化为 (openid, orderId, completionNotes)
     - 固定状态为 'Repaired'
     - 移除 repair_photos 处理
     - completion_notes 通过 addStatusHistory 保存到 status_history.notes
   - 调用处(case 'completeRepair')同步更新参数

### Review 总结 (2025-12-21)

**核心改动:**
- 维修员确认维修弹窗从复杂的三字段表单(维修结果+完成说明+维修照片)简化为单一可选字段(完成描述)
- 点击确认后状态固定变为"已修复",无需选择

**时间线显示说明:**
- 完成描述通过 addStatusHistory 保存到 status_history 的 notes 字段
- timeline-item 组件已有显示 description 的逻辑(第19-21行),无需额外修改
- 当完成描述不为空时,会自动显示在"已修复"状态卡片内

---

## 工单状态筛选默认值优化 - 已完成

### 需求描述
维修员和物业员工账号工作台页面工单状态筛选优化:
1. 去掉"全部"选项
2. 维修员默认选中"待接单"状态
3. 物业员工默认选中"已提报"状态
4. 切换页面或登录账号进入页面时应用默认状态

### 当前实现
- 维修员状态按钮:全部 | 待接单 | 维修中 | 已修复 | 需返工 | 已完成
- 物业员工状态按钮:全部 | 已提报 | 维修中 | 待复核 | 已完成
- 默认值统一为 `'all'`(第115行)

### 解决方案
1. 移除维修员和物业员工状态按钮数组中的"全部"选项
2. 修改默认值逻辑:
   - 维修员:默认为 `'pending_accept'`(待接单)
   - 物业员工:默认为 `'reported'`(已提报)
3. 保留重置状态逻辑,从子页面返回时保持原状态

### 待办事项
- [x] 移除维修员状态按钮数组中的"全部"选项(第180-186行)
- [x] 移除物业员工状态按钮数组中的"全部"选项(第171-176行)
- [x] 修改默认状态逻辑,根据角色设置不同默认值(第114-120行)
- [x] 验证页面初始加载时显示正确的默认状态
- [x] 验证tab切换时重置为正确的默认状态
- [x] 验证从工单详情页返回时保持选中的状态

### 修改的文件
- `gongdanbaoxiu/miniprogram/pages/index/index.js`

### 注意事项
- 物业经理保留"全部"选项,不做修改
- 从子页面返回时需保持用户选择的状态(第122行逻辑保留)
- 确保 scrollIntoView 与默认状态匹配(第135行)

### Review 总结 (2025-12-22)

**修改的文件:**
`gongdanbaoxiu/miniprogram/pages/index/index.js`

**核心改动:**
1. **物业员工状态按钮**(第171-176行):移除"全部"选项,保留4个状态(已提报、维修中、待复核、已完成)
2. **维修员状态按钮**(第180-186行):移除"全部"选项,保留5个状态(待接单、维修中、已修复、需返工、已完成)
3. **默认状态逻辑**(第114-120行):根据角色设置不同默认值
   - 维修员:`'pending_accept'`(待接单)
   - 物业员工:`'reported'`(已提报)
   - 物业经理:`'all'`(全部,保持不变)

**逻辑保持不变:**
- 从子页面返回时保持用户选择的状态(第122行 `shouldResetToDefault` 逻辑)
- 页面初始加载或tab切换时重置为角色对应的默认状态
- 滚动视图自动定位到选中的状态按钮(第135行 `scrollIntoView`)

**影响范围:**
只修改了3处代码,影响最小,保持简洁原则。

---

## 物业员工工作台增加"需返工"状态及详情页功能对齐 - 已完成

### 需求描述
1. 物业员工账号工作台页面增加"需返工"状态筛选选项
2. 物业员工点击"需返工"状态工单进入详情页后,底部按钮和功能与物业经理保持一致

### 当前实现分析
**工作台状态按钮(index.js 第176-181行):**
- 物业员工:已提报 | 维修中 | 待复核 | 已完成
- 物业经理:全部 | 已提报 | 维修中 | 已修复 | 待复核 | 需返工 | 已完成

**工单详情页按钮权限(work-order-detail/index.js 第279-289行):**
- 需返工状态下:
  - 维修员:显示"确认修复"按钮
  - 物业经理/物业员工:显示"催维修"按钮 + 删除菜单

### 解决方案
1. 在物业员工状态按钮数组中添加"需返工"选项
2. 工单详情页"需返工"状态下的按钮逻辑已经支持物业员工(第285行),无需修改

### 待办事项
- [x] 在物业员工状态按钮数组中添加"需返工"选项(第177-182行)
- [x] 验证物业员工可以看到"需返工"状态工单列表
- [x] 验证物业员工点击"需返工"工单后详情页显示正确的按钮(催维修+删除)

### 修改的文件
- `gongdanbaoxiu/miniprogram/pages/index/index.js`

### 注意事项
- 工单详情页逻辑已支持物业员工在"需返工"状态下的权限(work-order-detail/index.js:285行:`isPropertyManager || isPropertyStaff`)
- 只需添加工作台筛选选项即可

### Review 总结 (2025-12-22)

**修改的文件:**
`gongdanbaoxiu/miniprogram/pages/index/index.js`

**核心改动:**
- **物业员工状态按钮**(第176-182行):在数组中插入"需返工"选项
  - 修改前:已提报 | 维修中 | 待复核 | 已完成(4个选项)
  - 修改后:已提报 | 维修中 | 待复核 | 需返工 | 已完成(5个选项)
  - 配置:`{ key: 'rework', label: '需返工', status: 'Needs Rework' }`

**详情页功能验证:**
- 工单详情页代码已支持物业员工在"需返工"状态下的完整权限
- "需返工"状态下物业员工显示:
  - 三个点菜单(含删除选项)
  - "催维修"按钮
- 与物业经理功能完全一致(work-order-detail/index.js:285)

**影响范围:**
只增加1行代码,极简修改,无需改动详情页逻辑。

---

## 更新图标字体并替换三个点菜单删除图标 - 已完成

### 需求描述
1. 更新iconfont图标字体文件到最新版本
2. 将工单详情页三个点菜单中的删除图标从 emoji (🗑️) 替换为新的 iconfont 图标 (icon-shanchu1)

### 当前实现
- **图标字体版本**:font_5090968_asys14b3mm6 (旧版本)
- **删除图标**:使用 emoji 🗑️ (work-order-detail/index.wxml:296)

### 新版本信息
- **图标字体版本**:font_5090968_i5hfy9pvg8 (新版本)
- **新增图标**:
  - `icon-shanchu1` (\e613) - 新的删除图标
  - `icon-diyiming` (\e69d) - 第一名图标
  - `icon-dierming` (\e61c) - 第二名图标
  - `icon-disanming` (\e608) - 第三名图标

### 解决方案
1. 更新 `miniprogram/styles/iconfont.wxss` 中的字体源链接
2. 添加新增图标的 CSS 类定义
3. 替换工单详情页三个点菜单中的删除图标为 `icon-shanchu1`

### 待办事项
- [x] 更新iconfont.wxss字体源URL(从 asys14b3mm6 → i5hfy9pvg8)
- [x] 添加 icon-shanchu1、icon-diyiming、icon-dierming、icon-disanming 图标定义
- [x] 替换工单详情页删除图标:从 emoji 改为 iconfont
- [x] 验证新图标显示正确

### 修改的文件
- `miniprogram/styles/iconfont.wxss`
- `miniprogram/pages/work-order-detail/index.wxml`

### Review 总结 (2025-12-22)

**修改的文件:**
1. `miniprogram/styles/iconfont.wxss`
2. `miniprogram/pages/work-order-detail/index.wxml`

**核心改动:**

1. **更新图标字体源**(iconfont.wxss 第9-11行)
   - 字体版本:font_5090968_asys14b3mm6 → font_5090968_i5hfy9pvg8
   - 时间戳:1766156093735 → 1766413188191

2. **添加新增图标定义**(iconfont.wxss)
   - 第29-31行:添加 `icon-shanchu1` (\e613) - 新删除图标
   - 第130-142行:添加排名图标
     - `icon-diyiming` (\e69d) - 第一名
     - `icon-dierming` (\e61c) - 第二名
     - `icon-disanming` (\e608) - 第三名
   - 保留旧版本图标类(标注为"旧版本")以保持向后兼容

3. **替换删除图标**(work-order-detail/index.wxml 第276-278行)
   - 修改前:`<view class="more-action-icon delete-icon">🗑️</view>`
   - 修改后:
     ```xml
     <view class="more-action-icon delete-icon">
       <text class="iconfont icon-shanchu1"></text>
     </view>
     ```

**影响范围:**
- 所有使用新图标字体的页面都会自动更新
- 三个点菜单的删除图标从 emoji 变为矢量图标,显示更清晰
- 新增的排名图标可用于未来的数据页面优化

**兼容性说明:**
- 保留了旧版本图标类定义,不会影响现有代码
- 新增图标可立即使用,无需修改其他代码

---

## 消息页面待办事项通知系统 - 已完成

### 需求
在消息页面的待办事项模块实现工单状态变更的自动通知功能。

### 通知触发场景

**场景1:新工单提交(责任方=物业公司)**
- 触发条件:物业员工提交新工单,且责任方选择为"物业公司"
- 接收者:所有部门字段为"物业公司"的账号
- 消息格式:`工单编号:请确认并安排维修`,内容为`楼层 + 位置 + 故障描述 + "请确认并安排维修"`

**场景2:工单维修完成**
- 触发条件:工单状态变更为"已修复"
- 接收者:提交工单的物业员工
- 消息格式:`工单编号`,内容为`楼层 + 位置 + 故障描述 + "维修完成,请到场复核"`

**场景3:复核通过**
- 触发条件:物业员工点击"已复核"按钮
- 接收者:所有部门字段为"物业公司"的账号
- 消息格式:`工单编号`,内容为`楼层 + 位置 + 故障描述 + "复核通过,辛苦了!"`

**场景4:需要返工**
- 触发条件:物业员工点击"需返工"按钮
- 接收者:所有部门字段为"物业公司"的账号
- 消息格式:`工单编号`,内容为`楼层 + 位置 + 故障描述 + "现场复核未通过,请返工,返工说明:" + 返工原因内容`

### 待办事项
- [x] 修改 workOrderManager 云函数 - createNotification 函数支持批量发送
- [x] 更新场景1:新工单创建时的通知逻辑
- [x] 更新场景2:维修完成时的通知逻辑
- [x] 更新场景3+4:复核通过/返工时的通知逻辑
- [x] 修改消息主页面接入真实数据
- [x] 修改消息列表页接入真实数据
- [x] 实现消息点击跳转工单详情

### 修改的文件

#### 1. 云函数修改
**`cloudfunctions/workOrderManager/index.js`**
- 添加 `createBatchNotifications()` 函数:支持批量发送通知给多个用户
- 添加 `formatNotificationMessage()` 辅助函数:格式化通知消息为"楼层 位置 描述 后缀"格式
- 修改 `createWorkOrder()` 函数(场景1):当责任方为"物业公司"时,获取所有物业公司部门用户并批量发送通知
- 修改 `completeRepair()` 函数(场景2):发送格式化通知给工单提交者
- 修改 `reviewOrder()` 函数(场景3+4):根据复核状态(通过/返工)发送相应通知给所有物业公司部门用户

#### 2. 前端修改
**`miniprogram/pages/notifications/index.js`**
- 引入 `notificationService` 服务
- 修改 `loadMessageData()` 函数:调用真实API获取待办工单通知(未读)
- 添加 `formatTime()` 函数:格式化时间为相对时间(刚刚/N分钟前/N小时前/N天前)
- 筛选工单相关通知并更新UI显示

**`miniprogram/pages/message-list/index.js`**
- 引入 `notificationService` 服务
- 修改 `loadMessages()` 函数:对于 workorder 模块,调用真实API获取通知列表
- 修改 `handleMessageTap()` 函数:
  - 点击通知时调用真实API标记为已读
  - 从通知数据中提取 orderId 跳转到工单详情页
- 修改 `handleMarkAllRead()` 函数:对于工单通知,调用真实API标记所有为已读

### Review 总结 (2025-12-22)

**核心改动:**
1. 云函数实现了完整的批量通知系统,支持4种工单状态变更场景
2. 通知消息统一格式为"楼层 + 位置 + 描述 + 业务说明"
3. 前端消息页面和列表页完全接入真实通知数据
4. 实现了消息已读标记和点击跳转工单详情功能

**通知流程:**
- 工单状态变更 → workOrderManager 云函数触发
- 根据场景筛选接收者(提交者 or 物业公司部门用户)
- 批量保存通知到 notifications 集合
- 前端调用 notificationService 获取并显示通知
- 用户点击通知 → 标记已读 + 跳转工单详情页

**数据流向:**
```
工单状态变更
  ↓
createBatchNotifications() / createNotification()
  ↓
保存到微信云数据库 notifications 集合
  ↓
notificationService.getUserNotifications()
  ↓
消息页面/列表页显示
  ↓
点击 → 标记已读 + 跳转工单详情
```

---

## 历史开发记录

### 管理员后台功能 (已完成)
- 角色与权限管理页面
- 审计日志页面
- 系统配置页面

详见 git 历史记录。
