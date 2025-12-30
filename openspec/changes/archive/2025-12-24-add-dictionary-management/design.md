# Design: 字典管理功能

## Context

### 背景
项目当前的下拉选项（楼层、工单类别、责任方、部门）硬编码在多个位置：
- 前端页面 JS 文件
- 云函数常量
- 后端 Model ENUM

这导致：
1. 修改选项需要改多处代码并重新部署
2. 不同位置的选项值可能不一致
3. 管理员无法自助维护

### 约束
- 必须兼容历史数据，字典缺失时提供兜底默认值
- 必须遵循现有项目结构（云开发、角色权限）
- 只有管理员（role_id=1）可以管理字典

## Goals / Non-Goals

### Goals
- 管理员可通过小程序界面管理字典组和字典项
- 前端表单动态加载字典选项
- 字典缺失时使用硬编码兜底，不影响页面正常使用
- 字典数据带缓存，减少云函数调用

### Non-Goals
- 不修改优先级、状态、角色等与业务逻辑强绑定的枚举
- 不做多语言/国际化
- 不做字典版本控制/审计

## Decisions

### 1. 数据结构设计

**集合: `dictionaries`**
```javascript
{
  _id: string,              // 自动生成
  dict_key: string,         // 字典键，唯一标识，如 'floor', 'order_category'
  dict_name: string,        // 字典名称，如 '楼层', '工单类别'
  description: string,      // 描述
  items: [                  // 字典项数组
    {
      value: string,        // 值，如 '1楼', '电梯维修'
      label: string,        // 显示标签（可选，默认等于 value）
      sort: number,         // 排序号
      enabled: boolean,     // 是否启用
      extra: object         // 扩展字段（可选）
    }
  ],
  is_system: boolean,       // 是否系统字典（系统字典不可删除）
  created_at: Date,
  updated_at: Date
}
```

**理由**: 将字典项嵌入字典文档（而非独立集合），简化查询和缓存逻辑。单个字典的选项数量通常不超过 20 个，不会有性能问题。

### 2. 云函数设计

**云函数: `dictionaryManager`**

Actions:
- `list` - 获取所有字典组列表
- `get` - 获取单个字典（含所有字典项）
- `getBatch` - 批量获取多个字典（用于表单初始化）
- `create` - 创建字典组（管理员）
- `update` - 更新字典组/字典项（管理员）
- `delete` - 删除字典组（管理员，系统字典不可删）

### 3. 前端服务设计

**服务: `services/dictionary.js`**

```javascript
// 核心方法
getDictionary(dictKey)      // 获取单个字典，带缓存
getDictionaries(dictKeys)   // 批量获取，带缓存
getOptions(dictKey)         // 获取选项数组（用于 picker）
refreshCache(dictKey?)      // 刷新缓存

// 缓存策略
- 本地缓存 5 分钟
- 缓存失效时自动重新加载
- 加载失败时返回硬编码兜底值
```

**兜底值配置**:
```javascript
const FALLBACK_OPTIONS = {
  floor: ['1楼', '2楼', '3楼', '4楼', '5楼', 'B1', 'B2'],
  order_category: ['电梯维修', '水电维修', '消防维修', '空调维修', '其他'],
  responsible_party: ['信泰物业', '业主', '第三方'],
  department: ['行政部', '信泰物业', '工程总包', '供应商']
};
```

### 4. 页面结构

```
pages/admin/dict/
├── index.js/wxml/wxss/json    # 字典组列表
└── items.js/wxml/wxss/json    # 字典项管理
```

简化设计：不单独做 groupEdit 和 itemEdit 页面，使用弹窗形式编辑。

### 5. 管理员入口

在 `pages/admin/index` 管理员首页添加"字典管理"入口卡片。

## Risks / Trade-offs

| 风险 | 缓解措施 |
|-----|---------|
| 字典加载失败导致表单不可用 | 硬编码兜底值，即使云函数失败也能正常使用 |
| 缓存数据过期导致选项不一致 | 5 分钟缓存 + 管理员修改后手动刷新提示 |
| 字典项被删除导致历史数据显示异常 | 只做逻辑删除（enabled=false），保留历史值可识别 |

## Migration Plan

### 初始化数据
首次部署时，在 `initDatabase` 云函数中添加 `initDictionaries` action，预置 4 个系统字典：
- `floor` (楼层)
- `order_category` (工单类别)
- `responsible_party` (责任方)
- `department` (部门)

### 兼容性
- 前端改造后，优先从字典服务获取选项
- 字典服务返回失败时，使用本地硬编码兜底
- 不需要数据迁移，历史工单数据不受影响

## Open Questions

1. ~~是否需要字典项的扩展字段（extra）？~~ 预留，暂不使用
2. ~~是否需要支持字典项颜色配置？~~ 暂不支持，优先级/状态颜色保持硬编码
