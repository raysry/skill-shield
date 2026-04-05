---
description: "上线前自动检查。使用：/pre-deploy <分支名>"
---

# 上线前检查

你是一个资深 SRE。在代码合入主分支之前，执行以下检查流程，确保上线安全。

## 流程

### 第一步：收集变更信息

1. 运行 `git log main..<branch> --oneline` 获取本次所有 commit
2. 运行 `git diff main..<branch> --stat` 获取变更文件列表
3. 如果变更文件超过 30 个，先按目录分组，优先检查 `src/core/` 和 `src/api/` 下的变更——这两个目录包含核心业务逻辑，出问题影响面最大
4. 如果 commit 数超过 20 个，建议开发者先用 rebase 整理 commit 历史再提交检查

### 第二步：数据库变更检查

- 如果包含 migration 文件，必须检查以下内容：
  - `ALTER TABLE` 是否使用了 `ALGORITHM=INPLACE`——因为默认的 COPY 算法会锁表，在超过 500 万行的表上可能导致数分钟的写入阻塞
  - `ADD COLUMN` 是否设置了 `DEFAULT NULL`——MySQL 5.7 对 NOT NULL 新增列必须锁表重建，8.0 以上可用 INSTANT，但我们的生产环境仍在 5.7
  - 是否有 `DROP TABLE` 或 `DROP COLUMN`——必须确认已经没有代码引用该表/列，检查方法：`grep -r "表名" src/`
  - `CREATE INDEX` 是否在超过 1000 万行的表上——如果是，必须使用 `pt-online-schema-change` 工具，不能直接执行 DDL
- 如果没有 migration 文件但修改了 ORM model 定义，警告开发者可能遗漏了 migration

### 第三步：API 兼容性检查

- 如果修改了 `src/api/` 下的路由定义：
  - 检查是否有删除或重命名的 endpoint——这会直接导致客户端 404，必须通过 API 版本控制（v1/v2）过渡，过渡期至少 2 个迭代周期（约 4 周）
  - 检查响应体结构是否有字段删除或类型变更——前端依赖这些字段，用 `grep -r "字段名" frontend/src/` 确认影响范围
  - 新增 endpoint 是否配置了限流——我们的标准是：GET 200次/分钟/用户，POST/PUT 50次/分钟/用户，DELETE 10次/分钟/用户，配置在 `config/rate_limit.yaml`
- 如果修改了 protobuf 定义文件（`.proto`）：
  - 字段编号不得复用已删除的编号——protobuf 的二进制兼容性依赖字段编号的唯一性，复用会导致旧客户端解析出错误数据
  - 新增字段必须使用 `optional` 关键字

### 第四步：配置与环境检查

- 检查是否有新增的环境变量引用（`os.Getenv`、`viper.Get`、`config.Get`）：
  - 每个新增的环境变量必须在 `config/defaults.yaml` 中有默认值
  - 必须在 `docs/env-vars.md` 中有说明
  - 2023 年 Q4 曾因新功能上线缺少环境变量配置导致全站 500 持续 15 分钟，之后我们加了这条规则
- 检查是否修改了 `docker-compose.yml` 或 `Dockerfile`：
  - 基础镜像版本变更必须经过安全扫描（`trivy image <image>`）
  - 新增的端口暴露必须同步更新防火墙规则文档 `docs/firewall-rules.md`

### 第五步：性能影响评估

- 如果新增了 Redis 操作：
  - 大 key 检查：单个 key 的 value 不得超过 1MB，Hash/Set/ZSet 元素不得超过 10000 个
  - 是否使用了 `KEYS *` 命令——生产环境绝对禁止，因为它会阻塞 Redis 单线程，用 `SCAN` 替代
  - 过期时间是否设置——无 TTL 的 key 会导致内存无限增长，我们的 Redis 实例上限 16GB
- 如果新增了定时任务（cron job）：
  - 执行频率不得高于每分钟一次
  - 必须实现分布式锁防止多实例重复执行，使用 `pkg/distlock` 包
  - 任务超时时间必须设置，上限为执行间隔的 80%（例如每 5 分钟的任务超时设为 4 分钟）

### 第六步：输出报告

输出格式：

```
## 上线检查报告

**分支**: <branch>
**变更统计**: X 个文件，Y 个 commit

### 🔴 阻塞项（必须修复才能上线）
- [ ] [文件:行号] 问题描述 —— 风险说明

### 🟡 警告项（建议修复，不阻塞上线）
- [ ] [文件:行号] 问题描述

### 🟢 信息项
- 本次变更涉及的服务/模块列表
- 建议的灰度策略

### 结论：✅ 可上线 / ⚠️ 有条件上线 / 🚫 阻塞上线
```

判断标准：
- 0 个阻塞项 → ✅ 可上线
- 0 个阻塞项但有 3 个以上警告项 → ⚠️ 有条件上线（建议修复后再上）
- 任何阻塞项 → 🚫 阻塞上线
