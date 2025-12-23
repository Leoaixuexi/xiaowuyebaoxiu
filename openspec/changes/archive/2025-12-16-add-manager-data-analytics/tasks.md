# 实施任务清单 - 物业经理数据页面重构

**🔴 重要提示：**
- 本次是**完全重构**物业经理视图（role_id=2），删除旧代码，替换为全新Tab设计
- 物业员工（role_id=4）和维修员（role_id=3）视图**完全不动**，一行代码都不改
- 使用**微信云开发**架构（云函数 + 云数据库）
- 实现**Tab切换**：数据统计 / 可视化图表

---

## 阶段 0：依赖项设置

### 0.1 安装和配置 ECharts
- [ ] 0.1.1 从GitHub下载echarts-for-weixin (https://github.com/ecomfe/echarts-for-weixin)
- [ ] 0.1.2 将`ec-canvas`组件复制到`/miniprogram/components/ec-canvas/`
- [ ] 0.1.3 在`/miniprogram/pages/data/index.json`中注册ec-canvas组件：
  ```json
  {
    "usingComponents": {
      "ec-canvas": "/components/ec-canvas/ec-canvas"
    }
  }
  ```
- [ ] 0.1.4 创建测试页面验证ECharts基本渲染功能
- [ ] 0.1.5 验证环形图、折线图、饼图、柱状图都能正常渲染

### 0.2 准备动画工具
- [ ] 0.2.1 在`/miniprogram/utils/animationUtils.js`创建动画工具模块
- [ ] 0.2.2 使用wx.createAnimation()原生API实现淡入和上滑动画：
  ```javascript
  // fadeInSlideUp(duration = 300, delay = 0)
  function fadeInSlideUp(duration, delay) {
    const animation = wx.createAnimation({
      duration,
      timingFunction: 'ease-out',
      delay
    });
    animation.opacity(1).translateY(0).step();
    return animation.export();
  }
  ```
- [ ] 0.2.3 在示例组件上测试动画效果
- [ ] 0.2.4 验证错开延迟动画（0ms, 100ms, 200ms, 300ms）

### 0.3 准备工具函数
- [ ] 0.3.1 在`/miniprogram/utils/dateUtils.js`创建日期工具模块
- [ ] 0.3.2 实现时间范围函数：
  - `getYesterdayRange()` - 返回昨天的开始和结束时间
  - `getTodayRange()` - 返回今天的开始和结束时间
  - `getWeekRange()` - 返回本周的开始和结束时间
  - `getMonthRange()` - 返回本月的开始和结束时间
- [ ] 0.3.3 在`/miniprogram/utils/chartUtils.js`创建图表配置工具
- [ ] 0.3.4 实现图表配置函数：
  - `getRingChartOption(data)` - 环形图配置 [NEW]
  - `getLineChartOption(xData, yData1, yData2)` - 折线图配置
  - `getPieChartOption(data)` - 饼图配置
  - `getBarChartOption(xData, yData)` - 柱状图配置

---

## 阶段 1：云函数开发

**注意：使用微信云开发云函数替代传统后端API**

### 1.1 验证现有云函数
- [ ] 1.1.1 检查`/cloudfunctions/getAnalyticsOverview/`是否存在
- [ ] 1.1.2 验证返回数据格式包含：totalOrders, completedOrders, inProgressOrders, avgCompletionTime
- [ ] 1.1.3 检查`/cloudfunctions/getAnalyticsTrends/`是否存在
- [ ] 1.1.4 验证返回每日的submitted和completed数据
- [ ] 1.1.5 检查`/cloudfunctions/getAnalyticsByCategory/`是否存在
- [ ] 1.1.6 验证返回故障类型分布数据

### 1.2 创建新云函数 - 状态分布（环形图）[NEW]
- [ ] 1.2.1 创建云函数目录`/cloudfunctions/getAnalyticsByStatus/`
- [ ] 1.2.2 创建index.js，实现以下逻辑：
  ```javascript
  const cloud = require('wx-server-sdk');
  cloud.init({ env: cloud.DYNAMIC_CURRENT_ENV });
  const db = cloud.database();

  exports.main = async (event, context) => {
    const { startDate, endDate } = event;
    const result = await db.collection('workorders')
      .where({
        created_at: db.command.gte(new Date(startDate)).and(db.command.lte(new Date(endDate)))
      })
      .aggregate()
      .group({
        _id: '$status',
        count: db.command.sum(1)
      })
      .end();
    return result;
  };
  ```
- [ ] 1.2.3 创建package.json配置文件
- [ ] 1.2.4 上传并部署云函数
- [ ] 1.2.5 在小程序端测试调用

### 1.3 创建新云函数 - 楼层分布
- [ ] 1.3.1 创建云函数目录`/cloudfunctions/getAnalyticsByFloor/`
- [ ] 1.3.2 创建index.js，按floor字段聚合工单数量
- [ ] 1.3.3 添加排序逻辑（按数量降序）
- [ ] 1.3.4 创建package.json
- [ ] 1.3.5 上传并部署云函数
- [ ] 1.3.6 测试云函数调用

### 1.4 创建新云函数 - 位置分布
- [ ] 1.4.1 创建云函数目录`/cloudfunctions/getAnalyticsByLocation/`
- [ ] 1.4.2 创建index.js，按location字段聚合工单数量
- [ ] 1.4.3 添加排序逻辑（按数量降序）
- [ ] 1.4.4 创建package.json
- [ ] 1.4.5 上传并部署云函数
- [ ] 1.4.6 测试云函数调用

### 1.5 创建新云函数 - 员工排名
- [ ] 1.5.1 创建云函数目录`/cloudfunctions/getEmployeeRanking/`
- [ ] 1.5.2 创建index.js，实现以下逻辑：
  - 查询日期范围内的工单
  - 联接users集合（role_id=4）
  - 按submitter_id分组
  - 统计总提交数和已完成数
  - 按已完成数降序排序
- [ ] 1.5.3 创建package.json
- [ ] 1.5.4 上传并部署云函数
- [ ] 1.5.5 测试云函数调用

### 1.6 创建新云函数 - 责任方排名
- [ ] 1.6.1 创建云函数目录`/cloudfunctions/getResponsiblePartyRanking/`
- [ ] 1.6.2 创建index.js，实现以下逻辑：
  - 查询日期范围内status="已完成"的工单
  - 按responsible_party分组
  - 统计每个责任方的工单数和百分比
  - 按数量降序排序
- [ ] 1.6.3 创建package.json
- [ ] 1.6.4 上传并部署云函数
- [ ] 1.6.5 测试云函数调用

### 1.7 创建新云函数 - 服务简报生成（可选）
- [ ] 1.7.1 创建云函数目录`/cloudfunctions/generateServiceReport/`
- [ ] 1.7.2 创建index.js，实现PDF/图片生成逻辑
- [ ] 1.7.3 集成云存储，上传生成的文件
- [ ] 1.7.4 返回云存储URL
- [ ] 1.7.5 上传并部署云函数
- [ ] 1.7.6 测试简报生成功能

### 1.8 云函数集成测试
- [ ] 1.8.1 使用小程序开发工具测试所有云函数调用
- [ ] 1.8.2 验证日期范围过滤正常工作
- [ ] 1.8.3 验证云端自动缓存生效
- [ ] 1.8.4 验证返回数据格式符合前端需求
- [ ] 1.8.5 使用大数据集测试性能（1000+工单）

---

## 阶段 2：前端 - 删除旧代码并准备重构

**🔴 关键步骤：先删除物业经理的旧代码，再写新代码**

### 2.1 备份和清理
- [ ] 2.1.1 创建代码备份（git commit或复制文件）
- [ ] 2.1.2 打开`/miniprogram/pages/data/index.js`
- [ ] 2.1.3 **找到并删除**所有 `role_id === 2` 或 `isManager` 相关的旧代码：
  - 删除旧的数据属性
  - 删除旧的方法
  - 删除旧的事件处理函数
  - **保留** role_id=4 和 role_id=3 的所有代码
- [ ] 2.1.4 打开`/miniprogram/pages/data/index.wxml`
- [ ] 2.1.5 **找到并删除** `<block wx:if="{{isManager}}">` 内的所有旧模板代码
  - **保留** `<block wx:if="{{!isManager}}">` 的所有代码（物业员工/维修员）
- [ ] 2.1.6 打开`/miniprogram/pages/data/index.wxss`
- [ ] 2.1.7 **删除**所有 `.manager-*` 或物业经理相关的旧样式类
  - **保留**其他角色的样式

### 2.2 实现新的数据结构
- [ ] 2.2.1 在`/miniprogram/pages/data/index.js`的data中添加新属性：
  ```javascript
  data: {
    // 角色检测
    isManager: false,

    // Tab切换
    activeTab: 'stats',  // 'stats' | 'charts'
    chartsInitialized: false,

    // 时间过滤
    timeFilter: 'today',
    startDate: '',
    endDate: '',
    showCustomPicker: false,

    // KPI数据
    totalOrders: 0,
    completedOrders: 0,
    inProgressOrders: 0,
    avgCompletionTime: 0,

    // 排名数据
    employeeRanking: [],
    responsiblePartyRanking: [],

    // 图表数据
    statusDistribution: [],  // 环形图
    trendData: { dates: [], submitted: [], completed: [] },
    faultTypeData: [],
    responsiblePartyData: [],
    floorData: [],
    locationData: [],

    // UI状态
    isLoading: false
  }
  ```

### 2.3 实现角色检测
- [ ] 2.3.1 在onLoad()中添加角色检测逻辑：
  ```javascript
  onLoad() {
    const userInfo = wx.getStorageSync('userInfo');
    if (userInfo.role_id === 2) {
      this.setData({
        isManager: true,
        activeTab: 'stats'
      });
      this.initDateRange('today');
      this.fetchAllData();
    }
    // 保留其他角色的现有逻辑
  }
  ```

---

## 阶段 3：模块1 - 日期范围选择器 & KPI卡片

### 3.1 实现日期范围选择器
- [ ] 3.1.1 在WXML中创建日期过滤标签栏：
  ```xml
  <view class="date-filter-bar">
    <view class="filter-tab {{timeFilter === 'yesterday' ? 'active' : ''}}" bindtap="onYesterdayFilter">昨</view>
    <view class="filter-tab {{timeFilter === 'today' ? 'active' : ''}}" bindtap="onTodayFilter">今</view>
    <view class="filter-tab {{timeFilter === 'week' ? 'active' : ''}}" bindtap="onWeekFilter">本周</view>
    <view class="filter-tab {{timeFilter === 'month' ? 'active' : ''}}" bindtap="onMonthFilter">本月</view>
    <view class="filter-tab {{timeFilter === 'custom' ? 'active' : ''}}" bindtap="onCustomFilter">自定义</view>
  </view>
  ```
- [ ] 3.1.2 在index.js中实现过滤器处理方法（onYesterdayFilter, onTodayFilter等）
- [ ] 3.1.3 在`/miniprogram/utils/dateUtils.js`中创建日期范围计算工具：
  - getYesterdayRange() - 返回昨天的{ startDate, endDate }
  - getTodayRange() - 返回今天的{ startDate, endDate }
  - getWeekRange() - 返回本周的{ startDate, endDate }（周一-周日）
  - getMonthRange() - 返回本月的{ startDate, endDate }
- [ ] 3.1.4 使用`wx.chooseDate`或日期选择器组件实现自定义日期选择器
- [ ] 3.1.5 添加自动刷新逻辑：时间过滤器变更时调用fetchAllData()
- [ ] 3.1.6 在index.wxss中为日期过滤栏添加样式，包含激活状态高亮

### 3.2 实现KPI卡片（4个卡片）
- [ ] 3.2.1 在WXML中创建KPI卡片网格布局：
  ```xml
  <view class="kpi-cards-grid">
    <view class="kpi-card card-1" animation="{{card1Animation}}">
      <view class="card-icon">📋</view>
      <view class="card-value">{{totalOrders}}</view>
      <view class="card-label">总工单数</view>
    </view>
    <!-- 其他3个卡片重复 -->
  </view>
  ```
- [ ] 3.2.2 实现卡片1：总工单数
  - 图标：📋（蓝色）
  - 数值：totalOrders
  - 标签："总工单数"

- [ ] 3.2.3 实现卡片2：已完成工单
  - 数值：completedOrders
  - 副标题：completionRate + "%"
  - 标签："已完成"

- [ ] 3.2.4 实现卡片3：进行中
  - 数值：inProgressOrders
  - 标签："进行中"

- [ ] 3.2.5 实现卡片4：平均完成时间
  - 数值：avgCompletionTime（格式化为1位小数）
  - 标签："平均完成时间 (小时)"

- [ ] 3.2.6 在index.js中实现卡片动画：
  ```js
  initCardAnimations() {
    const cards = ['card1', 'card2', 'card3', 'card4'];
    cards.forEach((card, index) => {
      setTimeout(() => {
        const animation = animationUtils.fadeInSlideUp(300);
        this.setData({ [`${card}Animation`]: animation.export() });
      }, index * 100); // 错开延迟
    });
  }
  ```
- [ ] 3.2.7 在index.wxss中为KPI卡片添加样式：
  - 移动端2列网格布局
  - 卡片阴影和边框
  - 响应式间距
  - 卡片1图标的蓝色

### 3.3 实现排名
- [ ] 3.3.1 在WXML中创建员工排名部分：
  ```xml
  <view class="ranking-section">
    <view class="section-title">员工已完成工单排名</view>
    <view class="ranking-list">
      <view class="ranking-item" wx:for="{{employeeRanking}}" wx:key="user_id">
        <view class="rank-number">{{index + 1}}</view>
        <view class="employee-name">{{item.name}}</view>
        <view class="completed-count">{{item.completed}}</view>
      </view>
    </view>
  </view>
  ```
- [ ] 3.3.2 在WXML中创建责任方排名部分
- [ ] 3.3.3 从GET /analytics/employee-ranking获取员工排名数据
- [ ] 3.3.4 从GET /analytics/responsible-party-ranking获取责任方排名
- [ ] 3.3.5 为排名部分添加适当的布局和颜色样式

---

## 阶段 4：模块2 - ECharts可视化

### 4.1 设置ECharts组件
- [ ] 4.1.1 在page.json中注册ec-canvas组件：
  ```json
  {
    "usingComponents": {
      "ec-canvas": "/components/ec-canvas/ec-canvas"
    }
  }
  ```
- [ ] 4.1.2 创建图表工具模块`/miniprogram/utils/chartUtils.js`，包含方法：
  - initChart(canvasId, option) - 初始化ECharts实例
  - updateChart(chart, option) - 用新数据更新图表
  - getLineChartOption(xData, yData1, yData2) - 生成折线图配置
  - getPieChartOption(data) - 生成饼图配置
  - getBarChartOption(xData, yData) - 生成柱状图配置

### 4.2 实现图表1：工单趋势折线图
- [ ] 4.2.1 在WXML中添加ec-canvas组件：
  ```xml
  <view class="chart-container">
    <view class="chart-title">工单趋势图</view>
    <ec-canvas id="trendChart" canvas-id="trend-chart" ec="{{ trendChartEc }}"></ec-canvas>
  </view>
  ```
- [ ] 4.2.2 在index.js中初始化趋势图：
  ```js
  initTrendChart() {
    this.setData({
      trendChartEc: {
        onInit: (canvas, width, height, dpr) => {
          const chart = echarts.init(canvas, null, { width, height, devicePixelRatio: dpr });
          const option = chartUtils.getLineChartOption(
            this.data.trendData.dates,
            this.data.trendData.submitted,
            this.data.trendData.completed
          );
          chart.setOption(option);
          return chart;
        }
      }
    });
  }
  ```
- [ ] 4.2.3 配置折线图选项：
  - X轴：MM-DD格式的日期
  - Y轴：工单数量
  - 系列1：已提交工单的蓝色线
  - 系列2：已完成工单的绿色线
  - 数据点点击时的提示框
  - 可点击图例
- [ ] 4.2.4 从GET /analytics/trends?period=daily获取趋势数据
- [ ] 4.2.5 将API响应转换为图表格式
- [ ] 4.2.6 实现时间过滤器变更时的图表更新

### 4.3 实现图表2：故障类型分布饼图
- [ ] 4.3.1 为故障类型图表添加ec-canvas组件
- [ ] 4.3.2 初始化故障类型饼图
- [ ] 4.3.3 配置饼图选项：
  - 数据：故障类型名称和数量
  - 标签格式化器：`{b}: {d}%`（名称 + 百分比）
  - 显示数量和百分比的提示框
  - 包含所有故障类型的图例
  - 5个类别的调色板
- [ ] 4.3.4 从GET /analytics/by-category获取数据
- [ ] 4.3.5 将数据转换为ECharts格式：`[{name: '电梯维修', value: 15}, ...]`

### 4.4 实现图表3：责任方分布饼图
- [ ] 4.4.1 为责任方图表添加ec-canvas组件
- [ ] 4.4.2 初始化责任方饼图
- [ ] 4.4.3 配置饼图选项，类似故障类型图表
- [ ] 4.4.4 从GET /analytics/responsible-party-ranking获取数据或从工单中提取
- [ ] 4.4.5 将数据转换为ECharts格式

### 4.5 实现图表4：楼层分布柱状图
- [ ] 4.5.1 为楼层图表添加ec-canvas组件
- [ ] 4.5.2 初始化楼层柱状图
- [ ] 4.5.3 配置柱状图选项：
  - X轴：楼层名称/编号
  - Y轴：工单数量
  - 柱子按降序排序
  - 柱子点击时的提示框
  - 柱子颜色：蓝色渐变
- [ ] 4.5.4 从GET /analytics/by-floor获取数据
- [ ] 4.5.5 按数量降序排序数据
- [ ] 4.5.6 转换为ECharts格式：xData = ['1F', '2F', ...], yData = [25, 18, ...]

### 4.6 实现图表5：位置分布柱状图
- [ ] 4.6.1 为位置图表添加ec-canvas组件
- [ ] 4.6.2 初始化位置柱状图
- [ ] 4.6.3 配置柱状图选项，类似楼层图表
- [ ] 4.6.4 从GET /analytics/by-location获取数据
- [ ] 4.6.5 按数量降序排序数据
- [ ] 4.6.6 转换为ECharts格式

### 4.7 图表样式和布局
- [ ] 4.7.1 在WXSS中创建响应式图表网格布局：
  - 移动端1列
  - 平板及更大屏幕2列
- [ ] 4.7.2 设置图表容器高度（例如300px）
- [ ] 4.7.3 添加一致样式的图表标题
- [ ] 4.7.4 添加图表之间的间距
- [ ] 4.7.5 确保内容溢出时图表可滚动

---

## 阶段 5：数据获取和集成

### 5.1 实现数据获取方法
- [ ] 5.1.1 创建`fetchKPIMetrics()`方法：
  ```js
  async fetchKPIMetrics() {
    const { startDate, endDate } = this.data;
    const res = await api.get('/analytics/overview', { startDate, endDate });
    this.setData({
      totalOrders: res.data.totalOrders,
      completedOrders: res.data.completedOrders,
      inProgressOrders: res.data.inProgressOrders,
      avgCompletionTime: (res.data.avgCompletionTime / 3600).toFixed(1) // 将秒转换为小时
    });
  }
  ```
- [ ] 5.1.2 创建`fetchTrendData()`方法调用GET /analytics/trends
- [ ] 5.1.3 创建`fetchFaultTypeData()`方法调用GET /analytics/by-category
- [ ] 5.1.4 创建`fetchResponsiblePartyData()`方法
- [ ] 5.1.5 创建`fetchFloorData()`方法调用GET /analytics/by-floor
- [ ] 5.1.6 创建`fetchLocationData()`方法调用GET /analytics/by-location
- [ ] 5.1.7 创建`fetchEmployeeRanking()`方法
- [ ] 5.1.8 创建`fetchResponsiblePartyRanking()`方法

### 5.2 实现统一数据获取
- [ ] 5.2.1 创建`fetchAllData()`主方法，并行调用所有获取方法：
  ```js
  async fetchAllData() {
    wx.showLoading({ title: '加载中...' });
    try {
      await Promise.all([
        this.fetchKPIMetrics(),
        this.fetchTrendData(),
        this.fetchFaultTypeData(),
        this.fetchResponsiblePartyData(),
        this.fetchFloorData(),
        this.fetchLocationData(),
        this.fetchEmployeeRanking(),
        this.fetchResponsiblePartyRanking()
      ]);
      this.initAllCharts();
      this.initCardAnimations();
    } catch (error) {
      wx.showToast({ title: '数据加载失败', icon: 'none' });
      console.error('Data fetch error:', error);
    } finally {
      wx.hideLoading();
    }
  }
  ```
- [ ] 5.2.2 在onLoad生命周期中调用fetchAllData()（角色检测后）
- [ ] 5.2.3 时间过滤器变更时调用fetchAllData()

### 5.3 实现错误处理
- [ ] 5.3.1 为所有获取方法添加try-catch块
- [ ] 5.3.2 使用wx.showToast显示用户友好的错误消息
- [ ] 5.3.3 为失败的请求实现重试机制
- [ ] 5.3.4 数据加载失败时为图表显示占位符/空状态
- [ ] 5.3.5 将错误记录到控制台以供调试

---

## 阶段 6：刷新和自动刷新

### 6.1 实现手动刷新
- [ ] 6.1.1 在WXML中添加刷新按钮：
  ```xml
  <view class="refresh-btn" bindtap="onManualRefresh">
    <text class="icon-refresh">🔄</text>
    <text>刷新</text>
  </view>
  ```
- [ ] 6.1.2 实现onManualRefresh()处理器：
  ```js
  onManualRefresh() {
    this.fetchAllData();
    this.setData({ lastRefreshTime: new Date().toLocaleTimeString() });
  }
  ```
- [ ] 6.1.3 显示最后刷新时间戳

### 6.2 实现自动刷新
- [ ] 6.2.1 在WXML中添加自动刷新切换开关：
  ```xml
  <switch checked="{{autoRefresh}}" bindchange="onAutoRefreshToggle" />
  ```
- [ ] 6.2.2 实现onAutoRefreshToggle()处理器
- [ ] 6.2.3 使用setInterval实现自动刷新计时器：
  ```js
  startAutoRefresh() {
    this.autoRefreshTimer = setInterval(() => {
      this.fetchAllData();
      this.setData({ refreshCountdown: 300 }); // 重置为5分钟（300秒）
    }, 300000); // 5分钟

    this.countdownTimer = setInterval(() => {
      if (this.data.refreshCountdown > 0) {
        this.setData({ refreshCountdown: this.data.refreshCountdown - 1 });
      }
    }, 1000);
  }
  ```
- [ ] 6.2.4 显示倒计时器，显示距下次刷新的秒数
- [ ] 6.2.5 实现stopAutoRefresh()以清除间隔
- [ ] 6.2.6 在onUnload生命周期中清除计时器以防止内存泄漏

---

## 阶段 7：样式和响应式设计

### 7.1 创建管理员视图样式
- [ ] 7.1.1 打开`/miniprogram/pages/data/index.wxss`
- [ ] 7.1.2 创建日期过滤栏样式：
  - 水平flex布局
  - 蓝色背景的激活状态
  - 边框圆角和间距
- [ ] 7.1.3 创建KPI卡片网格样式：
  - 移动端2列网格（display: grid; grid-template-columns: 1fr 1fr;）
  - 卡片阴影、内边距、边框圆角
  - 图标大小和颜色
  - 数值字体大小（大、粗体）
  - 副标题字体大小（小、灰色）
- [ ] 7.1.4 创建排名部分样式：
  - 部分标题样式
  - 列表项布局（排名数字、姓名、数量）
  - 交替行颜色
- [ ] 7.1.5 创建图表容器样式：
  - 图表网格布局（移动端1列，平板+ 2列）
  - 图表标题样式
  - 图表画布高度（300px）
  - 图表之间的间距
- [ ] 7.1.6 创建刷新按钮样式
- [ ] 7.1.7 创建自动刷新切换开关样式

### 7.2 添加动画CSS
- [ ] 7.2.1 为淡入动画添加@keyframes
- [ ] 7.2.2 为上滑动画添加@keyframes
- [ ] 7.2.3 为悬停状态添加过渡效果

### 7.3 响应式设计测试
- [ ] 7.3.1 在iPhone模拟器上测试（375px宽度）
- [ ] 7.3.2 在Android模拟器上测试（各种尺寸）
- [ ] 7.3.3 在iPad模拟器上测试（平板布局）
- [ ] 7.3.4 验证移动端2列网格正常工作
- [ ] 7.3.5 验证图表在所有屏幕尺寸上可读

---

## 阶段 8：质量保证

### 8.1 单元测试
- [ ] 8.1.1 测试日期范围计算工具（getYesterdayRange, getTodayRange等）
- [ ] 8.1.2 测试图表数据转换工具
- [ ] 8.1.3 测试角色检测逻辑
- [ ] 8.1.4 测试动画初始化
- [ ] 8.1.5 测试KPI计算逻辑（完成率、平均时间）

### 8.2 集成测试
- [ ] 8.2.1 测试物业员工视图（role_id=4）保持不变
- [ ] 8.2.2 测试维修员视图（role_id=3）保持不变
- [ ] 8.2.3 测试物业经理视图（role_id=2）正确显示
- [ ] 8.2.4 测试不同时间过滤器之间的切换（昨天、今天、本周、本月、自定义）
- [ ] 8.2.5 测试各种范围的自定义日期范围选择
- [ ] 8.2.6 测试手动刷新功能
- [ ] 8.2.7 测试自动刷新启用/禁用
- [ ] 8.2.8 测试所有5个图表能用真实数据正确渲染
- [ ] 8.2.9 测试两个排名正确显示
- [ ] 8.2.10 测试时间过滤器变更时数据更新

### 8.3 UI/UX测试
- [ ] 8.3.1 验证所有4个KPI卡片显示正确的值和格式
- [ ] 8.3.2 验证卡片动画播放流畅，延迟错开
- [ ] 8.3.3 验证图表在不同屏幕尺寸上正确渲染
- [ ] 8.3.4 验证图表提示框和图例正常工作
- [ ] 8.3.5 验证数据获取期间显示加载指示器
- [ ] 8.3.6 验证错误消息用户友好且可操作
- [ ] 8.3.7 测试可访问性的颜色对比度
- [ ] 8.3.8 测试图表上的触摸交互（如适用，缩放、平移）

### 8.4 性能测试
- [ ] 8.4.1 测量初始页面加载时间（应< 2秒）
- [ ] 8.4.2 测量时间过滤器变更响应时间（有缓存时应< 1秒）
- [ ] 8.4.3 使用大型数据集测试（1000+工单）
- [ ] 8.4.4 验证ECharts在大型数据集下的性能
- [ ] 8.4.5 验证自动刷新不会导致内存泄漏（测试30分钟以上）
- [ ] 8.4.6 监控网络请求并确保正确缓存

### 8.5 边缘情况测试
- [ ] 8.5.1 测试选定时间范围内工单为零的情况
- [ ] 8.5.2 测试缺少数据的情况（例如，没有已完成的工单）
- [ ] 8.5.3 测试极端值（非常大的数字、非常小的数字）
- [ ] 8.5.4 测试日期范围验证（start_date > end_date应显示错误）
- [ ] 8.5.5 测试网络故障场景
- [ ] 8.5.6 测试快速切换时间过滤器

---

## 阶段 9：后端API测试

### 9.1 测试新端点
- [ ] 9.1.1 使用各种日期范围测试GET /analytics/by-location
- [ ] 9.1.2 使用各种日期范围测试GET /analytics/by-floor
- [ ] 9.1.3 使用各种日期范围测试GET /analytics/employee-ranking
- [ ] 9.1.4 使用各种日期范围测试GET /analytics/responsible-party-ranking
- [ ] 9.1.5 验证所有端点返回正确的数据格式
- [ ] 9.1.6 验证所有端点遵守日期范围过滤器
- [ ] 9.1.7 验证所有端点正确缓存（Redis）

### 9.2 负载测试
- [ ] 9.2.1 测试对分析端点的并发请求
- [ ] 9.2.2 验证大型数据集的数据库查询性能
- [ ] 9.2.3 验证缓存命中率高（>80%）
- [ ] 9.2.4 监控高峰使用期间的数据库负载

---

## 阶段 10：文档和部署

### 10.1 文档
- [ ] 10.1.1 使用物业经理数据页面截图更新用户指南
- [ ] 10.1.2 记录所有5个时间过滤器选项及其行为
- [ ] 10.1.3 记录KPI卡片指标和计算方法
- [ ] 10.1.4 记录所有5种图表类型及其解释
- [ ] 10.1.5 记录排名算法
- [ ] 10.1.6 为新端点创建API文档
- [ ] 10.1.7 使用ECharts设置说明更新技术文档

### 10.2 代码审查和清理
- [ ] 10.2.1 审查所有代码的简洁性和对项目约定的遵守
- [ ] 10.2.2 删除所有调试console.log语句
- [ ] 10.2.3 验证与现有admin-manager页面没有代码重复
- [ ] 10.2.4 为复杂逻辑添加注释（图表初始化、日期计算）
- [ ] 10.2.5 确保一致的命名约定
- [ ] 10.2.6 验证所有错误处理都已到位

### 10.3 部署准备
- [ ] 10.3.1 验证现有功能没有破坏性变更
- [ ] 10.3.2 在开发环境中测试
- [ ] 10.3.3 在预发布环境中测试（如有）
- [ ] 10.3.4 准备回滚计划
- [ ] 10.3.5 创建部署检查清单
- [ ] 10.3.6 安排部署时间窗口

### 10.4 部署执行
- [ ] 10.4.1 首先部署后端API变更
- [ ] 10.4.2 验证后端端点在生产环境中正常工作
- [ ] 10.4.3 部署前端变更（小程序更新）
- [ ] 10.4.4 提交小程序供微信审核
- [ ] 10.4.5 在初始部署期间监控错误日志

---

## 阶段 11：部署后验证

### 11.1 生产验证
- [ ] 11.1.1 验证物业经理可以访问数据页面并看到全局分析
- [ ] 11.1.2 验证所有4个KPI卡片显示正确的值
- [ ] 11.1.3 验证所有5个图表正确渲染
- [ ] 11.1.4 验证两个排名正确显示
- [ ] 11.1.5 验证时间过滤器正常工作
- [ ] 11.1.6 验证物业员工视图仍然正常工作
- [ ] 11.1.7 验证维修工视图仍然正常工作
- [ ] 11.1.8 监控错误日志以发现任何意外问题
- [ ] 11.1.9 收集物业经理的初步用户反馈

### 11.2 性能监控
- [ ] 11.2.1 监控分析端点响应时间
- [ ] 11.2.2 监控Redis缓存命中率
- [ ] 11.2.3 监控生产环境中的页面加载时间
- [ ] 11.2.4 监控数据库查询性能
- [ ] 11.2.5 为慢查询或高错误率设置警报

### 11.3 用户培训和支持
- [ ] 11.3.1 培训物业经理使用新数据页面功能
- [ ] 11.3.2 创建快速参考指南
- [ ] 11.3.3 设置问题和建议的反馈渠道
- [ ] 11.3.4 监控用户采用率

---

## 依赖关系和并行化

### 顺序依赖
- 阶段0（依赖项设置）必须在阶段2-4之前完成
- 阶段1（后端）必须在阶段5（数据获取）之前完成
- 阶段5必须在阶段6（刷新）之前完成
- 阶段8（QA）依赖于阶段2-7的完成
- 阶段10（部署）依赖于阶段8的通过

### 可并行化工作
- 阶段1（后端）可以与阶段0、2、3并行运行
- 阶段3（KPI卡片）和阶段4（图表）可以并行开发
- 阶段8.1-8.3（不同测试类型）可以并行运行
- 阶段10.1（文档）可以在阶段8（测试）期间开始

### 预估工作量
- 阶段0：0.5天
- 阶段1：2天
- 阶段2-3：2天
- 阶段4：3天
- 阶段5-6：1天
- 阶段7：1天
- 阶段8：2天
- 阶段9：0.5天
- 阶段10：1天
- 阶段11：0.5天
- **总计：约13.5天**
