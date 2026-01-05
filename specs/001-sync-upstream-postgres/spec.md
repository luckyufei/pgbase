# Feature Specification: Sync Upstream PocketBase Updates to PostgreSQL Branch

**Feature Branch**: `001-sync-upstream-postgres`  
**Created**: 2026-01-06  
**Status**: Draft  
**Input**: User description: "将 pocketbase/master-main 后面的逻辑更新到 postgres 分支，并改造成使用 postgresql db 的代码"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Merge Upstream Changes (Priority: P1)

作为项目维护者，我需要将上游 pocketbase/master 分支的最新提交合并到 postgres 分支，以便获取上游的 bug 修复和新功能。

**Why this priority**: 这是整个功能的基础，没有合并上游代码，后续的 PostgreSQL 适配工作无法进行。

**Independent Test**: 可以通过执行 `git merge pocketbase/master` 并解决冲突后，验证代码能够编译通过来测试。

**Acceptance Scenarios**:

1. **Given** postgres 分支基于 pocketbase/master 的某个历史提交, **When** 执行上游合并操作, **Then** 上游的新提交（目前为 c9dae081）被合并到 postgres 分支
2. **Given** 合并过程中存在代码冲突, **When** 解决冲突后, **Then** 代码能够成功编译且不丢失 PostgreSQL 相关的改动

---

### User Story 2 - Adapt SQLite-specific Code to PostgreSQL (Priority: P1)

作为项目维护者，我需要将上游新增的 SQLite 特定代码适配为 PostgreSQL 兼容的代码，以确保系统能够正常使用 PostgreSQL 数据库运行。

**Why this priority**: 这是核心功能需求，确保合并后的代码能够在 PostgreSQL 环境下正常工作。

**Independent Test**: 可以通过运行现有的测试套件并连接到 PostgreSQL 数据库来验证适配是否成功。

**Acceptance Scenarios**:

1. **Given** 上游代码包含 SQLite 特定的 SQL 语法, **When** 适配完成后, **Then** 相应代码使用 PostgreSQL 兼容的 SQL 语法
2. **Given** 上游代码使用 SQLite 特定函数（如 json_extract、group_concat）, **When** 适配完成后, **Then** 使用 PostgreSQL 等效函数（如 jsonb 操作符、string_agg）
3. **Given** 上游代码包含 SQLite 特定的数据类型, **When** 适配完成后, **Then** 使用 PostgreSQL 对应的数据类型

---

### User Story 3 - Maintain Database Abstraction Layer (Priority: P2)

作为开发者，我希望数据库访问层保持良好的抽象，以便未来能够更容易地适配其他数据库或接收上游更新。

**Why this priority**: 良好的抽象层设计能够降低未来维护成本，但不是当前功能的核心阻塞项。

**Independent Test**: 可以通过代码审查验证数据库特定代码是否集中在特定模块中。

**Acceptance Scenarios**:

1. **Given** 需要修改数据库特定代码, **When** 查看代码结构, **Then** PostgreSQL 特定代码集中在 `core/db_connect.go`、`migrations/` 等特定位置
2. **Given** 上游有新的数据库操作代码, **When** 适配完成后, **Then** 保持与现有 PostgreSQL 适配模式的一致性

---

### Edge Cases

- 上游代码引入了新的 SQLite 扩展功能，PostgreSQL 没有直接对应的实现时如何处理？
- 合并冲突涉及到核心数据库逻辑时，如何确保不破坏现有的 PostgreSQL 功能？
- 上游修改了数据库 schema 迁移逻辑时，如何保持 PostgreSQL 迁移的兼容性？

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: 系统 MUST 成功合并 pocketbase/master 分支从 b1da83e5 到最新提交的所有变更
- **FR-002**: 系统 MUST 将上游新增的 SQLite 特定 SQL 语法转换为 PostgreSQL 兼容语法
- **FR-003**: 系统 MUST 保持现有的 PostgreSQL 连接和查询功能正常工作
- **FR-004**: 系统 MUST 确保所有数据库迁移脚本在 PostgreSQL 上正确执行
- **FR-005**: 系统 MUST 保持 JSON 字段操作在 PostgreSQL 上的正确性（使用 jsonb 类型和相关操作符）
- **FR-006**: 系统 MUST 保持全文搜索功能在 PostgreSQL 上的正确性
- **FR-007**: 系统 MUST 保持地理位置（GeoPoint）字段在 PostgreSQL 上的正确性

### Key Entities

- **Database Connection**: 数据库连接配置，包含连接字符串、连接池设置等
- **Migration Scripts**: 数据库迁移脚本，负责创建和更新数据库 schema
- **Query Builder**: 查询构建器，需要处理 SQLite 和 PostgreSQL 的语法差异
- **JSON Operations**: JSON 字段操作，SQLite 使用 json_extract，PostgreSQL 使用 jsonb 操作符

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 合并完成后，代码能够成功编译，无编译错误
- **SC-002**: 所有现有单元测试在 PostgreSQL 环境下通过率达到 100%
- **SC-003**: 应用能够成功启动并连接到 PostgreSQL 数据库
- **SC-004**: 所有 CRUD 操作（创建、读取、更新、删除记录）在 PostgreSQL 上正常工作
- **SC-005**: 数据库迁移能够在全新的 PostgreSQL 数据库上成功执行
- **SC-006**: 复杂查询（包含 JSON 操作、关联查询、全文搜索）返回正确结果

## Assumptions

- 当前 postgres 分支已经包含基本的 PostgreSQL 适配代码
- 上游 pocketbase/master 的变更主要是 bug 修复和小功能改进，不涉及大规模架构变更
- PostgreSQL 版本为 12 或更高版本，支持 jsonb 和相关操作
- 开发环境已配置好 PostgreSQL 数据库用于测试
