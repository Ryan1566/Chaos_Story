# Story/ — 项目企划与设计工作区（文档真相源）

> **仓库布局（2026-10-05 起）**：本目录是**独立 git 仓库 `Chaos_Story`** 的内容目录。
> - 仓库根：`Assets/Chaos_Story/`（remote: `git@github.com:Ryan1566/Chaos_Story.git`，分支 `main`）
> - 文档根：`Assets/Chaos_Story/Story/`（即本 README 所在目录）
> - 因此**本文档内所有相对路径一律以「Chaos_Story 仓库根」为基准**，写作 `Story/03_配置表设计/…`。
>   这与 2026-10-05 之前以主仓库 `Assets/` 为基准的写法**字面完全一致**，故既有 169 处引用无需改写。

本目录是 Project_Chaos 的**设计文档唯一真相源**。代码实现必须以本目录的文档与
`Assets/Excel/Chaos_excel/` 下的配置表为准；文档与代码冲突时，先改文档、再改代码。

> 本目录由 Lead 建立骨架并维护本 README。各子目录的**写权限**归属见下表，
> 除非该目录的 owner 明确交接，不要跨目录写入。

## 1. 目录职责与归属

| 目录 | 装什么 | Owner |
| --- | --- | --- |
| `01_企划/` | 游戏总体企划：世界观、核心循环、玩法总览、目标平台与范围 | 策划 |
| `02_系统策划/` | 各系统详细设计：规则、数值、状态机、交互流程、UI 表现要求 | 策划 |
| `03_配置表设计/` | 每张配置表的**字段规格说明**：字段名 / 类型 / 取值 / 默认值 / 关联表 / 用途 | 策划 |
| `04_架构/` | 架构现状核查、目标文件结构、路径与命名规范、MVP 契约、落地方案与可行性/优化结论 | 架构师 |
| `05_程序/` | 框架能力清单、既有 API 使用范式、实现记录、技术备忘 | 程序 |

## 2. 文档命名约定（所有子目录统一）

```
NN_主题.md          例：01_核心循环.md、03_战斗系统.md、04_MVP契约.md
```

- `NN` 为两位序号，决定阅读顺序；同目录内序号唯一。
- 一份文档只讲一件事；拆细优于写长。
- 每份文档头部必须有元信息块：

```markdown
> 状态：草稿 / 评审中 / 已定稿 / 已实现
> Owner：策划 | 架构师 | 程序
> 最后更新：YYYY-MM-DD
> 关联：<相关文档或配置表路径>
```

- 文档之间的引用一律写**仓库相对路径**（以 **Chaos_Story 仓库根 `Assets/Chaos_Story/`** 为基准，
  即形如 `Story/04_架构/01_架构现状核查.md`），不要写绝对路径。
- 定稿的文档不要再改语义；要改就改状态并注明变更点，避免代码实现时对不上。

## 3. 配置表（Excel）约定

- 源表目录：`Assets/Excel/Chaos_excel/`，文件命名 `英文表名_中文说明.xlsx`。
- 类名 / Json 名由**文件名第一个下划线前的英文部分**推导，因此该段必须唯一且是合法 C# 标识符。
- 行约定（导出器硬编码，不可自行改动行号）：

| 行 | 用途 |
| --- | --- |
| 第 1 行 | 类名（预留，仅人工查看） |
| 第 2 行 | 是否读取（预留，导出器**未实现**，纯装饰） |
| 第 3 行 | 中文备注 → 生成 C# 字段的行尾注释 |
| 第 4 行 | 英文属性名 → 生成字段名与 Json key（**不可为空，唯一强校验**） |
| 第 5 行 | 端标记 → 导出器**已消费**：填 `server` 会把该列从 C# 类和 Json 中**整列丢弃**；填**空**只从 Json 丢弃（字段还在类里，值永远是默认值）；只能填小写 `client` / `both` / `server` |
| 第 6 行 | 类型 → 决定 C# 字段类型与 Json 转换方式（受白名单限制） |
| 第 7 行 | **第一条正式数据**（不是示例行 —— 取值循环 `for (row = 7; ...)` 从本行开始） |
| 第 8 行起 | 后续正式数据 |

