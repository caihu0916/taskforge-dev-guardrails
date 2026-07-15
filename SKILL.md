---
name: taskforge-dev-guardrails
description: >
  TaskForge/TaskOS 项目开发护栏。基于 534+ 项历史错误的根因分析，提供 10 条强制开发规则，
  在编写代码、批量修改、安全配置、前后端对接等场景自动检查并阻止常见失误模式。
  触发场景：(1) 修改超过 5 个文件 (2) 安全相关修改（密钥/密码/CORS/认证）
  (3) 前后端接口变更 (4) 批量脚本操作 (5) 新技术/新工具引入
  (6) 敏感数据操作（.env/数据库/凭证）(7) docker-compose 修改
  (8) 引擎模块新增或修改 (9) 任何涉及 TaskForge 项目的开发任务
name_cn: TaskForge 开发护栏
description_cn: 基于 534+ 项历史错误提炼的 10 条强制开发规则，自动阻止批量破坏、安全漏洞、前后端断链等常见失误
create_source: super-agent-skill-creator
---

# TaskForge 开发护栏

## 核心原则

预防的成本是治疗的 1/10 到 1/50。每条规则背后都是真实的事故代价。

## 十条强制规则

### 规则 1：批量操作必须先试跑再全量

修改 > 5 个文件时：

1. 先在 3-5 个文件上试跑
2. 试跑后执行 `tsc --noEmit` / `pytest` / `docker-compose config` 确认 0 新错误
3. 人工 diff 抽查至少 3 个文件
4. 通过后再全量
5. 禁止用 Python/Shell 正则做 .ts/.tsx/.js/.jsx 代码替换，必须用 AST 工具

**代价参考**: AST 迁移脚本 v3 损坏 282 文件 + 11,011 编译错误，恢复 6.5 小时。

### 规则 2：CI 门禁是必须基础设施

每次提交必须通过：

- `ruff check` 零错误
- `pytest --cov-fail-under=30` 通过
- `npx tsc --noEmit` 零错误
- 安全扫描（trivy/bandit）无 CRITICAL/HIGH

PR 有新增错误时自动阻止合并。

### 规则 3：前后端必须有共享接口契约

- 后端 FastAPI 自动生成 OpenAPI schema
- 前端通过 openapi-typescript 自动生成 TypeScript 类型
- 新增后端端点必须有对应前端调用或标记为"API-only"
- 禁止前端直接操作数据库，所有 HTTP 走 ky singleton

### 规则 4：模块完成度必须可验证

每个引擎模块必须满足：

- `__init__.py` 有有效导出（零导出 = P0 阻断）
- `get_xxx_engine()` singleton accessor 存在
- `CROSS_ENGINE_FK_REGISTRY` 注册完整
- Health 端点基于实际检查，禁止硬编码 `True`

### 规则 5：安全默认值必须安全

- `.env.example` 仅含占位符，绝不含真实值
- docker-compose 密码通过环境变量注入，禁止硬编码
- 加密密钥为空时生产环境拒绝启动
- 容器禁止以 root 运行
- CORS 域名禁止硬编码在代码默认值中
- 加密口令禁止硬编码默认值

### 规则 6：测试是开发的一部分

- 新功能必须有对应测试（TDD: RED → GREEN → REFACTOR）
- 测试隔离：每个 fixture 必须清理环境变量/单例/数据库
- 测试先行但实现未落地时标记 `@pytest.mark.skip("pending implementation")`
- 前端必须安装测试框架（vitest + testing-library）
- 覆盖率门禁: 后端 >= 30%，核心模块 >= 80%

### 规则 7：批量修改必须原子性

- 修改 > 5 个文件时，先写临时文件，验证后再替换原文件
- docker-compose.yml 修改必须用完整 service 替换，不能片段拼接
- 修改前必须验证 API 是否存在（如 `ky.extendRequest()` 不存在）
- 修改后必须做功能回归验证（至少 TS 编译 + docker-compose config）
- 每个批次修改后 git commit，可精确回退

### 规则 8：新技术引入前必须 POC 验证

- 新技术/新工具引入时，先写最小 POC（<= 30 分钟）
- POC 必须覆盖至少 3 种边缘情况
- 对不确定的 API，先读官方文档确认存在性再使用
- POC 通过后才投入正式开发

### 规则 9：敏感数据必须与代码库隔离

- `.gitignore` 必须包含 `.env`、`data/*.db`、`data/secrets/`
- 打包前必须运行脱敏脚本：删除数据库、创建 .env.example、检查凭证文件
- 开发数据库文件不进入版本控制
- 根目录仅保留必要文件，临时文件归入 `_archive/`
- 收款码路径正确指向 public 目录

### 规则 10：代码模式必须强制一致

- 引擎模块统一模式: `engine.py`(singleton + stats) + `__init__.py`(导出) + `scenario.py`(health + metrics)
- Health 端点统一格式: `{scenario, tables_ok, engine_ok}`
- stats() 签名统一: `stats(days: int = 30, **kwargs)`
- 新增场景必须有完整 Tab 页面宿主 + CRUD 交互
- 消除重复 service 层，保留 `scenario-xxx.ts` 为权威源

## 执行检查清单

在执行任何 TaskForge 开发任务前，确认以下事项：

```
□ 本次修改涉及几个文件？（>5 → 触发规则 1、7）
□ 是否涉及安全配置？（→ 触发规则 5、9）
□ 是否修改前后端接口？（→ 触发规则 3）
□ 是否引入新技术/新 API？（→ 触发规则 8）
□ 是否涉及引擎模块？（→ 触发规则 4、10）
□ 是否修改 docker-compose？（→ 触发规则 7，必须完整 service 替换）
□ 修改后是否验证？（→ tsc/pytest/docker-compose config）
□ 是否需要 git commit 分批？（→ 规则 7，每批可回退）
```

## 事故案例库

详见 [references/casebook.md](references/casebook.md)，包含每条规则对应的历史事故详情、根因分析和代价量化。遇到类似场景时应主动查阅。
