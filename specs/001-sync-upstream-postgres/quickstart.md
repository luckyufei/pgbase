# Quickstart: Sync Upstream PocketBase Updates to PostgreSQL Branch

**Created**: 2026-01-06

## 前置条件

- Go 1.24.0 或更高版本
- PostgreSQL 12 或更高版本
- Git

## 快速开始

### 1. 合并上游变更

```bash
# 确保在 postgres 分支
git checkout postgres

# 获取最新的上游代码
git fetch pocketbase

# 合并上游 master 分支
git merge pocketbase/master

# 如有冲突，解决后继续
# git add .
# git merge --continue
```

### 2. 验证编译

```bash
# 编译项目
go build ./...
```

### 3. 配置 PostgreSQL

确保 PostgreSQL 数据库可用：

```bash
# 创建测试数据库（可选）
createdb pb_data
createdb pb_auxiliary
```

### 4. 运行测试

```bash
# 设置 PostgreSQL 连接（根据实际配置调整）
export PGPASSWORD=pass

# 运行测试
go test ./...
```

### 5. 启动应用

```go
package main

import (
    "log"
    "github.com/pocketbase/pocketbase"
    "github.com/pocketbase/pocketbase/core"
)

func main() {
    app := pocketbase.NewWithConfig(pocketbase.Config{
        DefaultDataDir: "./pb_data",
    })

    // 配置 PostgreSQL
    app.RootCmd.PersistentFlags().StringVar(
        &app.Config().PostgresURL,
        "postgres-url",
        "postgres://user:pass@localhost:5432?sslmode=disable",
        "PostgreSQL connection URL",
    )

    if err := app.Start(); err != nil {
        log.Fatal(err)
    }
}
```

## 常见问题

### Q: 如何处理合并冲突？

优先保留 PostgreSQL 适配代码。关键文件：
- `core/base.go` - 连接配置
- `tools/dbutils/json.go` - JSON 函数
- `migrations/postgres_functions.go` - PostgreSQL 函数

### Q: 测试失败怎么办？

1. 确认 PostgreSQL 服务正在运行
2. 检查连接配置（用户名、密码、端口）
3. 确保测试数据库存在或有创建权限

### Q: 如何验证 PostgreSQL 适配是否正确？

检查以下功能：
- 创建/读取/更新/删除记录
- JSON 字段操作
- 关联查询
- 全文搜索
