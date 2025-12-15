# 物业经理数据页面 - 架构设计

## 1. 页面布局架构

**🔴 重要说明：**
1. **完全重构物业经理视图（role_id=2）** - 替换掉现有的所有内容，使用全新的Tab设计
2. **物业员工（role_id=4）和维修员（role_id=3）视图完全不变** - 一行代码都不动
3. **这不是添加功能，而是完全替换物业经理的数据页面**

**核心设计：使用Tab切换分离"数据统计"和"可视化图表"两个模块**

```
┌─────────────────────────────────────────────────────────────────┐
│                    数据页面 (Data Page)                          │
│                    /pages/data/index                             │
└─────────────────────────────────────────────────────────────────┘
                               │
                               │ 角色检测 (role_id)
                               │
                ┌──────────────┼──────────────┐
                │              │              │
         role_id=4      role_id=3      role_id=2
                │              │              │
                ▼              ▼              ▼
┌───────────────────┐  ┌───────────────────┐  ┌───────────────────────────────┐
│ ❌ 不修改          │  │ ❌ 不修改          │  │ 🔄 完全重构                    │
│ 物业员工视图       │  │ 维修员视图         │  │ 物业经理全局分析视图           │
│ (保持现状)        │  │ (保持现状)        │  │ (替换掉所有旧内容)            │
│                   │  │                   │  │                               │
│ - 今日提报        │  │ - 今日维修        │  │ ┌───────────────────────────┐ │
│ - 维修中          │  │ - 已修复          │  │ │ Tab切换栏                   │ │
│ - 待复核          │  │ - 需重修          │  │ │ ┌─────────┬─────────────┐ │ │
│ - 已完成          │  │ - 已完成          │  │ │ │数据统计  │可视化图表    │ │ │
│                   │  │                   │  │ │ │(active) │            │ │ │
│ (个人工单)        │  │ (分配给自己)      │  │ │ └─────────┴─────────────┘ │ │
└───────────────────┘  └───────────────────┘  │ └───────────────────────────┘ │
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │ 共享：日期范围选择器         │ │
                                              │ │   ┌─┬─┬──┬──┬────┐       │ │
                                              │ │   │昨│今│周│月│日历│       │ │
                                              │ │   └─┴─┴──┴──┴────┘       │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              ├─ Tab 1: 数据统计 ─────────────┤
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │ KPI统计卡片 (2x2网格)      │ │
                                              │ │ ┌─────────┬─────────┐     │ │
                                              │ │ │ 总数     │ 已完成   │     │ │
                                              │ │ │ 500     │ 380(76%)│     │ │
                                              │ │ ├─────────┼─────────┤     │ │
                                              │ │ │ 进行中   │ 平均时长 │     │ │
                                              │ │ │ 85      │ 12.5小时│     │ │
                                              │ │ └─────────┴─────────┘     │ │
                                              │ │   (淡入+上滑动画)          │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │ 员工已完成工单排名          │ │
                                              │ │ 1️⃣ 张三  45单            │ │
                                              │ │ 2️⃣ 李四  38单            │ │
                                              │ │ 3️⃣ 王五  32单            │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │ 责任方已修复工单排名        │ │
                                              │ │ 🏢 物业公司  280单(70%)   │ │
                                              │ │ 👤 业主      80单(20%)    │ │
                                              │ │ 🏭 第三方    40单(10%)    │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              ├─ Tab 2: 可视化图表 ───────────┤
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │  右上角：[服务简报] 按钮    │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │ 工单处理进度 (环形图)       │ │
                                              │ │        🍩                  │ │
                                              │ │   (彩色甜甜圈图)           │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              │ ┌───────────────────────────┐ │
                                              │ │ 工单趋势图 (折线图)         │ │
                                              │ │  📈 蓝线:已提报            │ │
                                              │ │     绿线:已完成            │ │
                                              │ └───────────────────────────┘ │
                                              │                               │
                                              │ ┌──────────┬──────────┐      │
                                              │ │故障类型   │责任方     │      │
                                              │ │分布饼图   │分布饼图   │      │
                                              │ └──────────┴──────────┘      │
                                              │                               │
                                              │ ┌──────────┬──────────┐      │
                                              │ │楼层分布   │位置分布   │      │
                                              │ │柱状图     │柱状图     │      │
                                              │ └──────────┴──────────┘      │
                                              └───────────────────────────────┘
```

## 2. 组件层次结构

**🔴 关键说明：物业经理视图（role_id=2）将被完全重构，旧内容全部删除，替换为新设计**

