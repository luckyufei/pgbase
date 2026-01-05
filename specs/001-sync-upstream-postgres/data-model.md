# Data Model: Sync Upstream PocketBase Updates to PostgreSQL Branch

**Created**: 2026-01-06  
**Status**: Complete

## Overview

本次功能不引入新的数据模型变更。主要工作是将上游代码合并并适配现有的 PostgreSQL 数据模型。

## 现有数据模型（参考）

PocketBase 使用动态数据模型，核心实体包括：

### 核心表

| 表名 | 描述 | PostgreSQL 特殊处理 |
|------|------|---------------------|
| `_collections` | 集合（表）定义 | `schema` 字段使用 `jsonb` 类型 |
| `_params` | 系统参数 | `value` 字段使用 `jsonb` 类型 |
| `_externalAuths` | 外部认证 | 标准关系表 |
| `_mfas` | 多因素认证 | 标准关系表 |
| `_otps` | 一次性密码 | 标准关系表 |
| `_authOrigins` | 认证来源 | 标准关系表 |
| `_superusers` | 超级用户 | 认证集合 |
| 动态集合表 | 用户定义的数据表 | 根据字段类型使用相应 PostgreSQL 类型 |

### 辅助数据库表

| 表名 | 描述 |
|------|------|
| `_logs` | 系统日志 |
| `_migrations` | 迁移记录 |

## PostgreSQL 类型映射

| PocketBase 字段类型 | PostgreSQL 类型 |
|---------------------|-----------------|
| text | TEXT |
| number | DOUBLE PRECISION |
| bool | BOOLEAN |
| date | TEXT (ISO 8601 格式) |
| select | TEXT (单选) / JSONB (多选) |
| json | JSONB |
| file | TEXT (单文件) / JSONB (多文件) |
| relation | TEXT (单关系) / JSONB (多关系) |
| email | TEXT |
| url | TEXT |
| editor | TEXT |
| password | TEXT (bcrypt hash) |
| autodate | TEXT (ISO 8601 格式) |
| geoPoint | JSONB `{"lon": float, "lat": float}` |

## 无变更声明

本次合并不涉及数据模型变更，仅涉及：

1. 文档文件的合并
2. 潜在的代码适配（如有新增 SQLite 特定代码）

所有现有的 PostgreSQL 数据模型保持不变。
