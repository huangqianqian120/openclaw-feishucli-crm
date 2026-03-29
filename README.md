# openclaw-crm-skill: 飞书 CRM Skill (OpenClaw 版)

> 通过自然语言对话，直接管理飞书多维表格中的 CRM 系统。专为 OpenClaw 打造。

## 概述

这是一个 **OpenClaw Skill**，将飞书多维表格 CRM 系统封装成对话式管理工具。

你不需要记住任何表名、字段名或命令参数，只需用自然语言描述意图，AI 会自动完成操作。

## 工作原理

```
用户自然语言
     ↓
意图路由（关键词匹配）→ 执行 lark-cli 命令
     ↓
配置自动发现（用户提供 URL → 自动解析）
     ↓
格式化输出给用户
```

## 安装

### 1. 安装 lark-cli

```bash
npm install -g @larksuite/cli
export PATH="$HOME/.local/bin:$PATH"  # 加到 ~/.zshrc 永久生效
```

### 2. 配置 lark-cli 认证

```bash
# 初始化（会输出授权链接，用飞书账号扫码授权）
lark-cli config init --new

# 切换为用户身份（推荐，获得完整权限）
lark-cli config default-as user

# 授权多维表格权限
lark-cli auth login --domain base --recommend
```

### 3. 安装 Skill

```bash
# 方法一：从 clawhub 安装
npx clawhub@latest install lark-crm

# 方法二：手动 clone
git clone https://github.com/huangqianqian120/openclaw-feishucli-crm.git \
  ~/.local/lib/node_modules/openclaw/skills/lark-crm
```

### 4. 重启 OpenClaw Gateway

```bash
openclaw gateway restart
```

## 快速开始

### 首次使用

提供你的飞书 CRM 多维表格 URL：

```
"https://xxx.feishu.cn/base/XXXXXX?table=tblXXX&view=vewXXX"
```

系统会自动：
1. 从 URL 解析 `base_token`
2. 调用 `+table-list` 发现所有表结构
3. 保存到 `~/.openclaw/workspace/lark-crm-config.json`

### 对话示例

```
"帮我查一下所有客户"
"添加一个新客户：北京科技公司，互联网行业，北京"
"看看销售漏斗"
"按销售人员统计业绩"
"查一下赢单的商机"
"给这个客户添加一条跟进记录：电话沟通，客户很有意向"
```

## 功能列表

| 类别 | 支持的操作 |
|------|----------|
| 客户 | 查询、新增、更新、删除 |
| 商机 | 查询、新增、更新阶段（赢单/丢单/推进） |
| 跟进记录 | 添加、查看 |
| 合同 | 查询、新增 |
| 销售报表 | 漏斗统计、业绩排名、行业分布、合同汇总 |
| 销售团队 | 查看团队成员、区域划分 |

## 表结构

| 表名 | 用途 |
|------|------|
| 客户信息 | 客户基本信息和联系人 |
| 商机管理 | 销售商机和阶段追踪 |
| 客户跟进记录 | 跟进活动日志 |
| 合同管理 | 合同和签约信息 |
| 销售人员管理 | 销售团队和区域 |
| 市场活动管理 | 市场活动 |
| 线索管理 | 销售线索 |
| 销售订单 | 订单管理 |
| 订单产品明细表 | 订单产品 |
| 产品管理 | 产品目录 |
| 销售目标管理 | 销售目标 |

## 配置说明

配置文件位置：`~/.openclaw/workspace/lark-crm-config.json`

```json
{
  "base_token": "xxxxxx",
  "tables": {
    "客户信息": "tblxxx",
    "商机管理": "tblyyy",
    ...
  },
  "updated_at": "2026-03-29"
}
```

**切换 CRM 表格**：提供新 URL，系统自动更新配置。

## 文件结构

```
lark-crm/
├── SKILL.md                    # 主入口（OpenClaw skill 格式）
└── references/
    └── crm-analytics.md        # 数据分析 DSL 模板
```

## 依赖

| 依赖 | 说明 |
|------|------|
| `@larksuite/cli` | 飞书官方 CLI 工具 |
| `lark-cli auth` | 用户身份授权 |
| OpenClaw | 运行环境 |

## 注意事项

1. **lark-cli PATH**：确保 `~/.local/bin` 在 PATH 中
2. **用户身份**：推荐用 `lark-cli config default-as user` 获得完整权限
3. **权限要求**：多维表格需对应用授予「可编辑」权限
4. **关联字段**：客户名、商机名等 link 字段返回 record_id，AI 会自动解析为可读名称

## License

MIT