```
pages/data/index
│
├─ 角色检测模块 (Role Detection)
│  └─ onLoad() → 读取 userInfo.role_id
│
├─ ❌ 物业员工视图 (role_id = 4) [不修改 - 保持现状]
│  ├─ 个人统计卡片
│  └─ 个人工单列表
│
├─ ❌ 维修员视图 (role_id = 3) [不修改 - 保持现状]
│  ├─ 个人统计卡片
│  └─ 个人工单列表
│
└─ 🔄 物业经理视图 (role_id = 2) [完全重构 - 删除旧内容，全新实现]
   │
   │  【旧内容将被删除】
   │  ├─ ❌ 删除旧的统计卡片
   │  ├─ ❌ 删除旧的图表
   │  └─ ❌ 删除旧的任何其他组件
   │
   │  【新内容实现】
   │
   ├─ Tab切换控制器
   │  ├─ Tab 1: 数据统计 (activeTab === 'stats')
   │  └─ Tab 2: 可视化图表 (activeTab === 'charts')
   │
   ├─ 共享组件区域
   │  └─ 日期范围选择器组件
   │     ├─ 预设过滤器按钮组
   │     │  ├─ 昨天按钮 (昨)
   │     │  ├─ 今天按钮 (今) - 默认选中
   │     │  ├─ 本周按钮 (周)
   │     │  ├─ 本月按钮 (月)
   │     │  └─ 自定义按钮 (日历图标)
   │     └─ 自定义日期选择器 (wx:if="{{showCustomPicker}}")
   │        ├─ 开始日期选择 (picker mode="date")
   │        └─ 结束日期选择 (picker mode="date")
   │
   ├─ Tab 1 内容区：数据统计 (wx:if="{{activeTab === 'stats'}}")
   │  │
   │  ├─ KPI统计卡片网格 (2x2 Grid)
   │  │  ├─ 卡片1: 总数
   │  │  ├─ 卡片2: 已完成 (带完成率百分比)
   │  │  ├─ 卡片3: 进行中
   │  │  └─ 卡片4: 平均时长
   │  │  └─ 动画控制器
   │  │     └─ fadeInSlideUp (错开延迟: 0/100/200/300ms)
   │  │
   │  ├─ 员工已完成工单排名组件
   │  │  ├─ 排名列表 (list-view)
   │  │  │  ├─ 排名序号
   │  │  │  ├─ 员工姓名
   │  │  │  └─ 已完成数量
   │  │  └─ 空状态占位符
   │  │
   │  └─ 责任方已修复工单排名组件
   │     ├─ 排名列表 (list-view)
   │     │  ├─ 责任方名称
   │     │  ├─ 已修复数量
   │     │  └─ 占比百分比
   │     └─ 空状态占位符
   │
   └─ Tab 2 内容区：可视化图表 (wx:if="{{activeTab === 'charts'}}")
      │
      ├─ 顶部功能栏
      │  └─ 服务简报按钮 (右上角)
      │
      ├─ ECharts图表区域
      │  ├─ ec-canvas 组件 (x6)
      │  │  │
      │  │  ├─ 1. 工单处理进度环形图 (甜甜圈图)
      │  │  │  ├─ 类型: doughnut/ring chart
      │  │  │  ├─ 数据: 各状态工单占比
      │  │  │  └─ echarts.init() + option
      │  │  │
      │  │  ├─ 2. 工单趋势折线图
      │  │  │  ├─ 类型: line chart
      │  │  │  ├─ 双线对比: 已提报(蓝) vs 已完成(绿)
      │  │  │  └─ echarts.init() + option
      │  │  │
      │  │  ├─ 3. 故障类型饼图
      │  │  │  ├─ 类型: pie chart
      │  │  │  └─ echarts.init() + option
      │  │  │
      │  │  ├─ 4. 责任方饼图
      │  │  │  ├─ 类型: pie chart
      │  │  │  └─ echarts.init() + option
      │  │  │
      │  │  ├─ 5. 楼层柱状图
      │  │  │  ├─ 类型: bar chart
      │  │  │  └─ echarts.init() + option
      │  │  │
      │  │  └─ 6. 位置柱状图
      │  │     ├─ 类型: bar chart
      │  │     └─ echarts.init() + option
      │  │
      │  └─ 图表工具函数
      │     ├─ getRingChartOption()     [NEW - 环形图]
      │     ├─ getLineChartOption()
      │     ├─ getPieChartOption()
      │     └─ getBarChartOption()
      │
      └─ 空状态处理
         └─ 无数据占位符
```

