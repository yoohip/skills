---
name: fastadmin-mysql-design
description: FastAdmin + ThinkPHP6 MySQL 数据库设计规范。设计表结构、字段命名、索引，或配合 php think crud 一键生成 CRUD 时使用。涵盖 FastAdmin 字段后缀映射、Unix 时间戳、软删除、金额 DECIMAL(19,4)、ThinkORM 查询与模型约定。
---

# FastAdmin + ThinkPHP6 数据库设计规范

> 参考：[FastAdmin 数据库文档](https://doc.fastadmin.net/doc/database.html)、[ThinkPHP6 数据库手册](https://www.kancloud.cn/manual/thinkphp6_0/1037530)

## 触发条件

- 设计 FastAdmin 项目数据库表结构
- 定义字段类型、命名，以便 `php think crud -t 表名` 自动生成组件
- 编写 ThinkORM 模型、Db 查询、迁移 SQL
- 创建索引、关联表
- 命名表和字段

---

## Part 1: 命名规范

### 库名

| 规则 | 说明 |
|------|------|
| 小写字母 | `fa_admin` |
| 下划线分隔 | 不用驼峰 |
| 业务前缀 | 按项目约定，如 `fa_` |

### 表名

| 类型 | 规范 | 示例 |
|------|------|------|
| 全小写 | 仅 `a-z` 和 `_`，禁止拼音 | `fa_user`, `fa_article` |
| 插件表 | `{前缀}{插件标识}_{模块}` | `fa_mydemo_item`, `fa_mydemo_log` |
| 日志表 | 模块名 + `_log` | `fa_order_log` |
| 关联表 | 两表名组合或 `_rel` | `fa_user_group` |

### 字段命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 主键 | `id` | `id` |
| 外键（单选） | `{表名}_id` | `user_id`, `category_id` |
| 外键（多选） | `{表名}_ids` | `user_ids`（逗号分隔，非 JSON） |
| 自关联 | `pid` / `parent_id` / `father_id` | 关联本表主键 |
| 时间 | 以 `time` 结尾 | `createtime`, `paytime`, `expiretime` |
| 时长（秒） | 以 `seconds` 结尾 | `onlineseconds` |
| 数量 | 以 `nums` 结尾 | `buynums`, `salenums` |
| 金额 | 以 `price` / `amount` / `fee` / `money` 结尾 | `unit_price`, `total_amount`, `pay_fee` |
| 可数名词 | 加 `s` | `comments`, `views` |
| JSON 内容 | 以 `data` 结尾 | `itemdata`, `rewarddata` |
| 多 user_id | 自己用 `user_id`，他人用 `receiver_user_id` | 多人用 `receiver_user_ids` |

### 索引命名

| 类型 | 前缀 | 示例 |
|------|------|------|
| 主键 | `PRIMARY KEY` | `PRIMARY KEY (id)` |
| 唯一索引 | `uk_` | `uk_username` |
| 普通索引 | `idx_` 或直接 KEY | `idx_user_id`, KEY `pid` (`pid`) |
| 组合索引 | `idx_` | `idx_status_weigh` |

---

## Part 2: 引擎与字符集

```sql
ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
```

| 设置 | 说明 |
|------|------|
| InnoDB | 事务、行锁，FastAdmin 默认 |
| utf8mb4 | 支持 emoji |
| utf8mb4_general_ci | FastAdmin 官方推荐排序规则 |

插件 `install.sql` 中使用 `__PREFIX__` 占位符，安装时自动替换为配置的表前缀：

```sql
CREATE TABLE IF NOT EXISTS `__PREFIX__mydemo_list` (
  ...
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='示例表';
```

---

## Part 3: 通用字段（FastAdmin 约定）

### 推荐必备字段

```sql
`id` int(16) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
`createtime` int(16) DEFAULT NULL COMMENT '创建时间',
`updatetime` int(16) DEFAULT NULL COMMENT '更新时间',
`deletetime` int(16) DEFAULT NULL COMMENT '删除时间',
PRIMARY KEY (`id`)
```

### 软删除与回收站

```sql
`deletetime` int(10) DEFAULT NULL COMMENT '删除时间'
```

- 存在 `deletetime` 且默认值为 `NULL` 时，FastAdmin 自动生成回收站功能
- 框架自动维护，无需手动赋值
- 与 youlai 规范的 `deleted TINYINT` 不同，FastAdmin 优先使用 `deletetime`

### 常用可选字段

```sql
`weigh` int(10) NOT NULL DEFAULT 0 COMMENT '权重',
`status` enum('normal','hidden') NOT NULL DEFAULT 'normal' COMMENT '状态',
`memo` varchar(255) DEFAULT '' COMMENT '备注'
```

| 字段 | 类型 | 作用 |
|------|------|------|
| `weigh` | int | 后台拖拽排序 |
| `status` | enum | TAB 选项卡筛选列表 |
| `category_id` | int | 关联 `fa_category`，生成 selectpage |
| `user_id` | int | 关联 `fa_user`，生成 selectpage |

### 时间字段规范

FastAdmin **统一使用 Unix 秒级时间戳**，不用 DATETIME：

```sql
`createtime` int(10) DEFAULT NULL COMMENT '创建时间',
`updatetime` int(10) DEFAULT NULL COMMENT '更新时间',
`publishtime` int(10) DEFAULT NULL COMMENT '发布时间'
```

---

## Part 4: 数值类型规范

### 金额字段（默认）

金额、单价、费用、余额等**一律使用 `DECIMAL(19,4)`**，禁止使用 `FLOAT` / `DOUBLE`（浮点精度丢失）。

```sql
`unit_price` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '单价',
`total_amount` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '总金额',
`pay_fee` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '手续费'
```

| 场景 | 类型 | 说明 |
|------|------|------|
| 单价 / 金额 / 费用 / 余额 | DECIMAL(19,4) | 默认规范，整数部分最多 15 位，小数 4 位 |
| 数量 / 计数 | int | 非金额整数，如库存、浏览量 |
| 百分比 | DECIMAL(5,2) | 0.00–100.00，非金额场景 |
| 汇率 | DECIMAL(19,6) | 需更高精度时单独约定，仍用 DECIMAL |

**命名**：字段名体现含义，推荐 `unit_price`、`total_amount`、`pay_amount`、`refund_fee`、`balance` 等。

**默认值**：金额字段 `NOT NULL DEFAULT 0.0000`，避免 NULL 参与运算。

**PHP 运算**：模型内用 `bcadd` / `bcsub` / `bcmul` / `bcdiv`（scale=4），禁止直接用 float 做金额加减乘除。

### 其他数值类型

| 场景 | 类型 | 示例 |
|------|------|------|
| 主键 / 外键 | int(10) UNSIGNED | `id`, `user_id` |
| 排序 | int | `weigh` |
| 计数 | int | `views`, `buynums` |

---

## Part 5: 字段类型与 CRUD 组件映射

按字段类型，`php think crud` 自动生成对应表单组件：

| 类型 | 生成组件 |
|------|----------|
| int | type=number 文本框，步长 1 |
| enum | 单选下拉 |
| set | 多选下拉 |
| float / decimal | number 文本框，步长随小数位（decimal 推荐步长 0.0001） |
| text | textarea |
| datetime / date / timestamp | 日期时间组件 |

### 特殊字段（精确匹配）

| 字段名 | 类型 | CRUD 行为 |
|--------|------|-----------|
| category_id | int | 分类下拉（单选） |
| category_ids | varchar | 分类下拉（多选） |
| weigh | int | 排序拖拽按钮 |
| createtime | int | 自动维护，不需手填 |
| updatetime | int | 自动维护 |
| deletetime | int | 软删除 + 回收站 |
| status | enum | TAB 选项卡 + 筛选 |

### 字段后缀规则（以 xxx 结尾）

| 后缀 | 示例 | 类型要求 | 生成组件 |
|------|------|----------|----------|
| time | refreshtime | int | 日期时间选择器 |
| image | smallimage | varchar | 单图上传 |
| images | smallimages | varchar | 多图上传 |
| file | attachfile | varchar | 单文件上传 |
| files | attachfiles | varchar | 多文件上传 |
| avatar | miniavatar | varchar | 头像上传（单图） |
| avatars | miniavatars | varchar | 头像上传（多图） |
| content | maincontent | text | 富文本编辑器 |
| _id | user_id | int/varchar | selectpage 关联（单选） |
| _ids | user_ids | varchar | selectpage 关联（多选） |
| list | timelist | enum/set | 单选/多选下拉 |
| data | hobbydata | enum/set | 单选框/复选框 |
| json | configjson | varchar | 键值录入组件 |
| switch | siteswitch | tinyint | 开关组件 |
| price | unit_price | decimal(19,4) | number 文本框，步长 0.0001 |
| amount | total_amount | decimal(19,4) | number 文本框，步长 0.0001 |
| fee | pay_fee | decimal(19,4) | number 文本框，步长 0.0001 |

> `list` / `data` 后缀必须搭配 enum 或 set 类型才生效。

### 注释与字典

字段 COMMENT 会被 CRUD 解析为字典：

```sql
`status` enum('0','1','2') NOT NULL DEFAULT '1' COMMENT '状态:0=隐藏,1=正常,2=推荐'
```

---

## Part 6: 完整表示例

```sql
CREATE TABLE IF NOT EXISTS `__PREFIX__article` (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `category_id` int(10) UNSIGNED NOT NULL DEFAULT 0 COMMENT '分类ID',
  `user_id` int(10) UNSIGNED DEFAULT NULL COMMENT '作者ID',
  `title` varchar(200) NOT NULL DEFAULT '' COMMENT '标题',
  `image` varchar(255) DEFAULT '' COMMENT '封面图',
  `content` text COMMENT '内容',
  `views` int(10) UNSIGNED NOT NULL DEFAULT 0 COMMENT '浏览量',
  `weigh` int(10) NOT NULL DEFAULT 0 COMMENT '权重',
  `status` enum('normal','hidden') NOT NULL DEFAULT 'normal' COMMENT '状态',
  `createtime` int(10) DEFAULT NULL COMMENT '创建时间',
  `updatetime` int(10) DEFAULT NULL COMMENT '更新时间',
  `deletetime` int(10) DEFAULT NULL COMMENT '删除时间',
  PRIMARY KEY (`id`),
  KEY `category_id` (`category_id`),
  KEY `user_id` (`user_id`),
  KEY `weigh` (`weigh`),
  KEY `status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='文章表';
```

### 关联表示例

```sql
CREATE TABLE IF NOT EXISTS `__PREFIX__user_group` (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `user_id` int(10) UNSIGNED NOT NULL COMMENT '用户ID',
  `group_id` int(10) UNSIGNED NOT NULL COMMENT '角色组ID',
  `createtime` int(10) DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_user_group` (`user_id`, `group_id`),
  KEY `group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户角色关联表';
```

### 含金额字段表示例

```sql
CREATE TABLE IF NOT EXISTS `__PREFIX__order` (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'ID',
  `user_id` int(10) UNSIGNED NOT NULL COMMENT '用户ID',
  `order_no` varchar(32) NOT NULL DEFAULT '' COMMENT '订单号',
  `total_amount` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '订单总金额',
  `pay_amount` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '实付金额',
  `discount_amount` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '优惠金额',
  `pay_fee` DECIMAL(19,4) NOT NULL DEFAULT 0.0000 COMMENT '手续费',
  `status` enum('normal','hidden') NOT NULL DEFAULT 'normal' COMMENT '状态',
  `paytime` int(10) DEFAULT NULL COMMENT '支付时间',
  `createtime` int(10) DEFAULT NULL COMMENT '创建时间',
  `updatetime` int(10) DEFAULT NULL COMMENT '更新时间',
  `deletetime` int(10) DEFAULT NULL COMMENT '删除时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_order_no` (`order_no`),
  KEY `user_id` (`user_id`),
  KEY `status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

关联命名：`{关联表}_id` 对应 `fa_{关联表}` 主键，如 `company_info_id` → `fa_company_info`。

---

## Part 7: 索引规范

| 原则 | 说明 |
|------|------|
| 主键 | 单字段 `id`，不支持复合主键 |
| 外键字段 | 必须建索引 |
| 筛选字段 | `status`、`category_id` 等常查字段建索引 |
| 排序字段 | `weigh`、`createtime` 按需建索引 |
| 控制数量 | 单表索引不超过 5 个 |
| 组合优先 | 高频组合查询用组合索引 |

### 索引避坑

| 避免 | 原因 |
|------|------|
| `LIKE '%xxx%'` | 前缀模糊不走索引 |
| WHERE 中对字段用函数 | `YEAR(createtime)` 不走索引 |
| 字符串与数字隐式转换 | 导致索引失效 |
| 大 OFFSET 分页 | 改用 id 游标分页 |

---

## Part 8: ThinkPHP6 / ThinkORM 规范

> 必须使用门面 `think\facade\Db` 操作数据库。

### 数据库配置（config/database.php）

```php
return [
    'default'     => 'mysql',
    'connections' => [
        'mysql' => [
            'type'     => 'mysql',
            'hostname' => '127.0.0.1',
            'database' => 'fastadmin',
            'username' => 'root',
            'password' => '',
            'hostport' => '3306',
            'charset'  => 'utf8mb4',
            'prefix'   => 'fa_',
            'params'   => [],
        ],
    ],
];
```

### Db 查询

```php
use think\facade\Db;

// 查询
Db::name('article')->where('status', 'normal')->order('weigh desc')->select();

// 分页
Db::name('article')->where('status', 'normal')->paginate(20);

// 插入
Db::name('article')->insert(['title' => '标题', 'createtime' => time()]);

// 更新（必须带 WHERE）
Db::name('article')->where('id', 1)->update(['title' => '新标题']);

// 软删除（deletetime 方案）
Db::name('article')->where('id', 1)->update(['deletetime' => time()]);

// 事务
Db::transaction(function () {
    Db::name('order')->insert($order);
    Db::name('order_item')->insertAll($items);
});
```

### 模型约定

```php
namespace app\admin\model;

use think\Model;
use traits\model\SoftDelete;

class Article extends Model
{
    use SoftDelete;

    protected $name = 'article';
    protected $deleteTime = 'deletetime';
    protected $autoWriteTimestamp = true;
    protected $createTime = 'createtime';
    protected $updateTime = 'updatetime';

    // 金额字段类型转换，保留 4 位小数
    protected $type = [
        'total_amount'    => 'decimal:4',
        'pay_amount'      => 'decimal:4',
        'discount_amount' => 'decimal:4',
        'pay_fee'         => 'decimal:4',
    ];
}
```

| 配置项 | FastAdmin 推荐值 |
|--------|------------------|
| autoWriteTimestamp | `true` 或 `'int'` |
| createTime | `createtime` |
| updateTime | `updatetime` |
| deleteTime | `deletetime`（启用 SoftDelete trait） |

### 关联预载入

```php
Article::with(['category', 'user'])->where('status', 'normal')->select();
```

`user_id` 字段 CRUD 默认匹配 `user/index` 控制器；关联数据不显示时检查视图 `data-source` 配置。

---

## Part 9: SQL 与 CRUD 工作流

### SELECT

```sql
-- 指定字段，避免 SELECT *
SELECT id, title, status FROM fa_article WHERE status = 'normal';

-- 分页
SELECT id, title FROM fa_article WHERE status = 'normal' LIMIT 0, 20;
```

### 逻辑删除 vs 物理删除

```sql
-- 逻辑删除（FastAdmin 回收站）
UPDATE fa_article SET deletetime = UNIX_TIMESTAMP() WHERE id = 1;

-- 物理删除（关联表、中间表慎用）
DELETE FROM fa_user_group WHERE user_id = 1;
```

### CRUD 生成流程

1. 按本规范设计并创建数据表
2. 执行 `php think crud -t fa_表名`（或 `-t 表名`，视配置前缀而定）
3. 修改表结构或新增字段后，需**重新生成 CRUD** 或手动改视图/JS
4. 插件 SQL 中禁止 `DROP TABLE`；尽量不修改框架自带表结构

---

## Part 10: 设计检查清单

- [ ] 表名、字段名全小写英文 + 下划线，无拼音
- [ ] 主键为单字段 `id`
- [ ] 使用 InnoDB + utf8mb4_general_ci
- [ ] 时间字段用 int(10) Unix 时间戳，以 `time` 结尾
- [ ] 含 `createtime`、`updatetime`；需回收站则加 `deletetime`（默认 NULL）
- [ ] 需排序则加 `weigh`；需 TAB 筛选则加 `status` enum
- [ ] 关联字段以 `_id` / `_ids` 结尾并已建索引
- [ ] 字段 COMMENT 含中文说明；enum 字段用 `键=值` 格式便于 CRUD 字典
- [ ] 每个字段、表均有 COMMENT
- [ ] 外键与高频筛选字段已建索引，单表索引 ≤ 5
- [ ] 插件 install.sql 使用 `__PREFIX__`，不含 DROP TABLE
- [ ] 模型配置与表字段时间戳、软删除字段一致
- [ ] 金额字段使用 DECIMAL(19,4)，默认 0.0000，禁止 FLOAT/DOUBLE
- [ ] 模型中为金额字段配置 `decimal:4` 类型转换，PHP 侧用 bcmath 运算
