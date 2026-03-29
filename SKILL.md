# 飞书 CRM 系统

> 飞书多维表格 CRM 管理 skill，直接调用 `lark-cli` 执行操作。

## 快速开始

**首次使用必须先配置：**

用户提供飞书 CRM 多维表格 URL，格式如：
```
https://xxx.feishu.cn/base/XXXXXX?table=tblXXX&view=vewXXX
```

系统会自动：
1. 从 URL 提取 `base_token`
2. 调用 `+table-list` 发现所有表的 `table_id`
3. 缓存到 `~/.openclaw/workspace/lark-crm-config.json`

配置完成后，后续所有命令都会自动使用缓存的配置。

---

## 意图路由

```
用户输入
├─ 客户相关（客户/联系人/对接人）
│   ├─ 查询/查看/搜索/找 → 执行 +客户查询
│   ├─ 新增/添加/创建/录入 → 执行 +客户新增
│   └─ 更新/修改/编辑 → 执行 +客户更新
├─ 商机相关（商机/销售漏斗/赢单/丢单/交易/机会）
│   ├─ 查询/查看/漏斗 → 执行 +商机查询
│   ├─ 新增/创建 → 执行 +商机新增
│   └─ 更新阶段/状态 → 执行 +商机更新
├─ 跟进相关（跟进/拜访/电话/沟通/回访）
│   └─ 记录/添加/写 → 执行 +跟进记录
├─ 合同相关（合同/签约/协议）
│   ├─ 查询/查看 → 执行 +合同查询
│   └─ 新增/录入 → 执行 +合同新增
├─ 删除相关（删除/移除/不要了）
│   └─ 执行 +记录删除（需 --yes 确认）
├─ 报表分析（报表/统计/分析/排名/总计/汇总/业绩/漏斗）
│   └─ 执行 +销售报表（读取 references/crm-analytics.md）
└─ 销售团队（销售团队/销售人员/销售区域）
    └─ 执行 +销售团队查询
```

---

## 配置初始化

### 检查配置

```bash
cat ~/.openclaw/workspace/lark-crm-config.json
```

### 初始化流程

1. 用户提供飞书 CRM 多维表格 URL
2. 从 URL 解析 `base_token`（`/base/` 后面的字符串）
3. 执行 `lark-cli base +table-list --base-token <base_token>` 获取所有表
4. 按表名关键字匹配 table_id：
   - 客户管理 → 客户信息
   - 商机管理 → 销售商机
   - 客户跟进记录 → 跟进记录
   - 合同管理 → 合同信息
   - 销售人员管理 → 销售团队
5. 保存到 `~/.openclaw/workspace/lark-crm-config.json`

### 配置格式

```json
{
  "base_token": "xxxxxx",
  "tables": {
    "客户管理": "tblxxx",
    "商机管理": "tblyyy",
    "客户跟进记录": "tblzzz",
    "合同管理": "tblwww",
    "销售人员管理": "tblvvv"
  },
  "updated_at": "2026-03-29"
}
```

---

## CRM 操作命令

> 执行命令前确保已配置好 base_token 和 table_id。
> `lark-cli` 命令需要加路径（若 PATH 未设置）：`~/.local/bin/lark-cli`

### +客户查询

```bash
~/.local/bin/lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --limit 50
```

输出字段：客户名称、行业、规模、城市、对接人姓名/职位、客户所有人、最近跟进时间

### +客户新增

**可写字段**：

| 字段名 | 类型 | 必填 | 格式 |
|--------|------|------|------|
| 客户名称 | text | 是 | 字符串 |
| 行业信息 | select | 否 | 电商/互联网/金融/医疗/餐饮/科技/制造 |
| 客户规模 | select | 否 | <10/10-100/100-1000/1000-10000/>10000 |
| 所在城市 | text | 否 | 字符串 |
| 对接人姓名 | text | 否 | 字符串 |
| 对接人职位 | select | 否 | CEO/CTO/IT负责人/人事负责人/产品负责人/运营负责人 |
| 对接人邮箱 | text |否 | 邮箱字符串 |
| 联系电话 | number | 否 | 数字 |
| 客户所有人 | user | 否 | `[{"id":"ou_xxx"}]` |