> ⚠️ **2026-10-05 Lead 复核更正**：本节此前照抄 `Assets/Learn/Excel配置表工具使用说明.docx`，
> 其中「第 5 行未被消费」「第 7 行是示例行、不会被导出」**都是过时说法**。
> 已由源码与磁盘产物双重复核（`ConfigManager.cs` L52 `valueIndex = 7`、L297 `for (int row = valueIndex; ...)`、
> L160 / L191 端标记过滤；产物 `Resources/Data/Json/Runtime/TestTableConfig.json` 为 4 条，
> 对应 xlsx 第 7~10 行，且缺 `speed` 列 —— 因该列表头为 `server`）。
> 细节见 `Story/03_配置表设计/01_配置表填写规范.md`。

- 类型（第 6 行）：**唯一权威是 `ConfigManager.Convert` 的 `case` 列表**。
  当前可用集合、新增类型的流程与实测依据全部见 **第 7 节**；**本节不再维护类型清单**，以免与第 7 节不同步。
  - `enum` / `DateTime` 一律不支持。
  - `decimal` **严禁使用**（`JsonUtility` 静默丢弃，不报错）；`char` 暂不纳入（JSON 里存成数字）。
  - ⚠️ 2026-10-05 前的旧说法「**不要**填 `bool`」**已作废**：`bool` 已按用户裁决纳入（见第 7 节）。
- 只读第 1 个页签；中途不能有空列；表尾不能留空行。
- 参考模板：`Assets/Excel/Chaos_excel/TestTable_测试表.xlsx`（5 列，含 uint / string / float）。
- 详细规则见 `Assets/Learn/Excel配置表工具使用说明.docx`。

### 3.1 谁可以动 Excel 与生成物

| 动作 | 允许的人 |
| --- | --- |
| 新建 / 编辑 `Excel/Chaos_excel/*.xlsx` | 策划 |
| 新建**技术夹具表**（如类型探针表 `TypeProbe_类型探针表.xlsx`，**当前不存在、需要时重建**） | **程序**（2026-10-05 Lead 授权的例外：此类表不是策划配置，是导出器的回归夹具；仍须遵守 7 行表头约定，且**第 5 行每列都要填**，否则会出现「类里有字段、Json 里没 key」的假阴性） |
| 改 `TestTable_测试表.xlsx` | 仅 Lead 批准后（它是对照模板） |
| 执行 Unity 菜单 `ExcelTool → ExportExcelConfigs / ExportExcelModels` | **程序**（或 Lead） |
| 改 `MVP/Model/ConfigData/*.cs`、`MVP/Model/ModelData/*.cs`、`Resources/Data/Json/**` | **禁止手改**，一律由导出器生成 |

## 4. 协作流程

```
  用户需求
     │
     ▼
① 策划：写 01_企划 / 02_系统策划 文档 + 03_配置表设计 字段规格 + Excel 源表
     │
     ▼
② 架构师：落地方案评审（可行性、性能、可维护性）+ 文件路径与命名规划
     │  ├─ 有异议 ──► 退回策划改文档 / 改字段 ──┐
     │  └─ 通过                                 │
     │                                          │
     ▼                                          │
③ 架构师：输出 04_架构 落地方案，明确「改哪些文件、按什么顺序」◄──┘
     │
     ▼
④ 程序：按 02/03 的文档与表 + 04 的落地方案实现，产出写进 05_程序/ 记录
```

**硬性闸门**：程序只有在「策划文档已定稿」**且**「架构师落地方案已定稿」之后才动手写业务逻辑。
提前写代码属于返工风险，一律不做。

> **例外：轻量任务走快速通道** —— 满足 §8 全部条件时，**只派程序一个人**，不通知架构师与策划、不建自测指针、不走闸门。
> 判定清单与反例见 **§8**。

## 5. 开工前必读（防止踩坑）

1. **C# 文件是混合编码**：`Scripts/` 下 91 个 `.cs` 中，37 个是 GBK(936)、31 个带 UTF-8 BOM、23 个是无 BOM UTF-8（逐字节扫描确认）。
   用通用文本工具改 GBK 文件会把中文注释写成乱码。改 `.cs` 前先确认编码，保持原编码写回。