## 3. 数据流架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        前端 (小程序)                              │
└─────────────────────────────────────────────────────────────────┘
                               │
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                    页面加载 & 角色检测                            │
│                                                                  │
│  onLoad() {                                                      │
│    const userInfo = wx.getStorageSync('userInfo');              │
│    if (userInfo.role_id === 2) {                                │
│      this.setData({                                             │
│        isManager: true,                                         │
│        activeTab: 'stats'       // 默认显示数据统计tab         │
│      });                                                         │
│      this.initDateRange('today');    // 默认今天                │
│      this.fetchAllData();            // 获取所有数据             │
│    }                                                             │
│  }                                                               │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ├─ isManager = true, activeTab = 'stats'
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                   并行数据获取 (Promise.all)                      │
│                                                                  │
│  fetchAllData() {                                                │
│    Promise.all([                                                 │
│      // Tab 1: 数据统计需要的数据                                 │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsOverview',                             │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getEmployeeRanking',                               │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getResponsiblePartyRanking',                       │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│                                                                  │
│      // Tab 2: 可视化图表需要的数据                               │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsByStatus',                             │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsTrends',                               │
│        data: { startDate, endDate, period: 'daily' }            │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsByCategory',                           │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsByResponsibleParty',                   │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsByFloor',                              │
│        data: { startDate, endDate }                              │
│      }),                                                          │
│      wx.cloud.callFunction({                                     │
│        name: 'getAnalyticsByLocation',                           │
│        data: { startDate, endDate }                              │
│      })                                                           │
│    ])                                                            │
│  }                                                               │
│                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │ 云函数调用     │  │ 云函数调用     │  │ 云函数调用     │   │
│  │      ↓         │  │      ↓         │  │      ↓         │   │
│  │ getAnalytics   │  │ getEmployee    │  │ getResponsible │   │
│  │   Overview     │  │  Ranking       │  │ PartyRanking   │   │
│  └────────────────┘  └────────────────┘  └────────────────┘   │
│                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │ 云函数调用     │  │ 云函数调用     │  │ 云函数调用     │   │
│  │      ↓         │  │      ↓         │  │      ↓         │   │
│  │ getAnalytics   │  │ getAnalytics   │  │ getAnalytics   │   │
│  │  ByStatus      │  │   Trends       │  │  ByCategory    │   │
│  └────────────────┘  └────────────────┘  └────────────────┘   │
│                                                                  │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐   │
│  │ 云函数调用     │  │ 云函数调用     │  │ 云函数调用     │   │
│  │      ↓         │  │      ↓         │  │      ↓         │   │
│  │ getAnalytics   │  │ getAnalytics   │  │ getAnalytics   │   │
│  │ ByResponsible  │  │   ByFloor      │  │  ByLocation    │   │
│  │     Party      │  └────────────────┘  └────────────────┘   │
│  └────────────────┘                                            │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                               ├─ 所有数据返回
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                        数据处理 & 渲染                            │
│                                                                  │
│  1. 更新状态 (setData)                                           │
│     ├─ Tab 1 数据:                                              │
│     │  ├─ totalOrders, completedOrders, inProgressOrders       │
│     │  ├─ avgCompletionTime (秒转小时)                          │
│     │  ├─ employeeRanking                                       │
│     │  └─ responsiblePartyRanking                               │
│     │                                                            │
│     └─ Tab 2 数据:                                              │
│        ├─ statusDistribution (环形图数据)                       │
│        ├─ trendData (折线图数据)                                │
│        ├─ faultTypeData (饼图数据)                              │
│        ├─ responsiblePartyData (饼图数据)                       │
│        ├─ floorData (柱状图数据)                                │
│        └─ locationData (柱状图数据)                             │
│                                                                  │
│  2. 根据当前activeTab渲染                                        │
│     ├─ if activeTab === 'stats':                                │
│     │  ├─ 渲染KPI卡片                                           │
│     │  ├─ 播放卡片动画 (initCardAnimations)                    │
│     │  └─ 渲染排名列表                                          │
│     │                                                            │
│     └─ if activeTab === 'charts':                               │
│        └─ 初始化所有图表 (initAllCharts)                        │
│           ├─ initStatusRingChart()    → ECharts实例化 [NEW]    │
│           ├─ initTrendChart()         → ECharts实例化          │
│           ├─ initFaultTypeChart()     → ECharts实例化          │
│           ├─ initResponsibleChart()   → ECharts实例化          │
│           ├─ initFloorChart()         → ECharts实例化          │
│           └─ initLocationChart()      → ECharts实例化          │
│                                                                  │
│  3. Tab切换处理                                                  │
│     onTabChange(tab) {                                          │
│       this.setData({ activeTab: tab });                        │
│       if (tab === 'charts') {                                   │
│         // 延迟初始化图表，确保DOM已渲染                         │
│         setTimeout(() => this.initAllCharts(), 100);           │
│       }                                                          │
│     }                                                            │
└──────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────┐
│                        云函数层                                   │
└─────────────────────────────────────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                    云函数调用 (云开发)                            │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐         │
│  │  云函数列表:                                        │         │
│  │  wx.cloud.callFunction({                           │         │
│  │    name: 'getAnalyticsOverview'                    │         │
│  │    name: 'getAnalyticsTrends'                      │         │
│  │    name: 'getAnalyticsByStatus'      [NEW]         │         │
│  │    name: 'getAnalyticsByCategory'                  │         │
│  │    name: 'getAnalyticsByFloor'                     │         │
│  │    name: 'getAnalyticsByLocation'                  │         │
│  │    name: 'getEmployeeRanking'                      │         │
│  │    name: 'getResponsiblePartyRanking'              │         │
│  │  })                                                │         │
│  └────────────────────────────────────────────────────┘         │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                   云数据库操作层                                  │
│                                                                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │ 云数据库查询    │  │ 数据聚合        │  │ 结果返回        │ │
│  │ db.collection() │→ │ aggregate()     │→ │ 格式化数据      │ │
│  │ where() 条件    │  │ group(), count()│  │ JSON返回        │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
│                                                                  │
│  云数据库特性:                                                    │
│  ├─ 自动索引优化                                                 │
│  ├─ 权限控制 (仅云函数可写)                                       │
│  └─ 数据缓存 (云端自动管理)                                       │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                        云数据库                                   │
│                                                                  │
│  workorders 集合:                                                │
│  ├─ _id, order_number, floor, location                          │
│  ├─ order_category, responsible_party, priority, status         │
│  ├─ report_time, completed_at, assigned_at, started_at         │
│  ├─ submitter_id, assigned_technician_id                        │
│  └─ created_at, updated_at                                      │
│                                                                  │
│  users 集合:                                                     │
│  ├─ _id, name, role_id                                          │
│  └─ created_at, updated_at                                      │
│                                                                  │
│  索引 (云数据库自动优化):                                         │
│  ├─ status                                                      │
│  ├─ created_at                                                  │
│  ├─ submitter_id                                                │
│  └─ assigned_technician_id                                      │
└──────────────────────────────────────────────────────────────────┘
```

## 4. 技术架构栈

```
┌─────────────────────────────────────────────────────────────────┐
│                          展示层                                   │
├─────────────────────────────────────────────────────────────────┤
│  微信小程序框架                                                   │
│  ├─ WXML (模板) - Tab切换、条件渲染                               │
│  ├─ WXSS (样式) - 响应式Grid布局                                 │
│  ├─ JavaScript (逻辑) - 状态管理、事件处理                        │
│  └─ JSON (配置) - ec-canvas组件注册                              │
├─────────────────────────────────────────────────────────────────┤
│  UI组件库                                                        │
│  ├─ 小程序内置组件 (view, text, scroll-view, picker)             │
│  └─ 自定义组件 (KPI卡片, 排名列表, Tab切换栏)                     │
├─────────────────────────────────────────────────────────────────┤
│  图表库                                                          │
│  ├─ echarts-for-weixin (官方ECharts适配器)                       │
│  │  ├─ ec-canvas 组件                                           │
│  │  └─ ECharts 5.x 核心 (支持环形图、折线图、饼图、柱状图)         │
│  └─ 图表配置工具 (chartUtils.js)                                 │
├─────────────────────────────────────────────────────────────────┤
│  动画库                                                          │
│  └─ wx.createAnimation() (原生API - 用于KPI卡片动画)             │
│     └─ 淡入 + 上滑效果 (fadeIn + translateY)                     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                         工具层                                    │
├─────────────────────────────────────────────────────────────────┤
│  日期工具 (dateUtils.js)                                         │
│  ├─ getYesterdayRange()                                         │
│  ├─ getTodayRange()                                             │
│  ├─ getWeekRange()                                              │
│  └─ getMonthRange()                                             │
├─────────────────────────────────────────────────────────────────┤
│  图表工具 (chartUtils.js)                                        │
│  ├─ getRingChartOption(data)              [NEW - 环形图]        │
│  ├─ getLineChartOption(xData, yData1, yData2)                   │
│  ├─ getPieChartOption(data)                                     │
│  └─ getBarChartOption(xData, yData)                             │
├─────────────────────────────────────────────────────────────────┤
│  动画工具 (animationUtils.js)                                    │
│  ├─ fadeInSlideUp(duration, delay)                              │
│  └─ 错开延迟控制                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        网络层                                     │
├─────────────────────────────────────────────────────────────────┤
│  API服务 (request.js)                                            │
│  ├─ wx.request() 封装                                            │
│  ├─ 请求拦截器 (添加token)                                        │
│  ├─ 响应拦截器 (错误处理)                                         │
│  └─ 重试机制                                                     │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       云开发后端层                                 │
├─────────────────────────────────────────────────────────────────┤
│  微信云开发 / 云函数                                              │
│  ├─ 云函数 (cloudfunctions/)                                     │
│  │  ├─ getAnalyticsOverview/                                    │
│  │  ├─ getAnalyticsTrends/                                      │
│  │  ├─ getAnalyticsByStatus/        [NEW]                       │
│  │  ├─ getAnalyticsByCategory/                                  │
│  │  ├─ getAnalyticsByFloor/                                     │
│  │  ├─ getAnalyticsByLocation/                                  │
│  │  ├─ getEmployeeRanking/                                      │
│  │  └─ getResponsiblePartyRanking/                              │
│  │                                                               │
│  └─ 云数据库 (Cloud Database)                                    │
│     ├─ workorders 集合                                           │
│     └─ users 集合                                                │
├─────────────────────────────────────────────────────────────────┤
│  云函数特性                                                       │
│  ├─ 自动扩缩容                                                   │
│  ├─ 免服务器运维                                                 │
│  ├─ 按量计费                                                     │
│  └─ 自带权限控制                                                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       云数据库层                                  │
├─────────────────────────────────────────────────────────────────┤
│  云数据库 (Cloud Database - 文档型数据库)                         │
│  ├─ workorders 集合 (工单数据)                                   │
│  ├─ users 集合 (用户数据)                                        │
│  └─ 聚合查询支持 (aggregate pipeline)                            │
├─────────────────────────────────────────────────────────────────┤
│  索引优化                                                        │
│  ├─ 云端自动索引推荐                                             │
│  ├─ 手动索引: status, created_at                                │
│  └─ 复合索引: {status: 1, created_at: -1}                       │
├─────────────────────────────────────────────────────────────────┤
│  数据缓存                                                        │
│  └─ 云端自动缓存 (无需手动配置Redis)                             │
└─────────────────────────────────────────────────────────────────┘
```

## 5. 交互流程图

### 场景1: 用户切换Tab

```
用户操作: 点击"可视化图表"Tab
    ↓