```bash
~/.local/bin/lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --json '{"客户名称":"XX公司","行业信息":"互联网","所在城市":"北京"}'
```

### +客户更新

需要先获取 record_id，再更新：

```bash
~/.local/bin/lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户管理_table_id> \
  --record-id recXXXXXX \
  --json '{"所在城市":"深圳"}'
```

### +商机查询

```bash
~/.local/bin/lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --limit 50
```

**跟进阶段枚举**：待接触、初步提议、方案报价、协商议价、丢失客户、赢单

### +商机新增

需要先通过客户查询获取客户的 record_id：

```bash
~/.local/bin/lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --json '{
    "客户名称":[{"id":"recXXXXXX"}],
    "跟进阶段":"待接触",
    "业务价值":100000,
    "商机描述":"客户对方案很感兴趣"
  }'
```

### +商机更新

```bash
~/.local/bin/lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <商机管理_table_id> \
  --record-id recXXXXXX \
  --json '{"跟进阶段":"赢单"}'
```

### +跟进记录

```bash
~/.local/bin/lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <客户跟进记录_table_id> \
  --json '{
    "客户名称":[{"id":"recXXXXXX"}],
    "跟进内容":"拜访客户讨论方案",
    "跟进形式":"现场拜访"
  }'
```

**跟进形式枚举**：现场拜访、电话沟通

### +合同查询

```bash
~/.local/bin/lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <合同管理_table_id> \
  --limit 50
```

### +合同新增

```bash
~/.local/bin/lark-cli base +record-upsert \
  --base-token <base_token> \
  --table-id <合同管理_table_id> \
  --json '{
    "合同编号":"20260329-001",
    "客户名称":[{"id":"recXXXXXX"}],
    "签约日期":"2026-03-29 00:00:00",
    "合同金额":500000
  }'
```

### +记录删除

```bash
~/.local/bin/lark-cli base +record-delete \
  --base-token <base_token> \
  --table-id <table_id> \
  --record-id recXXXXXX \
  --yes
```

⚠️ 删除前必须确认用户意图！

### +销售团队查询

```bash
~/.local/bin/lark-cli base +record-list \
  --base-token <base_token> \
  --table-id <销售人员管理_table_id> \
  --limit 50
```

输出字段：销售姓名、销售区域、销售 Leader

---

## 销售报表 (+data-query)

执行 `+data-query` 前先读取 `references/crm-analytics.md` 获取 DSL 模板。

```bash
~/.local/bin/lark-cli base +data-query \
  --base-token <base_token> \
  --dsl '<DSL_JSON>'
```

**常用分析**：
- 销售漏斗（按阶段统计）
- 按销售人员统计业绩
- 按区域统计
- 合同金额汇总
- 行业/规模分布

> 注意：`+data-query` 需要 base 完全访问权限（FA）。无权限时用 `+record-list` 拉取后本地聚合。

---

## 重要注意事项

1. **link 字段**（客户名称、商机名称等）返回 record_id，不是可读名称。需要先查对应表建立 ID→名称映射。
2. **user 字段**需要 open_id（如 `ou_xxx`），通过 `+record-list` 已有记录中匹配或 `lark-cli contact +search` 查找。
3. **select 字段值**必须在枚举列表内，否则报错。
4. **公式/lookup字段**不可写，由系统自动计算。
5. **403 权限错误**需要 base 管理员授予编辑权限。
6. **删除必须加 --yes**，否则报 `unsafe_operation_blocked`。
7. **金额聚合结果**可能是带 `¥` 符号的字符串（如 `"¥9,215,261.00"`）。
