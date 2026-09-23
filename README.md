English | [简体中文](README_zh.md)

<p align="center">
  <img src="assets/pluginIcon.svg" width="88" height="88" alt="CN Comment Lens · 了然（隶书 / clerical script）">
</p>

# CN Comment Lens

> The plugin's Chinese name is **了然** (liǎorán), from 一目了然 — "clear at a glance".

Shows **Chinese comments** for fields, constants and method parameters right at the end of code lines — no more jumping between files just to read a comment.

```java
order.setCustomerName(name);              // customerName:客户姓名
order.setStatus(OrderStatus.UNPAID);      // 待支付 = UNPAID:待支付
submit(order.getOrderNo());               // 订单编号，全局唯一 = orderNo:订单编号
```

- Plugin ID: `com.jzt.international.cncomment` | Version: `2.6.4-261`
- Supported IDEs: IntelliJ IDEA and PhpStorm — one package, language capabilities enabled on demand
- UI language: 中文 / English (follows the IDE, or can be set manually)
- 📖 [User guide](https://www.kdocs.cn/l/cjINZ8uJoVkl) (Chinese; in-repo copy: [docs/user-guide_zh.md](docs/user-guide_zh.md)): for every feature, where it lives, how to turn it on, and what "normal" looks like

---

## 1. Introduction

The plugin does one thing: it turns "jump to the declaration to read the comment" into "read the comment right here".

- **Inline Chinese comments**: comments of fields, enum entries, class constants, method and constructor parameters are rendered at the end of the code line; inlays never enter the text buffer, so typing at the end of a line and postfix completion are unaffected.
- **Project view file comments**: a description appended after file names (aggregated from annotations / doc comments / line comments).
- **Hover the full comment**: hovering a **reference or call site** (`obj.getPayNo()`, an assignment, `OBJ.FIELD`) shows the untruncated comment with its source (annotation / doc comment / line comment above / trailing comment) and the line number, plus a clickable "Open comment source" that jumps there; declarations keep the platform's own documentation.
- **Comment coverage inspections & report**: classes and public members missing a Chinese comment are flagged, with a one-click comment-skeleton fix; the report scans with directory / package filters and exports Markdown (overview / per language / per package).
- **Database schema lookup (PHP query builders / MyBatis mappers)**: string column names show their column comment and type — from the project's own `.sql` DDL statements (📦) or from data sources already connected in the IDE Database tool window (🛢, zero credentials, nothing to configure here); typing `alias.` in a mapper XML completes that table's columns and hovering shows one line per data source; with several sources, clicking the comment on a source line jumps to that source, and the column reference itself jumps to where the column is defined (the `.sql` column-level source line or the read-only DDL view row). The candidate database list mirrors what you introspected in the Database tool window and leaves out `information_schema` / `mysql` / `performance_schema` / `sys`; filters and probing work **out of the box** — they take effect when a project opens, with no trip to the settings page.
- **Table & value completion (PHP)**: `Db::name / table('…')`, `model('…')` and `db('…')` complete table names (a short name is matched after stripping the configured table prefix and inserted short as well); the value position of `where('status', …)` completes model constants / enum cases and the values spelled out in the column comment.
- **Adjustable display & style**: hint scope (current line / current method / whole file), large-file guard, separate font / size / color for identifiers and comments, assign symbol, separator, comment truncation length (counted in **Chinese characters**; a comment without any Chinese character falls back to plain character count).
- **English & Chinese UI** and a **Configuration** page (export / import every setting as JSON, per-group reset).

### Languages & IDEs

| Environment | Inline hints | Project view comments | Settings pages | Coverage inspections |
|---|---|---|---|---|
| IDEA (no PHP plugin) | Java ✅ / Kotlin ✅ / PHP ❌ | Java ✅ / Kotlin ✅ | Main + Project view + Java + Kotlin | Java + Kotlin |
| IDEA (with PHP plugin) | Java ✅ / Kotlin ✅ / PHP ✅ | Java ✅ / Kotlin ✅ / PHP ✅ | All (except Go) | Java + PHP + Kotlin |
| IDEA (with Go plugin) | Above + Go ✅ | Above + Go ✅ | Above + Go | Above + Go |
| PhpStorm | Kotlin ✅ / PHP ✅ / Java ❌ | Kotlin ✅ / PHP ✅ | Main + Project view + PHP + Kotlin | PHP + Kotlin |
| WebStorm and other IDEs | None | None | Main + Project view | None |

- Java, PHP and Go are **optional dependencies**: a missing language plugin only disables that language, the rest keeps working
- **Kotlin** is a bundled plugin of IDEA / PhpStorm (`org.jetbrains.kotlin`) — enabled as soon as it is present; the **Go** plugin must be installed from the Marketplace

### Comment sources (first match wins)

| Language | Priority |
|---|---|
| Java | annotation values (Swagger / custom) → JavaDoc summary → adjacent / trailing comments |
| Kotlin | doc tags → KDoc summary → adjacent / trailing comments (stdlib excluded by default) |
| PHP | attributes / doc tags (incl. `self::CONST` constant references) → PHPDoc summary → adjacent / trailing comments |
| Go | doc tags → godoc → trailing comments |

Call-site argument descriptions come from `@param`; when an argument itself has no description, the callee's / class's own comment is used instead. Every result can be filtered by "Chinese comments only" and is cached per project.

### Known limitations

- Hints appear only after indexing finishes on first launch; no hints inside diff viewers
- PHP: attribute constant references are limited to `self::` and the own class short name
- Kotlin: enum entries and object declarations are out of scope for hints and coverage
- Go: coverage covers exported members only (struct fields get hints but are not counted yet)
- Schema lookup: variable table names, sub-queries and closure `where` abort the anchor walk; joins only build an "alias → table" map for `alias.col` (no ON-condition / multi-table inference); the index reflects the project's `.sql` files, not the live database; the live source only sees tables / columns already expanded (introspected) in the Database tool window; MyBatis resolves only literal table names inside the same statement block (`<include>` fragments, dynamic tags and `${}` concatenation are not guessed); table completion returns at most 500 candidates when no prefix has been typed

## 2. Getting started

1. **Install**: `Settings | Plugins` → search `CN Comment Lens` in the Marketplace; for an offline install use `⚙ | Install Plugin from Disk…` with the release zip — then restart the IDE.
2. **First project**: hints show up once indexing finishes; the first project opened with each plugin version gets a one-time welcome notification (it links straight to the settings); a file containing no Chinese comment at all gets a banner at the top explaining the silence — one click turns it off for good.
3. **Toggles & shortcuts**: Tools menu or editor popup → `CN Comment Lens`; inline comments and Project view comments are two independent switches. Shortcuts: `Ctrl+Alt+Shift+K` (inline hints) and `Ctrl+Alt+Shift+P` (Project view comments), rebindable under `Settings | Keymap`.
4. **Coverage**: inspections are on by default under `Settings | Editor | Inspections | CN Comment Lens`; the report runs from the Tools menu (`CN Comment Lens → Comment coverage report`) or by right-clicking a directory / package in the Project view to scan just that scope.
5. **Schema lookup**: turn on the master switch under `Settings | Tools | CN Comment Lens | Database field lookup` (on by default) → fill in the **table prefix** and **DDL path patterns** as needed → pick a **data source strategy** (.sql only / Database only / Mixed). Settings work **out of the box**: a project open applies the saved filters and probes the live source once in the background, so there is no Apply step ("Probe now" / "Refresh now" stay available for manual runs); live-source options are greyed out while the strategy is ".sql only". To use the live database, connect and expand your databases in the Database tool window. In a hint, 📦 means the comment came from `.sql` and 🛢 means it came from the live database; clicking it jumps to where that column is defined.

## 3. Settings

Entry point: `Settings | Tools | CN Comment Lens`, up to 8 pages (the Java / PHP / Kotlin / Go pages appear only when the IDE has that language plugin).

| Page | Main options |
|---|---|
| CN Comment Lens (main) | Plugin language, hint display scope, enable inline Chinese comments, only show comments containing Chinese, always show identifier prefix, max lines of multi-line comments, max comment length (counted in Chinese characters), max file lines, separator for multiple comments, assignment symbol, font / size / color for identifiers and comments |
| File Comments | show comments after file names, color, max Chinese characters, first N lines, skip pure English, custom annotation FQNs and attribute names for file comments |
| Java | method parameters, POJO fields, enum constants, plain constants, read Swagger annotations, custom annotation FQNs and attribute names, package black / white lists |
| PHP | method parameters, class properties, class constants, custom tag / attribute names, namespace black / white lists |
| Kotlin | method parameters, property references, custom tag names, package black / white lists (`kotlin.*` / `kotlinx.*` excluded by default) |
| Go | field / constant references, custom tag names, package path black / white lists (Go has no Swagger or enum items) |
| Database field lookup | master switch (on by default), hover delay, comment truncation length (in Chinese characters); **.sql files (local scripts)**: DDL path patterns, incremental order patterns, incremental script directories; **live data sources**: data source strategy, snapshot refresh interval, liveness probe interval, only scan these data sources, only scan these databases, table prefix; plus Probe now / Refresh now / Rescan schema / table list and the index status; schema lookup help |
| Configuration | export / import every setting as JSON (import overwrites and confirms first), reset one settings group to its defaults |

Every setting takes effect after Apply (cache cleared + hints re-run + Project view refreshed) — no IDE restart needed; the lookup filters (data source / database) and probing apply when a project opens, with no manual Apply.

Page names in the settings tree are resolved from the **IDE language** only (a platform constraint); the plugin's own "plugin language" cannot change them — set the IDE language to Chinese if you want Chinese page names.

## 4. Acknowledgements

Some features of this plugin were inspired by the **Show Comment** plugin. Many thanks to its author.