前端: onTabChange('charts') 被触发
    ↓
前端: setData({ activeTab: 'charts' })
    ↓
前端: WXML 条件渲染更新
    ├─ wx:if="{{activeTab === 'stats'}}" → 隐藏数据统计
    └─ wx:if="{{activeTab === 'charts'}}" → 显示可视化图表
    ↓
前端: 检查图表是否已初始化
    ├─ 如果未初始化 → 调用 initAllCharts()
    └─ 如果已初始化 → 跳过
    ↓
前端: 延迟100ms确保DOM渲染完成
    ↓
前端: 逐个初始化6个ECharts实例
    ├─ initStatusRingChart()     → 工单处理进度环形图
    ├─ initTrendChart()          → 工单趋势折线图
    ├─ initFaultTypeChart()      → 故障类型饼图
    ├─ initResponsibleChart()    → 责任方饼图
    ├─ initFloorChart()          → 楼层柱状图
    └─ initLocationChart()       → 位置柱状图
    ↓
用户: 看到可视化图表内容
```

### 场景2: 用户更改时间过滤器

```
用户操作: 选择"本周"时间过滤器
    ↓
前端: onWeekFilter() 被触发
    ↓
前端: 调用 dateUtils.getWeekRange()
    ↓
前端: 更新 startDate, endDate, timeFilter='week'
    ↓
