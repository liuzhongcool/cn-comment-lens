[English](README.md) | 简体中文

<p align="center">
  <img src="assets/pluginIcon.svg" width="88" height="88" alt="CN Comment Lens · 了然（隶书）">
</p>

# CN Comment Lens

> 中文名「**了然**」，取「一目了然」之意——注释该在的地方，一眼就能看到。

在代码行尾直接显示字段、常量、方法参数的**中文注释**，免去在多个类文件之间跳转查看注释的麻烦。

```java
order.setCustomerName(name);              // customerName:客户姓名
order.setStatus(OrderStatus.UNPAID);      // 待支付 = UNPAID:待支付
submit(order.getOrderNo());               // 订单编号，全局唯一 = orderNo:订单编号
```

- 插件 ID：`com.jzt.international.cncomment` ｜ 版本：`2.6.3-261`
- 支持 IDE：IntelliJ IDEA、PhpStorm（同一份插件包，语言能力按需启用）
- 界面语言：中文 / English（跟随 IDE 语言，也可在设置里指定）
- 📖 [使用说明文档](https://www.kdocs.cn/l/cjINZ8uJoVkl)（仓库内副本：[docs/user-guide_zh.md](docs/user-guide_zh.md)）：逐个功能讲「在哪里、怎么开、看到什么算正常」

---

## 一、插件介绍

这个插件做的只有一件事：把「翻到声明处读注释」变成「在当前这行就看到注释」。

- **行尾中文注释**：字段、枚举常量、类常量、方法与构造器形参的注释贴在代码行尾；inlay 不进入文本流，行尾输入与后缀补全不受影响。
- **Project 视图文件注释**：文件名后追加该文件的说明（由注解 / 文档注释 / 行注释聚合而来）。
- **悬停看完整注释**：在引用处 / 调用点（`obj.getPayNo()`、赋值、`OBJ.FIELD`）悬停，显示不截断的完整注释、来源（注解 / 文档注释 / 上方行注释 / 行尾注释）与所在行号，可点「打开注释原文」直接跳过去；声明处交回平台原生文档。
- **注释覆盖率检查与报告**：类与 public 成员缺中文注释时告警，`Alt+Enter` 一键生成注释骨架；报告按目录 / 包名过滤扫描，导出 Markdown（总览 / 分语言 / 分包小计）。
- **数据库 Schema 反查（PHP 查询构造器 / MyBatis Mapper）**：字符串列名显示列注释与类型——来源可以是项目内 `.sql` 建表语句（📦），也可以是 IDE Database 工具窗里已连好的数据源（🛢，零凭据、不用另填连接）；Mapper XML 里写 `别名.` 补全该表字段，悬停按数据源分行展示；有多个来源时点来源行的注释可按来源跳转，字段本身可点击跳到该列的定义处（`.sql` 列级来源行或只读 DDL 视图列行）。库候选与你在 Database 工具窗里内省过的库一致，并排除 `information_schema` / `mysql` / `performance_schema` / `sys` 四个系统库；筛选与探测**装完即用**——打开项目即自动生效，不必先打开设置页点「应用」。
- **表名与取值补全（PHP）**：`Db::name / table('…')`、`model('…')`、`db('…')` 补全表名（写短名时按「表前缀」剥离后的名字匹配、也插入短名）；`where('status', …)` 的取值位置补全模型常量 / enum case 与列注释里写明的取值。
- **展示与样式可调**：展示范围（仅当前行 / 当前方法 / 整个文件）、大文件保护、标识符与注释各自的字体 / 字号 / 颜色、赋值符号、分隔符、注释截断长度（按**汉字个数**计，整条没有汉字时按字符数）。
- **中英双语界面**与**配置管理**（全部设置导出 / 导入 JSON、按设置组恢复默认）。

### 支持的语言与 IDE

| 环境 | 行尾提示 | Project 视图注释 | 设置子页 | 覆盖率检查 |
|---|---|---|---|---|
| IDEA（未装 PHP 插件） | Java ✅ / Kotlin ✅ / PHP ❌ | Java ✅ / Kotlin ✅ | 主 + Project 视图 + Java + Kotlin | Java + Kotlin |
| IDEA（装了 PHP 插件） | Java ✅ / Kotlin ✅ / PHP ✅ | Java ✅ / Kotlin ✅ / PHP ✅ | 全部（Go 除外） | Java + PHP + Kotlin |
| IDEA（装了 Go 插件） | 上述 + Go ✅ | 上述 + Go ✅ | 上述 + Go | 上述 + Go |
| PhpStorm | Kotlin ✅ / PHP ✅ / Java ❌ | Kotlin ✅ / PHP ✅ | 主 + Project 视图 + PHP + Kotlin | PHP + Kotlin |
| WebStorm 等其它 IDE | 都无 | 都无 | 主 + Project 视图 | 无 |

- Java、PHP、Go 都是**可选依赖**：IDE 缺哪个语言插件，就只启用其余语言，插件本身不受影响
- **Kotlin** 是 IDEA / PhpStorm 的内置插件（`org.jetbrains.kotlin`），装了即启用；**Go** 插件需从 Marketplace 安装

### 注释来源（命中即用）

| 语言 | 来源优先级 |
|---|---|
| Java | 注解值（Swagger 系 / 自定义）→ JavaDoc 摘要 → 相邻 / 行尾注释 |
| Kotlin | docblock 标签 → KDoc 摘要 → 相邻 / 行尾注释（标准库默认排除） |
| PHP | Attribute / docblock 标签（含 `self::CONST` 常量引用）→ PHPDoc 摘要 → 相邻 / 行尾注释 |
| Go | docblock 标签 → godoc → 行尾注释 |

调用点的实参说明取自 `@param`；形参本身没有说明时，回退为被调方法 / 类自身的注释。所有结果都可用「仅显示含中文的注释」过滤，并写入项目级缓存。

### 已知限制

- 首次打开项目要等索引完成，行尾提示才会出现；Diff 视图里不显示提示
- PHP：Attribute 常量引用只支持 `self::` 与本类短名
- Kotlin：枚举条目与 object 声明不在提示与覆盖率口径内
- Go：覆盖率只统计导出成员（结构体字段支持提示、暂不计入覆盖率）
- 反查：变量表名、子查询与闭包 `where` 会中断锚点回溯；join 只用于建立「别名 → 表」映射（不做 ON 条件与多表推断）；索引反映项目内 `.sql` 的当前内容，不是数据库实况；实时源只能看到已在 Database 工具窗展开（内省）的表 / 列；MyBatis 只解析同一语句块内的字面量表名（`<include>` 片段、动态标签与 `${}` 拼接的表名不猜）；表名补全在没输入前缀时最多返回 500 条候选

## 二、快速上手

1. **安装**：`Settings | Plugins` 在 Marketplace 搜索 `CN Comment Lens` 安装；离线安装走右上角 `⚙ | Install Plugin from Disk…` 选发布包 zip，然后重启 IDE。
2. **首次打开项目**：行尾提示等索引完成后出现；每个插件版本首次打开会弹一次引导通知（可直接进设置）；完全没有中文注释的文件会在编辑器顶部说明「为什么行尾没有提示」，点一次即永久关闭。
3. **开关与快捷键**：Tools 菜单或编辑器右键菜单 → `CN Comment Lens`，行尾注释与 Project 视图注释是两个独立开关；快捷键 `Ctrl+Alt+Shift+K`（行尾提示）、`Ctrl+Alt+Shift+P`（Project 视图注释），可在 `Settings | Keymap` 改。
4. **覆盖率检查**：默认开启，路径 `Settings | Editor | Inspections | CN Comment Lens`；报告走 Tools 菜单 `CN Comment Lens → 注释覆盖率报告`，或在 Project 视图右键目录 / 包直接扫描所选范围。
5. **数据库反查**：打开 `Settings | Tools | CN Comment Lens | 数据库字段反查` 的总开关（默认已开）→ 按需填写「表前缀」与「DDL 路径模式」→ 选「数据源策略」（仅 `.sql` / 仅数据库 / 混合）。设置**装完即用**：打开项目即按已保存的筛选生效并在后台探测一次活数据源，不必先点「应用」（「立即探测 / 立即刷新」仍在，可随时手动重来）；策略选「仅 `.sql`」时活库相关项会自动置灰。用实时数据库时，先在 Database 工具窗连好并展开目标库。行尾提示中 📦 = 来自 `.sql`，🛢 = 来自实时数据库，点击即可跳到该列的定义处。

## 三、插件设置

设置入口：`Settings | Tools | CN Comment Lens`，最多 8 个页面（Java / PHP / Kotlin / Go 四个语言页按 IDE 是否具备该语言插件出现）。

| 页面 | 主要设置项 |
|---|---|
| CN Comment Lens（主页面） | 插件语言、提示的展示范围、启用行尾中文注释、仅显示含中文的注释、始终显示标识符前缀、多行注释最多取前几行、单条注释最大长度（按汉字个数）、文件行数上限、多条注释的分隔符、赋值符号、标识符 / 注释各自的字体 · 字号 · 颜色 |
| Project 视图文件注释 | 在文件名后显示注释、注释颜色、最大汉字数、取前 N 行、跳过纯英文、文件注释自定义注解全限定名与属性名 |
| Java | 方法参数、POJO 字段、枚举常量、普通常量、读取 Swagger 注解、自定义注解全限定名与属性名、包名黑 / 白名单 |
| PHP | 方法形参、类属性、类常量、自定义标签 / Attribute 名、命名空间黑 / 白名单 |
| Kotlin | 方法参数、属性引用、自定义标签名、包名黑 / 白名单（默认排除 `kotlin.*` / `kotlinx.*`） |
| Go | 字段 / 常量引用、自定义标签名、包路径黑 / 白名单（Go 无 Swagger 与枚举项） |
| 数据库字段反查 | 总开关（默认开）、悬停延迟、反查注释截取长度（按汉字个数）；**.sql 文件（本地脚本）** 组：DDL 路径模式、增量脚本时序正则、增量脚本目录；**真实数据源（活库）** 组：数据源策略、快照刷新间隔、存活探测间隔、只扫描这些数据源、只扫描这些库、表前缀；「立即探测 / 立即刷新 / 重新扫描 Schema / 数据表列表」与索引状态；Schema 反查说明 |
| 配置管理 | 全部设置导出 / 导入（JSON，覆盖式导入先确认）、按设置组恢复默认 |

所有设置 Apply 后立即生效（清缓存 + 重跑行尾提示 + 刷新 Project 视图），无需重启 IDE；反查的数据源 / 库筛选与探测在**打开项目时自动生效**，不需要手动 Apply。

设置树里的页面名称只按 **IDE 语言**解析（平台的限制），插件自己的「插件语言」改不了它——想让菜单名也是中文，请把 IDE 语言一并设为中文。

## 四、鸣谢

本插件的部分功能受 **Show Comment** 插件启发，特此致谢。
