# CRM 数据分析与报表 DSL 模板

> 执行 `lark-cli base +data-query` 前使用本模板。

## 通用参数

```bash
~/.local/bin/lark-cli base +data-query \
  --base-token <base_token> \
  --dsl '<DSL_JSON>'
```

## 表 ID 占位符

- `<商机管理_table_id>` → 商机管理表
- `<客户管理_table_id>` → 客户管理表
- `<合同管理_table_id>` → 合同管理表
- `<销售人员管理_table_id>` → 销售人员管理表
- `<客户跟进记录_table_id>` → 客户跟进记录表

---

## 1. 销售漏斗（按阶段统计）

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<商机管理_table_id>" }
  },
  "dimensions": [
    { "field_name": "跟进阶段", "alias": "dim_stage" }
  ],
  "measures": [
    { "field_name": "跟进阶段", "aggregation": "count_all", "alias": "opp_count" },
    { "field_name": "业务价值", "aggregation": "sum", "alias": "total_value" }
  ],
  "sort": [
    { "field_name": "total_value", "order": "desc" }
  ],
  "pagination": { "limit": 100 },
  "shaper": { "format": "flat" }
}
```

输出示例：

| 跟进阶段 | 商机数 | 业务价值总额 |
|---------|--------|------------|
| 赢单 | 3 | 8,195,261 |
| 协商议价 | 1 | 1,750,000 |

---

## 2. 赢单率统计

需要两个查询：

**查询赢单数**：
```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<商机管理_table_id>" }
  },
  "dimensions": [],
  "measures": [
    { "field_name": "跟进阶段", "aggregation": "count_all", "alias": "win_count" }
  ],
  "filters": {
    "type": 1,
    "conjunction": "and",
    "conditions": [
      { "field_name": "跟进阶段", "operator": "is", "value": ["赢单"] }
    ]
  },
  "pagination": { "limit": 1 },
  "shaper": { "format": "flat" }
}
```

**查询总商机数**：
```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<商机管理_table_id>" }
  },
  "dimensions": [],
  "measures": [
    { "field_name": "跟进阶段", "aggregation": "count_all", "alias": "total_count" }
  ],
  "pagination": { "limit": 1 },
  "shaper": { "format": "flat" }
}
```

**赢单率** = 赢单数 / 总商机数

---

## 3. 按销售人员统计业绩

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<商机管理_table_id>" }
  },
  "dimensions": [
    { "field_name": "跟进销售人员", "alias": "dim_sales" }
  ],
  "measures": [
    { "field_name": "跟进销售人员", "aggregation": "count_all", "alias": "opp_count" },
    { "field_name": "业务价值", "aggregation": "sum", "alias": "total_value" }
  ],
  "sort": [
    { "field_name": "total_value", "order": "desc" }
  ],
  "pagination": { "limit": 100 },
  "shaper": { "format": "flat" }
}
```

---

## 4. 按销售区域统计

> 注意：销售区域是 lookup 字段，`+data-query` 不支持。两步方案：

**步骤 1**：查询销售人员管理表，建立「跟进销售人员 open_id → 销售区域」映射

**步骤 2**：查询商机管理表全部记录，本地按区域聚合

---

## 5. 合同金额汇总

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<合同管理_table_id>" }
  },
  "dimensions": [],
  "measures": [
    { "field_name": "合同金额", "aggregation": "sum", "alias": "total_contract" },
    { "field_name": "合同金额", "aggregation": "count_all", "alias": "contract_count" },
    { "field_name": "合同金额", "aggregation": "avg", "alias": "avg_contract" },
    { "field_name": "合同金额", "aggregation": "max", "alias": "max_contract" }
  ],
  "pagination": { "limit": 1 },
  "shaper": { "format": "flat" }
}
```

输出示例：

| 合同总额 | 合同数量 | 平均合同金额 | 最大合同金额 |
|---------|---------|------------|------------|
| 9,200,232 | 5 | 1,840,046 | 5,000,232 |

---

## 6. 按行业分布

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<客户管理_table_id>" }
  },
  "dimensions": [
    { "field_name": "行业信息", "alias": "dim_industry" }
  ],
  "measures": [
    { "field_name": "行业信息", "aggregation": "count_all", "alias": "customer_count" }
  ],
  "sort": [
    { "field_name": "customer_count", "order": "desc" }
  ],
  "pagination": { "limit": 100 },
  "shaper": { "format": "flat" }
}
```

---

## 7. 按客户规模分布

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<客户管理_table_id>" }
  },
  "dimensions": [
    { "field_name": "客户规模", "alias": "dim_scale" }
  ],
  "measures": [
    { "field_name": "客户规模", "aggregation": "count_all", "alias": "customer_count" }
  ],
  "sort": [
    { "field_name": "customer_count", "order": "desc" }
  ],
  "pagination": { "limit": 100 },
  "shaper": { "format": "flat" }
}
```

---

## 8. 按城市分布

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<客户管理_table_id>" }
  },
  "dimensions": [
    { "field_name": "所在城市", "alias": "dim_city" }
  ],
  "measures": [
    { "field_name": "所在城市", "aggregation": "count_all", "alias": "customer_count" }
  ],
  "filters": {
    "type": 1,
    "conjunction": "and",
    "conditions": [
      { "field_name": "所在城市", "operator": "isNotEmpty", "value": [] }
    ]
  },
  "sort": [
    { "field_name": "customer_count", "order": "desc" }
  ],
  "pagination": { "limit": 100 },
  "shaper": { "format": "flat" }
}
```

---

## 9. 丢单原因分析

```json
{
  "datasource": {
    "type": "table",
    "table": { "tableId": "<商机管理_table_id>" }
  },
  "dimensions": [
    { "field_name": "丢单原因", "alias": "dim_loss_reason" }
  ],
  "measures": [
    { "field_name": "丢单原因", "aggregation": "count_all", "alias": "loss_count" },
    { "field_name": "业务价值", "aggregation": "sum", "alias": "loss_value" }
  ],
  "filters": {
    "type": 1,
    "conjunction": "and",
    "conditions": [
      { "field_name": "跟进阶段", "operator": "is", "value": ["丢失客户"] }
    ]
  },
  "sort": [
    { "field_name": "loss_count", "order": "desc" }
  ],
  "pagination": { "limit": 100 },
  "shaper": { "format": "flat" }
}
```

---

## 注意事项

1. **data-query 权限**：需要 base 完全访问权限（FA）。无 FA 权限时用 `+record-list` 拉取后本地聚合。
2. **不支持的字段**：formula / lookup / 关联字段不能用作 dimensions/measures/filters/sort。
3. **字段名精确匹配**：必须与表字段名完全一致。
4. **shaper 必须填**：`"shaper": {"format": "flat"}"` 每个查询都要加。
5. **金额返回格式**：聚合结果可能是带 `¥` 符号的字符串。
6. **pagination.limit**：最大 5000。
