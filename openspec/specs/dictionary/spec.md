# dictionary Specification

## Purpose
TBD - created by archiving change add-dictionary-management. Update Purpose after archive.
## Requirements
### Requirement: Dictionary Data Storage

系统 SHALL 提供 `dictionaries` 云数据库集合存储字典数据。

每个字典文档包含：
- `dict_key`: 字典唯一标识（如 `floor`, `order_category`）
- `dict_name`: 字典显示名称
- `description`: 字典描述
- `items`: 字典项数组，每项包含 `value`, `label`, `sort`, `enabled`
- `is_system`: 是否系统字典（系统字典不可删除）
- `created_at`, `updated_at`: 时间戳

#### Scenario: 字典数据结构

- **GIVEN** 系统初始化完成
- **WHEN** 查询 dictionaries 集合
- **THEN** 应包含预置的系统字典（floor, order_category, responsible_party, department）
- **AND** 每个字典的 items 数组包含有效的选项数据

---

### Requirement: Dictionary Cloud Function

系统 SHALL 提供 `dictionaryManager` 云函数管理字典数据。

支持的 actions：
- `list`: 获取所有字典组列表
- `get`: 获取单个字典（含字典项）
- `getBatch`: 批量获取多个字典
- `create`: 创建字典组（仅管理员）
- `update`: 更新字典组/字典项（仅管理员）
- `delete`: 删除字典组（仅管理员，系统字典不可删）

#### Scenario: 获取字典列表

- **GIVEN** 用户已登录
- **WHEN** 调用 dictionaryManager 云函数，action='list'
- **THEN** 返回所有字典组列表
- **AND** 每个字典包含 dict_key, dict_name, 字典项数量

#### Scenario: 获取单个字典

- **GIVEN** 用户已登录
- **WHEN** 调用 dictionaryManager 云函数，action='get', dict_key='floor'
- **THEN** 返回完整的字典数据，包含所有启用的字典项
- **AND** 字典项按 sort 字段排序

#### Scenario: 批量获取字典

- **GIVEN** 用户已登录
- **WHEN** 调用 dictionaryManager 云函数，action='getBatch', keys=['floor', 'order_category']
- **THEN** 返回多个字典的数据
- **AND** 每个字典包含其启用的字典项

#### Scenario: 创建字典组（管理员）

- **GIVEN** 用户为管理员（role_id=1）
- **WHEN** 调用 dictionaryManager 云函数，action='create', data={dict_key, dict_name, items}
- **THEN** 创建新字典并返回成功

#### Scenario: 创建字典组（非管理员）

- **GIVEN** 用户不是管理员
- **WHEN** 调用 dictionaryManager 云函数，action='create'
- **THEN** 返回权限错误

#### Scenario: 更新字典项

- **GIVEN** 用户为管理员
- **WHEN** 调用 dictionaryManager 云函数，action='update', dict_key='floor', items=[...]
- **THEN** 更新字典项并返回成功

#### Scenario: 删除系统字典

- **GIVEN** 用户为管理员
- **WHEN** 尝试删除 is_system=true 的字典
- **THEN** 返回错误，系统字典不可删除

---

### Requirement: Dictionary Frontend Service

前端 SHALL 提供 `services/dictionary.js` 服务获取字典数据。

功能：
- `getDictionary(dictKey)`: 获取单个字典
- `getDictionaries(dictKeys)`: 批量获取
- `getOptions(dictKey)`: 返回选项数组（用于 picker）
- 本地缓存（5分钟有效）
- 加载失败时返回硬编码兜底值

#### Scenario: 获取字典选项（成功）

- **GIVEN** 云函数正常运行
- **WHEN** 调用 dictionary.getOptions('floor')
- **THEN** 返回楼层选项数组 ['1楼', '2楼', ...]

#### Scenario: 获取字典选项（云函数失败）

- **GIVEN** 云函数调用失败或超时
- **WHEN** 调用 dictionary.getOptions('floor')
- **THEN** 返回硬编码兜底值 ['1楼', '2楼', '3楼', '4楼', '5楼', 'B1', 'B2']
- **AND** 页面正常显示，不报错

#### Scenario: 字典缓存

- **GIVEN** 已成功获取过字典数据
- **WHEN** 5分钟内再次调用 getOptions
- **THEN** 直接返回缓存数据，不调用云函数

---

### Requirement: Dictionary Admin Pages

系统 SHALL 为管理员提供字典管理页面。

页面结构：
- `pages/admin/dict/index`: 字典组列表
- `pages/admin/dict/items`: 字典项管理

#### Scenario: 字典组列表页

- **GIVEN** 管理员进入字典管理页面
- **WHEN** 页面加载完成
- **THEN** 显示所有字典组（名称、键名、项数）
- **AND** 可点击进入字典项管理

#### Scenario: 新增字典组

- **GIVEN** 管理员在字典列表页
- **WHEN** 点击新增按钮并填写表单
- **THEN** 创建新字典组并刷新列表

#### Scenario: 字典项管理页

- **GIVEN** 管理员进入某字典的项管理页
- **WHEN** 页面加载完成
- **THEN** 显示该字典的所有项（值、标签、排序、状态）
- **AND** 可新增/编辑/删除/排序字典项

#### Scenario: 启用停用字典项

- **GIVEN** 管理员在字典项管理页
- **WHEN** 切换某项的启用状态
- **THEN** 更新字典项状态
- **AND** 停用的项不会在表单选项中显示

---

### Requirement: Form Page Integration

表单页面 SHALL 从字典服务获取选项数据。

涉及页面：
- `pages/property/submit`: 楼层、工单类别、责任方
- `pages/work-order-edit`: 工单类别、责任方
- `pages/admin/users/add`: 部门
- `pages/admin/users/edit`: 部门

#### Scenario: 提交工单页面加载字典

- **GIVEN** 用户进入提交工单页面
- **WHEN** 页面加载
- **THEN** 从字典服务获取楼层、类别、责任方选项
- **AND** 选项正确显示在对应的选择器中

#### Scenario: 字典加载失败不影响表单

- **GIVEN** 字典服务加载失败
- **WHEN** 用户进入提交工单页面
- **THEN** 使用兜底选项值
- **AND** 表单可正常使用

---

### Requirement: Dictionary Initialization

系统 SHALL 在数据库初始化时预置系统字典。

预置字典：
- `floor`: 楼层 - 1楼, 2楼, 3楼, 4楼, 5楼, B1, B2
- `order_category`: 工单类别 - 电梯维修, 水电维修, 消防维修, 空调维修, 其他
- `responsible_party`: 责任方 - 信泰物业, 业主, 第三方
- `department`: 部门 - 行政部, 信泰物业, 工程总包, 供应商

#### Scenario: 初始化系统字典

- **GIVEN** 执行 initDatabase 云函数，action='initDictionaries'
- **WHEN** 初始化完成
- **THEN** dictionaries 集合包含 4 个系统字典
- **AND** 每个字典的 is_system=true