2. **生成物不要手改**：`ConfigData/` `ModelData/` 与 `Resources/Data/Json/` 下的产物每次导出都会被覆盖。
3. **`MVP/Presenter/` 目前是空目录**，MVP 的 P 层契约尚未确定。
4. Unity 编辑器在线（2022.3.57f1c2，MCP 端口 7890），改写脚本后必须让编辑器重新编译并确认 0 error。
5. **版本控制现状（2026-10-05）**：
   - Story/ 是**独立 git 仓库 `Chaos_Story`**（`Assets/Chaos_Story/`，remote `Ryan1566/Chaos_Story`，分支 `main`），
     16 份文档 + 21 个 `.meta` 已提交并推送。
   - 主仓库 `.gitignore` 已由**用户于 2026-10-05 15:58 添加** `Assets/Chaos_Story` 与 `Assets/Chaos_Story.meta`
     （旧条目 `Assets/Story` 仍在但已失效）→ 嵌套仓被误加成 gitlink 的风险**已消除**。
   - ⚠️ **不要以为「`git status` 干净」就等于文档安全**：本目录随时可能有**未提交的在途改动**
     （2026-10-05 首次统一提交前一度累积到 15 份）。判断「文档是否已落地」要看
     `git -C Assets/Chaos_Story status`，不是看文件是否存在；**也别忘了在收口时显式提交**。
   - **提交约定**：文档改动在 `Assets/Chaos_Story/` 内**单独提交**（`git -C Assets/Chaos_Story ...`，显式路径），
     不要与主仓库的代码改动混在同一个提交里。
6. **源表目录里绝对不能放空页签的表** —— 已由 Lead 在 Unity 内用 EPPlus 4.5.3 实测（2026-10-05）：

   | 源表情形 | EPPlus `Dimension` | 导出器实际行为 |
   | --- | --- | --- |
   | 页签**完全空白** | `null` | `ConfigManager.cs:88`（及 `:124`）执行 `Dimension.End.Row` → **NullReferenceException**，被 `:99` catch 后**中断整批导出**，其后所有表都不导出（有 error 日志） |
   | 只有表头（1~6 行）无数据 | `End.Row = 6` | 通过 `:88`；`GetValues` 返回空 → `:205` 打日志「该表无任何导出项」后 **只跳过本表**，不影响后续表 |
   | 有数据 | `End.Row >= 7` | 正常导出，**第 7 行即第一条数据** |

   注：`if (workSheet.Dimension.End.Row == 0) return;` 这个守卫本身是错的 —— 真正空页签时 `Dimension` 是 `null`，
   永远不会等于 0，所以 `return` 分支实际不可达，真正的失败是 NRE；且应为 `continue` 而非 `return`。
   `FileUtil.LoadFiles` 用 `directory.GetFiles("*")`，枚举顺序不确定 → **「哪些表没导出」是不确定的**。
   修复排期见 `Story/04_架构/`（架构师 P1-4b）。
7. **在 Unity 运行时改 `.cs` 必须显式重编译**（2026-10-05 由程序踩坑后确认）：
   用 `File.WriteAllText` / `unity_execute_code` 从外部写 `.cs` **不会触发 `AssetDatabase` 重编译**，
   必须再调一次 `AssetDatabase.Refresh()`，**并等 `EditorApplication.isCompiling` 回落**，
   否则随后的 `unity_get_compilation_errors` 与反射验证读到的都是**旧程序集**，
   会得出「改动没进代码」的错误结论。
   → 所有「改代码 → 验证」的流程都必须把「Refresh + 等编译结束」写成显式步骤。
   **注意区分两种「看到旧代码」**（2026-10-05 架构师用磁盘字节 + mtime 举证澄清，
   不要把两者混为一谈、更不要因此不敢凭源码下结论）：
   - **旧程序集错配**（真实存在）：改完 `.cs` 没 `Refresh`，反射/编译查询读到上一次编译的程序集。
     这次是**程序在自测中踩到**的。
   - **准确的中间快照**（不是误判）：评审者直接读**磁盘源码**时，作者的改动**当时确实还没写完**
     （架构师那次：`bytes=18040, mtime=16:24:43`，`GetType` 确实还没有 `.Trim()`、`ExportJson` 确实还是裸调；
     85 秒后文件才变成 `19507` 字节）。这种结论是可信的。
8. **改 `EditorBuildSettings.scenes` 后必须存盘，否则只改内存**（2026-10-06 由程序踩坑后确认）：
   `EditorBuildSettings.scenes = …` **只写入内存，不会自动落盘**。表现是：内存读回是对的、编辑器里
   `LoadSceneAsync` 也能成功（**编辑器不校验 Build Settings**），于是"验证通过"，
   但磁盘上的 `ProjectSettings/EditorBuildSettings.asset` 仍是旧内容 → **打包后目标场景不存在、"开始游戏"照旧抛错**。
   → 设完必须执行一次 **`File/Save Project`**，并**核对磁盘文件的 mtime 与内容**（不能只看内存返回值）。