前端: 调用 fetchAllData()
    ├─ wx.showLoading('加载中...')
    └─ Promise.all([
        // Tab 1 数据
        wx.cloud.callFunction({name: 'getAnalyticsOverview', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getEmployeeRanking', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getResponsiblePartyRanking', data: {startDate, endDate}}),
        // Tab 2 数据
        wx.cloud.callFunction({name: 'getAnalyticsByStatus', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getAnalyticsTrends', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getAnalyticsByCategory', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getAnalyticsByResponsibleParty', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getAnalyticsByFloor', data: {startDate, endDate}}),
        wx.cloud.callFunction({name: 'getAnalyticsByLocation', data: {startDate, endDate}})
    ])
    ↓
云函数: 接收9个并行调用
    ↓
云函数: 云端自动处理并发请求
    ↓
云数据库: 执行查询 (where条件: created_at在时间范围内)
    ├─ db.collection('workorders').where({...})
    └─ aggregate([...]) 聚合管道
        ↓
        聚合计算: $group, $count, $avg
        ↓
        云端自动缓存
        ↓
        返回JSON数据
    ↓
前端: 所有Promise resolved
    ↓
前端: 更新页面数据 (setData)
    ↓
前端: 根据当前activeTab刷新视图
    ├─ if activeTab === 'stats':
    │  ├─ 更新KPI卡片数值
    │  ├─ 播放卡片动画 (可选)
    │  └─ 刷新排名列表
    │
    └─ if activeTab === 'charts':
       └─ 重新初始化所有图表 (initAllCharts)
          ├─ 环形图更新
          ├─ 趋势图更新
          ├─ 饼图更新
          └─ 柱状图更新
    ↓
