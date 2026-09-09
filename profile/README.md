<div align="center">

# 🧙 LotroCompanionCN

**LOTRO Companion 中文社区** — [Lotro Companion](https://github.com/LotroCompanionCN/lotro-companion) 等 LOTRO 相关工具的中文本地化与维护组织。

---

## 📦 最新发布版 (lotro-companion-release)

> `lotro-companion-release`

**最新版本：**

[⬇️ 前往 Releases 页面下载](https://github.com/LotroCompanionCN/lotro-companion-release/releases/latest)

</div>


# LOTRO Companion（中文本地化构建）

本仓库是 [LotRO Companion](https://github.com/LotroCompanion/lotro-companion-doc) 的一个**中文本地化 + 定制构建**仓库。
LotRO Companion 是《指环王OL》(Lord of the Rings Online, LOTRO) 的桌面辅助工具，用于角色规划、状态跟踪与游戏百科查询。

本仓库在官方项目基础上做了本地化适配，并重新组织了 Maven 多模块构建与发布（release）打包流程。

## 主要功能

- 角色状态跟踪（声望、制造、事迹/成就、美德、特性树等）
- 角色属性规划与模拟（配装 / essence / virtue 规划）
- 游戏数据百科浏览（任务、事迹、配方、头衔、表情、怪物、地图等）
- 时间曲线 / 统计图表（如角色升级、制造熟练度、声望历史）
- 界面与游戏内数据支持中文本地化（`Labels_zh` 等资源），统计图等 JFreeChart 图表亦适配中文字体

## 模块概览

仓库根聚合工程（`pom.xml`）的 reactor 构建模块：

| 模块 | 说明 |
| --- | --- |
| `lotro-core-overrides` | 运行时补丁 jar：提供 gzip 数据解析、I18n/本地化及 `delta-lotro-memory` 内存偏移资源覆盖，打包时放入 `lib/patches` 以优先于原类/资源加载（含 `tools/weekly.ps1` 每周适配工具） |
| `lotro-values` | 通用数值工具（内部基础模块） |
| `lotro-core` | 核心数据/逻辑层：数据库对象、角色状态、统计、游戏数据模型 |
| `lotro-maps` | 地图数据与地图界面支持 |
| `lotro-data-extractor` | 游戏数据提取工具 |
| `lotro-jukebox-core` | Jukebox（音乐）核心模块 |
| `lotro-companion` | 桌面应用主模块（GUI + 本地化资源） |
| `lotro-data` | 生成的 lore 数据（XML/BIN，发布时 gzip） |
| `lotro-icons` | 图标资源 |
| `lotro-items-db` | 物品数据库 |
| `lotro-companion-release` | 发布装配模块：产出可运行的 `target/app/` 目录 |

仓库内另有若干不在根 reactor 中、属于旧版工具/数据集的目录：`lotro-companion-distrib`（旧 JavaFX 打包脚本）、`lotro-tools`、`lotro-deeds-db`、`lotro-item-icons-db`、`lotro-maps-db`、`lotro-relics`、`lotro-jukebox` 等。

## 构建与打包

主应用与发布产物均通过 Maven 构建（聚合工程根 `pom.xml`）。

```bash
# 全量 clean install（跳过测试）
mvn clean install -DskipTests
```

> 注意：release 模块在 `verify` 阶段会运行 Nashorn 脚本压缩 lore 数据并转换地图图片，
> 需要 **JDK 11+**（推荐 17）。JDK 8 会因 nashorn-core 版本过旧而失败。

打包成功后，发布目录为：

```
lotro-companion-release/target/app/
  ├── run.bat            # 启动脚本（Windows）
  ├── runtime/           # 内置精简 JRE（预构建，JDK 21）
  ├── lib/               # 应用及其依赖 jar
  │   ├── patches/       # 覆盖补丁（lotro-core-overrides）
  │   └── ...
  └── data/lore/         # 游戏数据（gzip 压缩）
```

可用 `target/app/run.bat` 直接运行应用。

## 本地化与中文字体说明

- 界面文案位于主模块的 `_zh.properties` 资源（如 `Labels_zh.properties`）。
- 程序启动时会注册并优先选用支持中文的字体（微软雅黑 / 思源黑体等）。
- JFreeChart 统计图表单独把 CJK 字体应用到标题、坐标轴、图例（`JFreeChartFonts`），避免中文显示为方块。
- `lotro-core-overrides` 通过 `lib/patches` 以更高优先级覆盖原始 `delta-common` 的 XML 解析类，使读取器透明支持 `.gz` 数据；并遮蔽 `delta-lotro-memory` 的 `64bits.properties` 资源，从而无需 fork 上游 jar 即可适配每周更新的客户端偏移（见 `lotro-core-overrides/README.md`）。