9. **本机沙箱写不了工程目录（Windows 完整性标签机制）**：DSH 沙箱进程以 **Low 完整性级别**运行，
   靠**给目录打 `Low Mandatory Label`** 授权写入。
   - `Assets/Chaos_Story/**` 有该标签 → **可写**；`Scripts`/`Resources`/`Data`/`Excel`/`Scenes`/`Art` **都没有** →
     直写一律被拒（`[sandbox: file access denied under workspace-write mode]`）。
   - 实测**补 ACE 无效**（ACE 不是门槛）；**设标签需要管理员权限**（普通 token 无 `SeRelabelPrivilege`）。
   - **可行工作方式**：**所有文件写入都在 Unity 进程内做** —— `unity_execute_code` 里的
     `File.WriteAllText`（改 `.cs`，GBK 文件仍按 GBK 写回）、`AssetDatabase.CreateFolder/CopyAsset`（建目录/放图）、
     `EditorSceneManager`（建场景）、`EditorBuildSettings`（改 Build Settings）。Unity 不受该沙箱约束。
   - 需要"非 Unity 才能做"的原始落盘（如 xlsx 工具产物）→ **交 Lead**，Lead 用一次性放宽放置。

---

## 6. 分支与提交纪律（用户 2026-10-05 规定 —— 硬约束）

| 仓库 | 路径 | **允许修改的分支** | 远端 |
| --- | --- | --- | --- |
| 主工程 Project_Chaos | `D:\Unity Projects\Project_Chaos` | **`Branch_feature_1`** | `origin` |
| 配置表 Chaos_excel | `Assets/Excel/Chaos_excel/` | **`Feature_1`** | `origin` |
| 设计文档 Chaos_Story | `Assets/Chaos_Story/` | `main`（**用户明确批准的唯一例外**） | `origin` |

**铁律：**

1. **只在上述指定分支上改动。** 动手前先 `git branch --show-current` 确认；若不在指定分支，**停下来问 Lead**，
   不要自行切换分支、也不要新建分支。（2026-10-05 核实：三个仓库当前均已在各自指定的分支上。）
2. **只有用户能把分支合并进 `main`。** 任何成员（含 Lead）都**不得**执行 `merge`、不得 rebase 到 `main`、
   不得向 `main` 推送。文档仓 `Chaos_Story` 直接提交到 `main`，是用户对**该仓**的单独批准，**不适用于另外两个仓库**。
3. **任何分支都不许删除**（本地或远端）。`git branch -d/-D`、`git push --delete`、`git push origin :branch` 一律禁止。
4. **不要执行 `git add -A` / `git add .`** —— 一律**显式指定路径**。
   理由有二：① 主仓库里有一批与开发无关的噪声文件（`Assembly-CSharp.csproj`、`Project_Chaos.sln`、`Tools/`、`acl-*.json` 等），会被顺手带进提交；② 队友的在途改动会被一起扫进去。
   （2026-10-05 15:58 由用户在主仓库 `.gitignore` 补了 `Assets/Chaos_Story` 与 `Assets/Chaos_Story.meta`，
   嵌套仓被误加成 gitlink 的风险**已消除**；但上面两条理由仍然成立，故 `add -A` 依旧禁止。）
5. 主仓库有一个指向文档仓的远端 **`Story_origin`**（`git@github.com:Ryan1566/Chaos_Story.git`）。
   **不要使用它**，主仓库只用 `origin`（`Project_Chaos.git`），避免误把工程内容推进文档仓。
6. 提交（commit）前先问 Lead；**推送（push）由用户决定**，成员不得自行 push。

---

## 7. 配置表类型（Type）权威

> **唯一权威来源是 `ConfigManager.Convert` 的 `case` 列表**，不是本文档、不是 Excel 说明文档、也不是表头模板。
> 用户在 2026-10-05 规定：**配置表的 Type 上限以 `Convert` 支持的类型为准**；
> 按需求可以同时在 `Convert` 与配置表中新增类型，但**非必要谨慎新增**。

### 7.1 类型必须同时通过两道关

新增一个类型，必须**同时**满足：

