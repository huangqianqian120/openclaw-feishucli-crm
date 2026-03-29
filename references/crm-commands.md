# CRM 操作命令详解

> **前置条件：** 先阅读 [`../lark-shared/SKILL.md`](../../lark-shared/SKILL.md) 和 [`../lark-base/SKILL.md`](../../lark-base/SKILL.md)。
>
> **重要变更（v2.1）：** 本文档不再硬编码 base-token 和 table-id。所有命令中的 `<base_token>` 和 `<对应表_table_id>` 为占位符，执行前替换为配置初始化中缓存的值。参见 [`../SKILL.md`](../SKILL.md) 的「配置初始化」章节。

## 通用参数

所有命令共用：
```bash
--base-token <base_token>
--as user  # 默认，可省略
```

---

## +客户查询

### 查询所有客户

```bash
lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --limit 50
```

### 按条件筛选

**不要使用 `+view-set-filter`**，因为它需要修改共享视图权限（非管理员会 403），且必须指定 `--view-id`。

**正确方式**：用 `+record-list` 拉取全量数据后，AI 在本地按字段匹配过滤（行业、城市、规模、客户所有人等）。

### 输出字段顺序

查询结果以友好表格展示，优先展示以下字段：
1. 客户名称
2. 行业信息
3. 客户规模
4. 所在城市
5. 对接人姓名 / 对接人职位
6. 客户所有人
7. 最近跟进时间
8. 是否为公海客户

---

## +客户新增

### 完整示例

```bash
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --json '{
    "客户名称": "XX科技有限公司",
    "行业信息": "科技",
    "客户规模": "10-100",
    "所在城市": "上海",
    "对接人姓名": "张三",
    "对接人职位": "CTO",
    "对接人邮箱": "zhangsan@example.com",
    "联系电话": 13800138000,
    "客户所有人": [{"id": "ou_xxx"}]
  }'
```

### 最小必填示例

```bash
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --json '{"客户名称": "新客户公司"}'
```

### 选项值枚举

**行业信息**：电商、互联网、金融、医疗、餐饮、科技、制造

**客户规模**：<10、10-100、100-1000、1000-10000、>10000

**对接人职位**：CEO、CTO、IT负责人、人事负责人、产品负责人、运营负责人

### 注意事项

- 创建前先检查客户名称是否重复（系统有自动查重 formula 字段）
- 客户地址字段为 location 类型，需要传入 `{"lng": xx, "lat": xx}` 格式
- 成功后返回 record_id，用于后续关联商机和跟进记录

---

## +客户更新

```bash
# 更新单个字段
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --record-id recXXXXXX \
  --json '{"所在城市": "深圳"}'

# 更新多个字段
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --record-id recXXXXXX \
  --json '{"行业信息": "金融", "客户规模": ">10000", "对接人姓名": "李四"}'
```

---

## +商机查询

### 查询所有商机

```bash
lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --limit 50
```

### 按条件筛选

与客户查询相同，使用 `+record-list` 拉取后 AI 本地过滤。不要使用 `+view-set-filter`。

**跟进阶段枚举**：待接触、初步提议、方案报价、协商议价、丢失客户、赢单

### 输出字段

1. 机会名称（formula 自动生成）
2. 客户名称（关联显示，需查客户管理表解析 record_id）
3. 跟进阶段
4. 业务价值
5. 预计交易日期
6. 跟进销售人员
7. 销售区域（lookup）

---

## +商机新增

### 操作步骤

1. 先通过 `+客户查询` 获取目标客户的 record_id
2. （可选）通过 `+销售团队查询` 获取销售人员的 ou_id
3. 执行新增

### 完整示例

```bash
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --json '{
    "客户名称": [{"id": "recXXXXXX"}],
    "跟进阶段": "待接触",
    "业务价值": 150000,
    "商机描述": "客户对产品很感兴趣，初步沟通",
    "预计交易日期": "2026-06-30 00:00:00",
    "跟进销售人员": [{"id": "ou_xxx"}]
  }'
```

### 关键字段说明

- **客户名称**：link 类型，必须传入客户管理表的 record_id（不是客户名称文本）
- **跟进阶段**：如不指定，默认可留空
- **业务价值**：数字类型，不带千分位
- **预计交易日期**：datetime 格式 "YYYY-MM-DD HH:mm:ss"

---

## +商机更新

### 更新阶段

```bash
# 推进到下一阶段
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --record-id recXXXXXX \
  --json '{"跟进阶段": "协商议价"}'

# 标记赢单
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --record-id recXXXXXX \
  --json '{"跟进阶段": "赢单"}'

# 标记丢单（需填写原因）
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --record-id recXXXXXX \
  --json '{"跟进阶段": "丢失客户", "丢单原因": ["超客户预算"], "丢单原因备注": "客户预算上限50万"}'
```

---

## +跟进记录

### 操作步骤

1. 先通过 `+客户查询` 获取目标客户的 record_id
2. 添加跟进记录

### 完整示例

```bash
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户跟进记录_table_id> \
  --json '{
    "客户名称": [{"id": "recXXXXXX"}],
    "跟进内容": "电话沟通客户需求，客户对方案很感兴趣",
    "跟进形式": "电话沟通",
    "跟进人员": [{"id": "ou_xxx"}],
    "陪同人员": [{"id": "ou_yyy"}]
  }'
```

### 简化示例

```bash
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户跟进记录_table_id> \
  --json '{"客户名称": [{"id": "recXXXXXX"}], "跟进内容": "拜访客户讨论方案"}'
```

**跟进形式枚举**：现场拜访、电话沟通

### 注意事项

- 跟进时间为 `created_at` 类型，自动记录创建时间，无需手动填写
- 如需上传照片附件，先创建记录获取 record_id，再通过 `+record-upload-attachment` 上传

---

## +合同查询

### 查询所有合同

```bash
lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <合同管理_table_id> \
  --limit 50
```

### 输出字段

1. 合同编号
2. 客户名称（关联显示，需查客户管理表解析 record_id）
3. 商机名称（关联显示，需查商机管理表解析 record_id）
4. 签约日期
5. 合同金额
6. 签约人员（lookup）
7. 合同录入

---

## +合同新增

### 操作步骤

1. 先获取关联的商机 record_id（通过 `+商机查询`）
2. 先获取关联的客户 record_id（通过 `+客户查询`）
3. 新增合同

### 完整示例

```bash
lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <合同管理_table_id> \
  --json '{
    "合同编号": "20260328-001",
    "商机名称": [{"id": "recXXXXXX"}],
    "客户名称": [{"id": "recXXXXXX"}],
    "签约日期": "2026-03-28 00:00:00",
    "合同金额": 500000,
    "合同录入": [{"id": "ou_xxx"}]
  }'
```

---

## +记录删除

删除记录需要 `--yes` 参数确认，否则会报 `unsafe_operation_blocked` 错误。

```bash
lark-cli base +record-delete \
  --base-token <base_token> \
  --table-id <table_id> \
  --record-id recXXXXXX \
  --yes
```

**注意**：
- 删除是不可逆操作，执行前必须确认用户意图
- 如果有关联记录（如客户关联了商机），可能需要先删除关联记录

---

## +销售团队查询

```bash
lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <销售人员管理_table_id> \
  --limit 50
```

### 输出字段

1. 销售姓名（formula 自动生成）
2. 销售区域
3. 销售 Leader

### 销售人员 ou_id 获取

用于写入 user 字段时的 `ou_id`。从查询结果的「销售」字段（人员类型）中提取 `id` 值。

> **注意**：不要硬编码 ou_id，人员变动后 ID 可能失效。每次需要时从查询结果中实时获取。