前端: wx.hideLoading()
    ↓
用户: 看到更新后的"本周"数据
```

### 场景3: 用户点击"服务简报"按钮

```
用户操作: 在"可视化图表"Tab中点击右上角"服务简报"按钮
    ↓
前端: onServiceReportClick() 被触发
    ↓
前端: 显示加载提示 wx.showLoading('生成中...')
    ↓
前端: 收集当前数据
    ├─ 时间范围: startDate, endDate
    ├─ KPI指标: totalOrders, completedOrders, avgTime
    ├─ 图表数据: statusDistribution, trendData, etc.
    └─ 排名数据: employeeRanking, responsiblePartyRanking
    ↓
前端: 调用后端生成简报云函数
    wx.cloud.callFunction({
      name: 'generateServiceReport',
      data: { dateRange, metrics, chartData, rankings }
    })
    ↓
云函数: 生成PDF/图片简报
    ├─ 渲染HTML模板
    ├─ 转换为PDF/图片
    └─ 上传到云存储 (Cloud Storage)
    ↓
云函数: 返回云存储URL
    ↓
前端: wx.hideLoading()
    ↓
前端: 预览简报
    wx.previewImage() 或 wx.downloadFile() + wx.openDocument()
    ↓
用户: 查看/分享/保存服务简报
```

## 6. 文件结构和修改策略

**🔴 代码修改范围说明：**
- 🔄 **物业经理视图完全重构**：删除 `role_id=2` 的旧代码，替换为全新Tab设计
- ❌ **不修改现有代码**：物业员工（role_id=4）和维修员（role_id=3）的代码保持 100% 不变
- ✅ **条件渲染隔离**：使用 `wx:if` 完全隔离不同角色的代码
- ✅ **Tab切换实现**：使用 `activeTab` 状态变量控制视图切换

**重构步骤：**
1. 找到现有 `wx:if="{{isManager}}"` 或 `role_id === 2` 的所有代码块
2. **删除**这些代码块中的所有内容
3. 用新的Tab设计替换
4. 确保不影响其他角色的代码块

```
miniprogram/
├── pages/
│   └── data/
│       ├── index.js           # 修改说明：
│       │                      # 🔄 删除: 物业经理旧的数据逻辑和方法
│       │                      # ✅ 添加: isManager, activeTab 数据属性
│       │                      # ✅ 添加: onTabChange() Tab切换方法
│       │                      # ✅ 添加: 新的数据获取方法（云函数调用）
│       │                      # ✅ 添加: 6个图表初始化方法
│       │                      # ✅ 添加: fetchStatusDistribution() [NEW]
│       │                      # ✅ 添加: onServiceReportClick() [NEW]
│       │                      # ❌ 不改: 物业员工/维修工的逻辑
│       │
│       ├── index.wxml         # 修改说明：
│       │                      # 🔄 删除: <block wx:if="{{isManager}}"> 内的所有旧代码
│       │                      # ✅ 添加: <view class="tab-bar"> Tab切换栏
│       │                      # ✅ 添加: <block wx:if="{{isManager && activeTab === 'stats'}}">
│       │                      #        数据统计Tab内容（全新）
│       │                      # ✅ 添加: <block wx:if="{{isManager && activeTab === 'charts'}}">
│       │                      #        可视化图表Tab内容（全新）
│       │                      # ❌ 不改: 现有的 wx:if="{{!isManager}}" 模板
│       │                      #
│       │                      # 示例代码结构：
│       │                      # <!-- 现有代码 - 不修改 -->
│       │                      # <block wx:if="{{!isManager}}">
│       │                      #   ... 物业员工/维修工视图（完全不动）...
│       │                      # </block>
│       │                      #
│       │                      # <!-- 重构代码 - 物业经理视图 -->
│       │                      # <block wx:if="{{isManager}}">
│       │                      #   <!-- 【删除这里的所有旧代码】 -->
│       │                      #
│       │                      #   <!-- 【替换为新代码】 -->
│       │                      #   <!-- Tab切换栏 -->
│       │                      #   <view class="tab-bar">
│       │                      #     <view class="tab {{activeTab==='stats'?'active':''}}"
│       │                      #           bindtap="onTabChange" data-tab="stats">
│       │                      #       数据统计
│       │                      #     </view>
│       │                      #     <view class="tab {{activeTab==='charts'?'active':''}}"
│       │                      #           bindtap="onTabChange" data-tab="charts">
│       │                      #       可视化图表
│       │                      #     </view>
│       │                      #   </view>
│       │                      #
│       │                      #   <!-- 共享：日期选择器 -->
│       │                      #   <view class="date-filter">...</view>
│       │                      #
│       │                      #   <!-- Tab 1: 数据统计 -->
│       │                      #   <block wx:if="{{activeTab === 'stats'}}">
│       │                      #     <view class="kpi-cards">...</view>
│       │                      #     <view class="rankings">...</view>
│       │                      #   </block>
│       │                      #
│       │                      #   <!-- Tab 2: 可视化图表 -->
│       │                      #   <block wx:if="{{activeTab === 'charts'}}">
│       │                      #     <view class="service-report-btn">服务简报</view>
│       │                      #     <ec-canvas id="ringChart">...</ec-canvas>
│       │                      #     <ec-canvas id="trendChart">...</ec-canvas>
│       │                      #     ... 其他图表 ...
│       │                      #   </block>
│       │                      # </block>
│       │
│       ├── index.wxss         # 修改说明：
│       │                      # 🔄 删除: .manager-* 等物业经理旧样式类（如果有）
│       │                      # ✅ 添加: .tab-bar, .tab, .tab.active 样式
│       │                      # ✅ 添加: 物业经理视图的新样式类
│       │                      # ✅ 添加: .service-report-btn 样式
│       │                      # ❌ 不改: 现有其他角色的样式类
│       │
│       └── index.json         # 修改说明：
│                              # ✅ 添加: "ec-canvas": "/components/ec-canvas/ec-canvas"
│                              # ❌ 不改: 现有配置
│
├── components/
│   └── ec-canvas/             # [NEW] 全新添加的ECharts组件
│       ├── ec-canvas.js
│       ├── ec-canvas.wxml
│       ├── ec-canvas.wxss
│       └── echarts.js         # ECharts 5.x 完整库
│
├── utils/
│   ├── dateUtils.js           # [NEW] 全新文件 - 日期工具
│   ├── chartUtils.js          # [NEW] 全新文件 - 图表工具
│   │                          #       包含 getRingChartOption() [NEW]
│   ├── animationUtils.js      # [NEW] 全新文件 - 动画工具
│   ├── constants.js           # ❌ 不修改 (除非需要添加新常量)
│   └── request.js             # ❌ 不修改
│
└── app.json                   # ❌ 不修改