1. 是**合法的 C# 类型名** —— 因为 `ExportClass`（`ConfigManager.cs:163-166`）把第 6 行的文本**原样**当字段类型
   （`sb.Append($"\tpublic {fieldType} {fieldName};")`）；
2. 能被 **`UnityEngine.JsonUtility` 正确往返** —— 因为运行时读取链用的是 `JsonUtility.FromJson`。

### 7.2 当前权威白名单（T7 已落地，2026-10-05）

`ConfigManager.Convert` 现有 **12** 个 `case`，**这就是配置表第 6 行允许填写的全部类型**：

```
int   uint   long   ulong   byte   sbyte   short   ushort   float   double   bool   string
```

| 变化 | 类型 | 原因 |
| --- | --- | --- |
| **移除** | `int32` `int64` `long long` | C# 里不存在这些类型名 → 曾生成 `public int32 X;` **编译不过**；现在落到 `default`，在**导出阶段**就早失败 |
| **新增** | `byte` `sbyte` `short` `ushort` | 合法 C# 且 JsonUtility 往返实测通过；适合紧凑数值列 |
| **新增** | `bool` | 合法 C# 且可用；取值规则见 §7.4 |
| **实测后否决** | `decimal` | JsonUtility **静默丢弃** —— ToJson 输出里根本没有该字段、往返恒为 0（无声错值）→ **严禁新增** |
| **实测后否决** | `char` | 能往返，但 JSON 里存成**数字**（`'A'` → `65`），语义易错 |
| 保留 | `int` `uint` `long` `ulong` `float` `double` `string` | 无回归（`TestTable` 四个产物 SHA256 与改动前逐字节一致） |

**验证方式（不是只跑 Convert）**：T7 期间建过端到端夹具 `Assets/Excel/Chaos_excel/TypeProbe_类型探针表.xlsx`，
导出后 ① 生成的 `TypeProbeConfig.cs` **编译 0 error** ② `JsonUtility.FromJson` 读回**逐列断言全部通过**，
覆盖 `sbyte −128/127`、`short −32768/32767`、`ushort 0/65535` 等边界。

> ⚠️ **该夹具当前不存在**：2026-10-05 提交时**未被保留**（配置表仓现有 `README.md` + `TestTable_测试表.xlsx`，
> `TestTable_测试表.xlsx` 第 6 行为 `uint/string/uint/uint/float`，**不含 bool**）。
> 也就是说 **bool 路径目前没有活体样本**。需要类型回归时按 §7.4 重建，重建规则见 §3.1 的例外条款
> 与 `03_路径与命名规范.md` §9。

> ⚠️ **2026-10-05 前的旧说法「不要填 `bool`」已作废。** `bool` 现已可用（见 §7.4）。

> 下表是**为什么是这 12 个**的实测依据（Lead 用临时探针、架构师用 `Reflection.Emit` **各自独立**跑过，结论一致）：

| 类型 | JsonUtility 往返 | 备注 |
| --- | --- | --- |
| `int` `uint` `long` `ulong` | ✅ | `uint` 另用生产类型 `TestTableConfig` 复核 |
| `short` `ushort` `byte` `sbyte` | ✅ | 边界值（`sbyte −128/127`、`ushort 0/65535`）实测一致 |
| `bool` | ✅ | JSON 字面量 `true` / `false` / `1` / `0` / `"true"` **都能解析**；另用生产类型 `SettingsData.vSync` 复核 |
| `float` `double` | ✅ | |
| `char` | ⚠️ 能往返 | 但 JSON 里存成**数字**（`'A'` → `65`）→ 不纳入 |
| `decimal` | ❌ **静默丢弃** | ToJson 输出里根本没有该字段，往返后恒为 0 → 严禁新增 |

### 7.3 `bool` 取值规范

**Excel 里可填**：`true` / `false`、`1` / `0`、`是` / `否`（**不区分大小写**）。
**只认这几类**；`yes` / `y` / `no` / `n` 以及其它任何文本**一律抛异常**（多一套同义词就多一种填错方式）。
导出后 Json 里是**裸值** `true` / `false`。

**归一化的理由必须写对**（2026-10-05 Lead 亲自实测，**已推翻此前"大小写不敏感"的说法**）：
- **裸值形式大小写敏感**：`{"vSync":true}` / `{"vSync":false}` ✅；
  **`{"vSync":TRUE}`、`{"vSync":True}` → `ArgumentException: JSON parse error: Invalid value.`** ❌
