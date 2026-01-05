# Implementation Plan: Sync Upstream PocketBase Updates to PostgreSQL Branch

**Branch**: `001-sync-upstream-postgres` | **Date**: 2026-01-06 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-sync-upstream-postgres/spec.md`

## Summary

将上游 pocketbase/master 分支的最新提交合并到 postgres 分支，并将新增的 SQLite 特定代码适配为 PostgreSQL 兼容代码。当前 postgres 分支基于提交 b1da83e5，需要合并的上游提交为 c9dae081（docs: 添加 PocketBase 架构设计文档和完整指南）。

## Technical Context

**Language/Version**: Go 1.24.0  
**Primary Dependencies**: 
- `github.com/jackc/pgx/v5` - PostgreSQL 驱动
- `github.com/pocketbase/dbx` (替换为 `github.com/fondoger/dbx`) - 数据库抽象层
- `modernc.org/sqlite` - SQLite 驱动（保留用于兼容性）
**Storage**: PostgreSQL 12+（使用 jsonb 类型）  
**Testing**: Go 标准测试框架 (`go test`)  
**Target Platform**: Linux/macOS/Windows 服务器  
**Project Type**: 单体应用（后端服务 + 嵌入式前端 UI）  
**Performance Goals**: 支持 70 个并发数据库连接，20 个辅助数据库连接  
**Constraints**: 保持与上游 PocketBase 的兼容性，便于未来合并  
**Scale/Scope**: 中小型应用，单实例部署或通过 PostgreSQL LISTEN/NOTIFY 实现水平扩展

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| 代码质量 | ✅ Pass | 遵循现有代码风格和模式 |
| 测试覆盖 | ✅ Pass | 运行现有测试套件验证适配 |
| 数据库抽象 | ✅ Pass | PostgreSQL 特定代码集中在指定模块 |
| 向后兼容 | ✅ Pass | 保持 API 兼容性 |

## Project Structure

### Documentation (this feature)

```text
specs/001-sync-upstream-postgres/
├── plan.md              # This file
├── research.md          # Phase 0 output - SQLite to PostgreSQL 映射研究
├── data-model.md        # Phase 1 output - 数据模型无变更（仅适配）
├── quickstart.md        # Phase 1 output - 快速开始指南
├── contracts/           # Phase 1 output - 无新 API（仅内部适配）
└── tasks.md             # Phase 2 output (/speckit.tasks command)
```

### Source Code (repository root)

```text
# 现有项目结构（无需修改）
core/
├── base.go              # PostgreSQL 连接配置
├── db_connect.go        # 数据库连接函数
├── db_table.go          # 表操作
├── view.go              # 视图操作
└── ...

migrations/
├── postgres_functions.go # PostgreSQL 兼容函数
├── 1640988000_init.go   # 初始化迁移
└── ...

tools/
├── dbutils/
│   ├── json.go          # JSON 操作适配
│   └── index.go         # 索引操作
└── search/
    ├── filter.go        # 查询过滤器
    └── sort.go          # 排序逻辑
```

**Structure Decision**: 保持现有项目结构不变，仅在现有文件中进行 SQLite → PostgreSQL 适配。

## Complexity Tracking

> 无违规需要记录 - 本次任务是代码适配，不引入新的架构复杂性。
