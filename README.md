# skills

FastAdmin + ThinkPHP6 全栈开发 AI Agent Skills，涵盖 FastAdmin 后台 CRUD、ThinkORM 数据访问及 MySQL 数据库设计规范。

## Skills 列表

### 数据库

| Skill | 描述 | 技术栈 |
| --- | --- | --- |
| [`fastadmin-mysql-design`](https://skills.sh/yoohip/skills/fastadmin-mysql-design) | FastAdmin + ThinkPHP6 MySQL 数据库设计规范 | FastAdmin + ThinkPHP6 + MySQL 8 |

## 安装

```bash
npx skills add yoohip/skills
```

## 使用方式

Skills 根据项目类型自动触发：

| 项目类型 | 触发 Skill |
| --- | --- |
| FastAdmin 数据库设计 | `fastadmin-mysql-design` |
| 设计表结构 / 生成 CRUD | `fastadmin-mysql-design` |
| ThinkORM 模型与 Db 查询 | `fastadmin-mysql-design` |

## 每个 Skill 包含

- **命名规范** - 表名、字段名、索引命名规则
- **字段类型规范** - FastAdmin CRUD 字段类型与组件映射
- **通用字段** - createtime / updatetime / deletetime / weigh / status
- **金额规范** - 默认 DECIMAL(19,4)，bcmath 运算约定
- **完整表示例** - 业务表、关联表、订单金额表示例 SQL
- **ThinkORM 规范** - Db 查询、模型时间戳、软删除、类型转换
- **CRUD 工作流** - `php think crud` 生成与表结构变更注意事项
- **设计检查清单** - 建表完成检查项

## 发布到 skills.sh

1. 创建 GitHub 仓库 `yoohip/skills`
2. 推送代码到 main 分支
3. 访问 `https://skills.sh/yoohip/skills` 即可被索引

## 本地开发

```bash
# 克隆仓库
git clone https://github.com/yoohip/skills.git

# 目录结构
skills/
├── README.md
└── skills/
    └── fastadmin-mysql-design/SKILL.md
```