- **带引号的字符串形式才大小写不敏感**：`{"vSync":"true"}`、`{"vSync":"TRUE"}` 都 ✅
- `{"vSync":1}` / `{"vSync":0}` ✅（数字也接受）
- `{"vSync":是}` ❌ 抛异常；`{"vSync":"是"}` ⚠️ **不报错但静默变成 `false`**（无声错值）

导出器产出的是**裸值**，所以：**必须归一化成恰好小写 `true` / `false`**，
**绝不能和 `string` 归到一组去加引号**（否则 `"是"` 会解析失败并静默回退为 false）。
⚠️ 此前"`JsonUtility` 对 bool 大小写不敏感、所以理由不是要小写"的说法**只对带引号形式成立**，
**不要据此放行 `TRUE`** —— 上游注释 `ConfigManager.cs:349-353` 亦有此误，已列为待修。

**补充实测**：EPPlus 读取 Excel **原生布尔单元格**时 `.Text` 是 `"1"` 而不是 `"TRUE"` —— 所以 `1`/`0` 不是可选项，而是**必需**的别名。

### 7.4 新增类型的流程

策划提需求 → 架构师评估必要性（默认结论是「不加」）→ 程序在 `Branch_feature_1` 改 `Convert`
→ 在探针表 `TypeProbe_类型探针表.xlsx`（**当前不存在，需按 §3.1 例外条款先重建**）加一列、重新导出并做**端到端验证**（生成类能编译 + Json 读回断言）
→ 策划同步 `Story/03_配置表设计/01_配置表填写规范.md` 与 `00_配置表清单.md` → Lead 更新本节。

**本节是权威；但与 `Convert` 源码冲突时以源码为准，并立即通知 Lead 更正。**

---

## 8. 轻量改动快速通道（用户 2026-10-05 规定，为省 token）

**目的**：小改动不要再走「策划文档 → 架构评审 → 闸门 → 程序实现 → 自测指针」这套重流程。

### 8.1 判定清单 —— **四条全部满足**才可走快速通道

| # | 条件 | 反例（不满足 → 走正常流程） |
| --- | --- | --- |
| 1 | **逻辑改动量很小**（大致 ≤ 一屏 diff，几十行内） | 新增一个系统、重构一层、改状态机 |
| 2 | **只落在单个或少数几个脚本类里** | 跨 MVP 三层、跨多个 Manager、需要新建文件体系 |
| 3 | **与其他功能耦合很少**（不改公共基类/接口/事件常量/单例契约） | 改 `BasePanel`、改 `EventConstName`、改 `SingletonBase`、改 `GlobalPath` |
| 4 | **不需要修改配置表**（不动 `Excel/Chaos_excel/*.xlsx`，也不需要新增/改字段） | 加字段、加表、改类型、改第 5 行端标记 |

**另有一条隐含前提**：**需求本身是明确的**。需求含糊、有多种合理做法、或涉及取舍时，不属于「轻量」——先问 Lead/用户。

### 8.2 快速通道怎么走

1. **只派程序一人**（`programmer`）。**不通知架构师、不通知策划**，不建 T5/T6 那类闸门任务。
2. **不写自测指针 / 不建测试场景挂载 / 不做探针验证** —— **用户自己会测**，测不过或看不懂再来问。
   （仍然要做的底线：编译 **0 error**；若改动碰了 `Scripts/`，改完必须 `AssetDatabase.Refresh()` 并等 `isCompiling` 回落，见 §5 第 7 条。）
3. 程序**只改必要的那几个文件**，保持原编码，改完用**显式路径**报告「改了哪些文件 + 一句话说明」。
4. **不更新策划文档、不更新架构文档**（除非改动使某份文档变成错的 —— 那时只在该文档里加一行「已过时，见 XXX」并告知 Lead）。
5. 提交仍按 §6：**提交前问 Lead；推送由用户决定**。

### 8.3 判断与记录的边界

- **判定由 Lead 做**；程序若判断某任务其实不满足四条，**停下来问 Lead**，不要硬走快速通道。
- 快速通道的改动**仍要留痕**：在 `05_程序/` 的实现记录里加一行（文件 + 改动 + 用户自测结果待补），
  **目的是别让下一个 agent 重新发现**，不是为了走流程。
- 出现下列任一情况，**立即升级到正常流程**并通知相关成员：改动牵扯到公共接口、需要动配置表、
  发现既有缺陷、或用户反馈「测不过/看不懂」。
