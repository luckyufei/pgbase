# Research: Sync Upstream PocketBase Updates to PostgreSQL Branch

**Created**: 2026-01-06  
**Status**: Complete

## 1. 上游变更分析

### 待合并提交

| Commit | Message | Impact |
|--------|---------|--------|
| c9dae081 | docs: 添加 PocketBase 架构设计文档和完整指南 | 低 - 仅文档变更 |

**Decision**: 当前只有一个文档提交需要合并，不涉及代码变更，合并风险极低。

**Rationale**: 通过 `git log b1da83e5..pocketbase/master` 确认只有一个提交。

**Alternatives considered**: 无需考虑其他方案。

## 2. SQLite 到 PostgreSQL 映射模式

### 2.1 已实现的适配模式

项目中已经建立了完善的 SQLite → PostgreSQL 适配模式：

| SQLite 特性 | PostgreSQL 等效 | 实现位置 |
|-------------|-----------------|----------|
| `json_extract()` | `JSON_QUERY_OR_NULL()` 自定义函数 | `tools/dbutils/json.go` |
| `json_each()` | `jsonb_array_elements_text()` | `tools/dbutils/json.go` |
| `json_array_length()` | `jsonb_array_length()` | `tools/dbutils/json.go` |
| `json_valid()` | 自定义 `json_valid()` 函数 | `migrations/postgres_functions.go` |
| `hex()` | 自定义 `hex()` 函数 | `migrations/postgres_functions.go` |
| `randomblob()` | `gen_random_bytes()` 封装 | `migrations/postgres_functions.go` |
| 反引号标识符 `` ` `` | 双引号标识符 `"` | `core/base.go` (sqlLogReplacements) |
| `NOCASE` 排序规则 | 自定义 `nocase` 排序规则 | `migrations/postgres_functions.go` |

**Decision**: 继续使用现有的适配模式，保持代码一致性。

**Rationale**: 现有模式已经过验证，覆盖了主要的 SQLite 特性。

## 3. 数据库连接模式

### 3.1 连接池配置

| 参数 | SQLite 值 | PostgreSQL 值 | 原因 |
|------|-----------|---------------|------|
| DataMaxOpenConns | 120 | 70 | PostgreSQL 默认最大连接数为 100 |
| DataMaxIdleConns | - | 15 | 保持合理的空闲连接 |
| AuxMaxOpenConns | - | 20 | 辅助数据库连接 |
| AuxMaxIdleConns | - | 3 | 辅助数据库空闲连接 |

**Decision**: 保持现有的 PostgreSQL 连接池配置。

**Rationale**: 配置已考虑 PostgreSQL 的连接限制（默认 100），预留了 pgAdmin 等工具的连接空间。

## 4. 并发写入处理

### 4.1 SQLite vs PostgreSQL

| 特性 | SQLite | PostgreSQL |
|------|--------|------------|
| 并发写入 | 不支持（需要串行化） | 支持 |
| 连接分离 | 需要 concurrent/nonconcurrent 分离 | 可以共享连接 |

**Decision**: PostgreSQL 模式下 `concurrentDB` 和 `nonconcurrentDB` 指向同一连接池。

**Rationale**: PostgreSQL 原生支持并发写入，无需像 SQLite 那样分离连接。

**实现位置**: `core/base.go` 第 1250 行和第 1337 行。

## 5. 合并策略

### 5.1 推荐流程

1. **执行合并**: `git merge pocketbase/master`
2. **解决冲突**: 如有冲突，优先保留 PostgreSQL 适配代码
3. **编译验证**: `go build ./...`
4. **测试验证**: `go test ./...`

**Decision**: 使用标准 git merge 流程。

**Rationale**: 当前只有文档变更，预计无冲突。

## 6. 潜在风险区域

### 6.1 高风险文件（未来合并需关注）

| 文件 | 风险原因 | 适配策略 |
|------|----------|----------|
| `core/base.go` | 包含大量 PostgreSQL 特定配置 | 仔细审查每个变更 |
| `tools/dbutils/json.go` | JSON 函数适配 | 检查新增的 JSON 操作 |
| `tools/search/filter.go` | 查询构建逻辑 | 验证 SQL 语法兼容性 |
| `migrations/*.go` | 数据库迁移 | 确保 PostgreSQL 语法 |

**Decision**: 建立代码审查检查清单，关注上述文件的变更。

## 7. 测试策略

### 7.1 验证步骤

1. **编译测试**: 确保代码无编译错误
2. **单元测试**: 运行 `go test ./...`
3. **集成测试**: 连接实际 PostgreSQL 数据库验证
4. **功能测试**: 验证 CRUD、JSON 操作、全文搜索等核心功能

**Decision**: 使用现有测试套件，确保 PostgreSQL 环境下通过。

**Rationale**: 项目已有完善的测试覆盖，包括 PostgreSQL 特定的测试辅助函数 `NewBaseAppForTest`。