cloudfunctions/                 # 云函数目录
├── getAnalyticsOverview/       # [可能已存在] KPI指标云函数
│   ├── index.js               # 云函数入口
│   └── package.json
│
├── getAnalyticsTrends/         # [可能已存在] 趋势数据云函数
│   ├── index.js
│   └── package.json
│
├── getAnalyticsByStatus/       # [NEW] 状态分布云函数
│   ├── index.js               # 查询各状态工单数量
│   └── package.json
│
├── getAnalyticsByCategory/     # [可能已存在] 故障类型分布云函数
│   ├── index.js
│   └── package.json
│
├── getAnalyticsByFloor/        # [NEW] 楼层分布云函数
│   ├── index.js               # 查询各楼层工单数量
│   └── package.json
│
├── getAnalyticsByLocation/     # [NEW] 位置分布云函数
│   ├── index.js               # 查询各位置工单数量
│   └── package.json
│
├── getEmployeeRanking/         # [NEW] 员工排名云函数
│   ├── index.js               # 查询员工已完成工单排名
│   └── package.json
│
├── getResponsiblePartyRanking/ # [NEW] 责任方排名云函数
│   ├── index.js               # 查询责任方已修复工单排名
│   └── package.json
│
└── generateServiceReport/      # [NEW - 可选] 生成服务简报云函数
    ├── index.js               # 生成PDF/图片简报
    ├── package.json
    └── templates/
        └── report-template.html
```

### 关键实现策略

**1. Tab切换实现 (index.js)**
```javascript
data: {
  isManager: false,
  activeTab: 'stats',  // 'stats' | 'charts'
  chartsInitialized: false
},

