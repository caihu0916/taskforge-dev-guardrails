---
AIGC:
  ContentProducer: '001191110102MAD55U9H0F10002'
  ContentPropagator: '001191110102MAD55U9H0F10002'
  Label: '1'
  ProduceID: 'f27366ec-b4c7-40b1-b26f-8333a50d471d'
  PropagateID: 'f27366ec-b4c7-40b1-b26f-8333a50d471d'
  ReservedCode1: '500cf4e5-88fe-458c-9c72-c2c4aed4fd34'
  ReservedCode2: '500cf4e5-88fe-458c-9c72-c2c4aed4fd34'
---

# TaskForge 开发护栏

> 基于 534+ 项历史错误的根因分析，提炼的 10 条强制开发规则。预防的成本是治疗的 1/10 到 1/50。

## 这是什么？

一个 [TeleAgent 技能](https://github.com/teleagent)，在开发 TaskForge / TaskOS 项目时自动加载，阻止常见失误模式。

## 十条强制规则

| # | 规则 | 触发场景 | 事故代价 |
|---|------|----------|----------|
| 1 | 批量操作先试跑再全量 | 修改 > 5 个文件 | AST 迁移损坏 282 文件 + 11,011 编译错误，恢复 6.5h |
| 2 | CI 门禁必须存在 | 每次提交 | 818 个 TS 错误合入 main |
| 3 | 前后端共享接口契约 | 前后端接口变更 | E-commerce 13 端点 404，427 个孤立端点 |
| 4 | 模块完成度可验证 | 引擎模块新增/修改 | 4 个引擎导出断裂，2 个目录缺失 |
| 5 | 安全默认值必须安全 | 密钥/密码/CORS/认证 | 5 CRITICAL + 11 HIGH 安全漏洞 |
| 6 | 测试是开发的一部分 | 新功能开发 | 2% 覆盖率，安全修复无法验证回归 |
| 7 | 批量修改必须原子性 | docker-compose/批量脚本 | 8 文件功能崩溃，全部回滚 |
| 8 | 新技术先 POC 验证 | 新工具/API 引入 | 正则替换 JSX 全返工，ky.extendRequest 不存在 |
| 9 | 敏感数据与代码隔离 | .env/数据库/凭证 | 3.4GB 数据库差点打包分发 |
| 10 | 代码模式强制一致 | 引擎/场景开发 | 3 种 health 格式，重复 service 层 |

## 安装

### 方式 1：下载 .skill 文件

从 [Releases](../../releases) 下载 `taskforge-dev-guardrails.skill`，解压到 TeleAgent 的 `skills/` 目录。

### 方式 2：手动安装

```bash
# 克隆到 TeleAgent 技能目录
git clone https://github.com/caihu0916/taskforge-dev-guardrails.git \
  ~/.config/TeleAgent/skills/taskforge-dev-guardrails
```

### 方式 3：复制文件

将本仓库的文件复制到 TeleAgent 技能目录：

```
skills/taskforge-dev-guardrails/
├── SKILL.md                  # 核心规则（必须）
└── references/
    └── casebook.md           # 27 个历史事故详解
```

## 自动触发场景

涉及 TaskForge 项目的任何开发任务都会自动加载，特别是：

- 修改超过 5 个文件
- 安全相关修改（密钥/密码/CORS/认证）
- 前后端接口变更
- 批量脚本操作
- 新技术/新工具引入
- 敏感数据操作（.env/数据库/凭证）
- docker-compose 修改
- 引擎模块新增或修改

## 执行检查清单

```
□ 本次修改涉及几个文件？（>5 → 触发规则 1、7）
□ 是否涉及安全配置？（→ 触发规则 5、9）
□ 是否修改前后端接口？（→ 触发规则 3）
□ 是否引入新技术/新 API？（→ 触发规则 8）
□ 是否涉及引擎模块？（→ 触发规则 4、10）
□ 是否修改 docker-compose？（→ 触发规则 7）
□ 修改后是否验证？（→ tsc/pytest/docker-compose config）
□ 是否需要 git commit 分批？（→ 规则 7）
```

## 事故案例库

详见 [references/casebook.md](references/casebook.md)，包含每条规则对应的历史事故详情、根因分析和代价量化。

## License

MIT
