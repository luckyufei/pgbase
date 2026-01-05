# Tasks: Sync Upstream PocketBase Updates to PostgreSQL Branch

**Input**: Design documents from `/specs/001-sync-upstream-postgres/`
**Prerequisites**: plan.md ✓, spec.md ✓, research.md ✓, data-model.md ✓, quickstart.md ✓

**Tests**: 不需要编写新测试，使用现有测试套件验证适配。

**Organization**: 任务按用户故事分组，支持独立实现和测试。

## Format: `[ID] [P?] [Story] Description`

- **[P]**: 可并行执行（不同文件，无依赖）
- **[Story]**: 任务所属的用户故事（US1, US2, US3）
- 描述中包含确切的文件路径

## Path Conventions

- 本项目为单体应用，源码在仓库根目录
- 核心代码: `core/`, `tools/`, `migrations/`
- 无需创建新目录结构

---

## Phase 1: Setup (准备工作)

**Purpose**: 确保开发环境就绪，获取最新上游代码

- [X] T001 确保本地 postgres 分支是最新的: `git checkout postgres && git pull`
- [X] T002 获取上游最新代码: `git fetch pocketbase --prune`
- [X] T003 [P] 确认 PostgreSQL 数据库服务运行中并可连接
- [X] T004 [P] 验证 Go 1.24.0 环境: `go version`

---

## Phase 2: Foundational (基础验证)

**Purpose**: 在合并前验证当前代码状态

**⚠️ CRITICAL**: 确保当前代码在合并前是稳定的

- [X] T005 验证当前代码编译通过: `go build ./...`
- [X] T006 运行现有测试套件确认基线: `go test ./... -count=1`
- [X] T007 记录当前 postgres 分支的 HEAD commit: `git log -1 --oneline`
- [X] T008 查看待合并的上游提交: `git log postgres..pocketbase/master --oneline`

**Checkpoint**: 基础验证完成，可以开始合并操作

---

## Phase 3: User Story 1 - Merge Upstream Changes (Priority: P1) 🎯 MVP

**Goal**: 将上游 pocketbase/master 分支的最新提交合并到 postgres 分支

**Independent Test**: 合并后代码能够成功编译 (`go build ./...`)

### Implementation for User Story 1

- [X] T009 [US1] 执行上游合并: `git merge pocketbase/master --no-edit`
- [X] T010 [US1] 如有冲突，查看冲突文件列表: `git status`
- [X] T011 [US1] 解决冲突时优先保留 PostgreSQL 适配代码（参考 research.md 中的适配模式）
- [X] T012 [US1] 冲突解决后标记完成: `git add . && git merge --continue`
- [X] T013 [US1] 验证合并后代码编译通过: `go build ./...`
- [X] T014 [US1] 提交合并结果（如果使用 --no-edit 则自动完成）

**Checkpoint**: 上游代码已合并，代码可编译

---

## Phase 4: User Story 2 - Adapt SQLite-specific Code to PostgreSQL (Priority: P1)

**Goal**: 将上游新增的 SQLite 特定代码适配为 PostgreSQL 兼容代码

**Independent Test**: 所有测试在 PostgreSQL 环境下通过 (`go test ./...`)

### Implementation for User Story 2

- [ ] T015 [US2] 检查合并后是否有新增的 SQLite 特定代码: `git diff HEAD~1 --name-only | xargs grep -l "sqlite\|SQLite" 2>/dev/null || echo "No SQLite-specific changes"`
- [ ] T016 [P] [US2] 检查 core/base.go 是否有需要适配的变更
- [ ] T017 [P] [US2] 检查 core/db_connect.go 是否有需要适配的变更
- [ ] T018 [P] [US2] 检查 core/db_table.go 是否有需要适配的变更
- [ ] T019 [P] [US2] 检查 tools/dbutils/json.go 是否有需要适配的变更
- [ ] T020 [P] [US2] 检查 tools/search/filter.go 是否有需要适配的变更
- [ ] T021 [P] [US2] 检查 migrations/ 目录是否有新增迁移脚本需要适配
- [ ] T022 [US2] 如发现 SQLite 特定语法，按 research.md 中的映射模式进行适配
- [ ] T023 [US2] 验证适配后代码编译通过: `go build ./...`
- [ ] T024 [US2] 运行完整测试套件验证适配: `go test ./... -count=1`