onTabChange(e) {
  const tab = e.currentTarget.dataset.tab;
  this.setData({ activeTab: tab });

  // 如果切换到图表tab且图表未初始化，延迟初始化
  if (tab === 'charts' && !this.data.chartsInitialized) {
    setTimeout(() => {
      this.initAllCharts();
      this.setData({ chartsInitialized: true });
    }, 100);
  }
}
```

**2. 条件渲染隔离 (index.wxml)**
```xml
<!-- 现有视图 - 完全不动 -->
<block wx:if="{{!isManager}}">
  ... 物业员工/维修工视图 ...
</block>

<!-- 新增物业经理视图 - 完全隔离 -->
<block wx:if="{{isManager}}">
  <!-- Tab切换栏 -->
  <view class="tab-bar">
    <view class="tab {{activeTab==='stats'?'active':''}}" bindtap="onTabChange" data-tab="stats">数据统计</view>
    <view class="tab {{activeTab==='charts'?'active':''}}" bindtap="onTabChange" data-tab="charts">可视化图表</view>
  </view>

  <!-- 共享日期选择器 -->
  <view class="date-filter">...</view>

  <!-- Tab 1 内容 -->
  <block wx:if="{{activeTab === 'stats'}}">
    <view class="kpi-cards">...</view>
    <view class="rankings">...</view>
  </block>

  <!-- Tab 2 内容 -->
  <block wx:if="{{activeTab === 'charts'}}">
    <view class="service-report-btn" bindtap="onServiceReportClick">服务简报</view>
    <ec-canvas id="ringChart" canvas-id="ringChart"></ec-canvas>
    <ec-canvas id="trendChart" canvas-id="trendChart"></ec-canvas>
    ... 其他图表 ...
  </block>
</block>
```

**3. 样式隔离 (index.wxss)**
```css
/* 现有样式 - 完全不动 */

/* 新增物业经理样式 - 完全隔离 */
.tab-bar {
  display: flex;
  background: #fff;
  padding: 10rpx 32rpx;
  border-bottom: 1rpx solid #eee;
}

.tab {
  flex: 1;
  text-align: center;
  padding: 20rpx 0;
  font-size: 30rpx;
  color: #666;
  position: relative;
}

.tab.active {
  color: #07c160;
  font-weight: bold;
}

.tab.active::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 60rpx;
  height: 4rpx;
  background: #07c160;
  border-radius: 2rpx;
}

.service-report-btn {
  position: absolute;
  top: 20rpx;
  right: 32rpx;
  padding: 10rpx 24rpx;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border-radius: 20rpx;
  font-size: 26rpx;
  z-index: 10;
}
```

---

## 总结

这个架构设计完整展示了：

1. **页面布局** - 使用Tab切换分离"数据统计"和"可视化图表"两个模块
2. **组件层次** - 所有子组件和嵌套关系，明确Tab切换逻辑
3. **数据流** - 从用户操作到数据展示的完整流程，包括Tab切换和时间过滤
4. **技术栈** - 使用的所有技术和工具，新增环形图支持
5. **交互流程** - 3个典型场景的详细步骤（Tab切换、时间过滤、服务简报）
6. **文件结构** - 需要创建/修改的所有文件，明确重构策略和代码隔离方案

**关键特性：**
- 🔄 **完全重构物业经理视图** - 删除旧内容，替换为全新Tab设计
- ❌ **零破坏性改动** - 物业员工和维修工视图一行代码都不动
- ✅ **Tab切换设计** - 清晰分离统计和图表两个模块
- ✅ **完全代码隔离** - 使用 `wx:if` 条件渲染确保角色之间完全隔离
- ✅ **共享日期选择器** - 两个Tab使用同一套时间过滤数据
- ✅ **环形图展示** - 工单处理进度使用彩色甜甜圈图
- ✅ **服务简报功能** - 支持一键生成和分享数据报告
- ✅ **延迟图表初始化** - 仅在切换到图表Tab时才初始化ECharts，提升性能
- ✅ **云开发架构** - 使用微信云函数 + 云数据库，免服务器运维
- ✅ **自动缓存优化** - 云端自动管理数据缓存，无需手动配置Redis

**重构原则：**
1. **删除优先** - 先删除物业经理的旧代码，再写新代码
2. **隔离第一** - 不同角色的代码通过 `wx:if` 完全隔离，绝不交叉
3. **简单至上** - 每次修改影响最少代码，避免复杂重构
4. **零风险** - 物业员工和维修工的功能绝对不受影响