**Checkpoint**: 所有 SQLite 特定代码已适配为 PostgreSQL 兼容代码

---

## Phase 5: User Story 3 - Maintain Database Abstraction Layer (Priority: P2)

**Goal**: 确保数据库访问层保持良好的抽象，便于未来维护

**Independent Test**: 代码审查确认 PostgreSQL 特定代码集中在指定模块

### Implementation for User Story 3

- [ ] T025 [US3] 审查 core/db_connect.go 确认 PostgreSQL 连接逻辑集中
- [ ] T026 [P] [US3] 审查 migrations/postgres_functions.go 确认 PostgreSQL 函数定义集中
- [ ] T027 [P] [US3] 审查 tools/dbutils/json.go 确认 JSON 操作适配逻辑清晰
- [ ] T028 [US3] 如有分散的 PostgreSQL 特定代码，重构到指定模块
- [ ] T029 [US3] 确认代码中的 SQLite/PostgreSQL 注释清晰标注了差异

**Checkpoint**: 数据库抽象层结构良好，便于未来维护

---

## Phase 6: Polish & Validation (最终验证)

**Purpose**: 完整验证和清理

- [ ] T030 运行完整测试套件: `go test ./... -count=1 -v`
- [ ] T031 [P] 验证应用能够启动并连接 PostgreSQL: 参考 quickstart.md
- [ ] T032 [P] 验证 CRUD 操作正常工作
- [ ] T033 [P] 验证 JSON 字段操作正常工作
- [ ] T034 推送合并后的代码到远程: `git push`
- [ ] T035 更新 spec.md 状态为 Complete

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: 无依赖 - 可立即开始
- **Foundational (Phase 2)**: 依赖 Setup 完成
- **User Story 1 (Phase 3)**: 依赖 Foundational 完成 - 必须首先完成
- **User Story 2 (Phase 4)**: 依赖 User Story 1 完成（需要合并后的代码）
- **User Story 3 (Phase 5)**: 依赖 User Story 2 完成
- **Polish (Phase 6)**: 依赖所有 User Story 完成

### User Story Dependencies

- **User Story 1 (P1)**: 无其他故事依赖 - 是后续故事的前提
- **User Story 2 (P1)**: 依赖 US1 完成（需要合并后的代码才能适配）
- **User Story 3 (P2)**: 依赖 US2 完成（需要适配完成后才能审查抽象层）

### Within Each User Story

- 检查任务可并行执行
- 适配任务需要按发现的问题顺序执行
- 验证任务在适配完成后执行

### Parallel Opportunities

- T003, T004 可并行执行
- T016-T021 可并行执行（检查不同文件）
- T026, T027 可并行执行
- T031-T033 可并行执行

---

## Parallel Example: User Story 2 检查阶段

```bash
# 并行检查所有可能需要适配的文件:
Task: "检查 core/base.go 是否有需要适配的变更"
Task: "检查 core/db_connect.go 是否有需要适配的变更"
Task: "检查 core/db_table.go 是否有需要适配的变更"
Task: "检查 tools/dbutils/json.go 是否有需要适配的变更"
Task: "检查 tools/search/filter.go 是否有需要适配的变更"
Task: "检查 migrations/ 目录是否有新增迁移脚本需要适配"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. 完成 Phase 1: Setup
2. 完成 Phase 2: Foundational
3. 完成 Phase 3: User Story 1 (合并上游)
4. **STOP and VALIDATE**: 验证代码编译通过
5. 如果只是文档变更，可以直接跳到 Phase 6

### Incremental Delivery

1. 完成 Setup + Foundational → 准备就绪
2. 完成 User Story 1 → 验证编译 → 合并完成
3. 完成 User Story 2 → 验证测试 → 适配完成
4. 完成 User Story 3 → 代码审查 → 抽象层验证
5. 完成 Polish → 推送代码

### 当前情况说明

根据 research.md 分析，当前只有一个文档提交 (c9dae081) 需要合并：
- 预计无代码冲突
- 预计无需 SQLite → PostgreSQL 适配
- User Story 2 和 3 可能只需要验证确认，无需实际修改

---

## Notes

- [P] 任务 = 不同文件，无依赖
- [Story] 标签将任务映射到特定用户故事
- 当前合并只涉及文档变更，风险极低
- 如发现需要适配的代码，参考 research.md 中的映射模式
- 每个任务完成后提交
- 在任何检查点停下来验证
