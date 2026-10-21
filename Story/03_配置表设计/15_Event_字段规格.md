# `Event` 字段规格说明（剧情事件表）

> 状态：评审中
> Owner：策划
> 最后更新：2026-10-21
> 关联：`Story/02_系统策划/16_剧情事件.md`（规则与判定口径）、`Story/03_配置表设计/16_EventOption_字段规格.md`（选项与奖励）、`Story/03_配置表设计/01_配置表填写规范.md`、`Assets/Excel/Chaos_excel/Event_剧情事件表.xlsx`

> **触发条件只有 3 种**（用户裁决，原型期不得扩展）：`TriggerType = 1` runs / `2` maxClearedDepth / `3` familyTreeNodes。

> **两条硬约束（写给策划自己看）**
> 1. **顶层目录 `Assets/Excel/Chaos_excel/` 里任何一张表都必须 ≥ 1 行数据**（只有表头 = 空表）。只有表头 → 无导出项；**页签完全空白 → 抛异常并中断整批导出**，其后所有表都不导出。
> 2. **顶层同英文前缀只能有一张表**（`Event_A.xlsx` 与 `Event_B.xlsx` 会导出同一个 `EventConfig` 并**静默互相覆盖**）。
> 3. 表头固定 **7 行**，**第 7 行就是第一条数据**；表尾不得留空行；第 5 行每列填 `client`；第 6 行类型只能取 **12 类型白名单**且小写。

> ⚠️ **写手必读**：`TitleText` / `BodyText` 是**中文原文**，**严禁出现英文逗号 `,`、英文双引号 `"`、换行**（导出器用 `Split(',')` 重组单元格 → 英文逗号会**破坏整表 Json**）。中文标点（，。、；：！？「」）**随便用**。

---

## 1. 表定位

| 项目 | 内容 |
| --- | --- |
| 表名（英文前缀） | `Event` |
| 源文件名 | `Event_剧情事件表.xlsx`（顶层） |
| 用途（一句话） | 定义剧情事件：**触发条件 / 是否一次性 / 优先顺序 / 标题与正文** |
| 导出模式 | **Config** → `EventConfig.json` + `ConfigData/EventConfig.cs` |
| 数据量级 | **策划建议** MVP 5~15 行 |
| 关联系统 | M4 剧情事件（局外）、`16_剧情事件.md` |
| 状态 | **评审中**（规格已出；⚠️ xlsx **由程序用 EPPlus 建**，本次不建表） |

---

## 2. 表头填写区（7 行）

| 行 | A | B | C | D | E | F | G | H |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第 1 行（装饰） | `类名` | `Event` | | | | | | |
| 第 2 行（装饰） | `是否读取` | `IsConfig` | | | | | | |
| 第 3 行 中文备注 | 序号 | 触发类型 | 触发值 | 是否可重复 | 优先级 | 标题 | 正文 | 是否启用 |
| 第 4 行 英文属性名 | `Index` | `TriggerType` | `TriggerValue` | `Repeatable` | `Priority` | `TitleText` | `BodyText` | `Enable` |
| 第 5 行 端标记 | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` |
| 第 6 行 类型 | `uint` | `uint` | `uint` | `bool` | `uint` | `string` | `string` | `bool` |
| 第 7 行起 | — | — | — | — | — | — | — | **正式数据** |

---

## 3. 字段清单（逐列）

| 列 | 第 3 行 中文备注 | 第 4 行 属性名 | 第 5 行 端标记 | 第 6 行 类型 | 含义与取值 | 取值范围 | 示例 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 序号 | `Index` | `client` | `uint` | 事件唯一 ID；**存档 `firedEvents` 存的就是它**；`EventOption.EventId` 引用它 | ≥1 | `1001` |
| B | 触发类型 | `TriggerType` | `client` | `uint` | 见 §3.1；**只有 1/2/3** | `1~3` | `1` |
| C | 触发值 | `TriggerValue` | `client` | `uint` | `type=1`→`runs` 阈值；`type=2`→`maxClearedDepth` 阈值；`type=3`→**家族树节点 ID** | 见 §3.1 | `3` |
| D | 是否可重复 | `Repeatable` | `client` | `bool` | `false`=**一次性**（处置后写 `firedEvents`，永不再弹）；`true`=可重复（每次回局外最多一次） | `true`/`false` | `false` |
| E | 优先级 | `Priority` | `client` | `uint` | 同时满足多个时的弹出顺序：**数值大先弹**；相同 → `Index` 升序 | `0~99` | `10` |
| F | 标题 | `TitleText` | `client` | `string` | 事件标题（**中文原文**，非本地化 key）；**禁英文逗号/英文双引号/换行** | ≤30 字 | `村口的陌生人` |
| G | 正文 | `BodyText` | `client` | `string` | 事件正文（中文原文）；**禁英文逗号/英文双引号/换行**；建议 ≤300 字 | 见左 | `一名风尘仆仆的旅人…` |
| H | 是否启用 | `Enable` | `client` | `bool` | `false` = 不参与检查（保留行便于下线）。**不能留空** | `true`/`false` | `true` |

### 3.1 `TriggerType` 编码与 `TriggerValue` 语义（`uint` + 对照表，**不能用 enum**）

| `TriggerType` | 条件 | 判定（**逐字口径**） | `TriggerValue` 填什么 | 示例 |
| --- | --- | --- | --- | --- |
| **1** | 进入局内次数 | `runs >= TriggerValue` | **次数阈值** | `3` = 进过 3 次局内起可触发 |
| **2** | 通关过的最高深度 | `maxClearedDepth >= TriggerValue` | **深度阈值**（1~5） | `2` = 通关过深度 2 起可触发 |
| **3** | 已点亮局外技能点 | `familyTreeNodes.Contains(TriggerValue)` | **家族树节点 ID**（`101~503`） | `303` = 点亮"门客培养·Ⅲ"后触发 |
| `4+` | **不做**（用户裁决） | — | — | 新增必须重走裁决 |

> `type=3` 的节点 ID 见 `11_FamilyTree_字段规格.md` §3.2（`1xx` 影响力 / `2xx` 家族产业 / `3xx` 门客培养 / `4xx` 局外战斗 / `5xx` 局内战斗）。
> 条件**只判"是否达到"**，不判"是否刚达到"。

---

## 4. 示例数据（**示范内容，不是定稿文本**）

| 行 | `Index` | `TriggerType` | `TriggerValue` | `Repeatable` | `Priority` | `TitleText` | `BodyText` | `Enable` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第 7 行 | 1001 | 1 | 1 | `false` | 10 | 第一次出征 | 你带着门客从传送阵归来。村中老者在村口等了许久，说你们看上去有些不一样了。 | `true` |
| 第 8 行 | 1002 | 2 | 2 | `false` | 20 | 深层的回响 | 你们从更深处带回了不属于这个时代的东西。老族长摩挲着它，久久没有作声。 | `true` |
| 第 9 行 | 1003 | 3 | 303 | `false` | 30 | 门客的誓言 | 门客培养有成。一名门客在营前立誓，愿随你走更远的路。 | `true` |
| 第 10 行 | 1004 | 1 | 5 | `true` | 0 | 又一趟 | 第五次踏入边缘空间后，你开始习惯那种失重感。 | `true` |

> 上表 4 行都不含英文逗号/英文引号/换行 —— **可以直接照格式填**。

---

## 5. 数据约束与校验规则

| # | 约束 | 违反后果 | 谁来把关 |
| --- | --- | --- | --- |
| C1 | `Index` 唯一、非空、从 1001 起（便于与其它表区分） | 存档 `firedEvents` 冲突 | 程序/策划 |
| C2 | `TriggerType` ∈ `1~3` | 其它值 → 视为不满足 + Warn（事件永不触发） | 策划 |
| C3 | `type=1/2` 的 `TriggerValue ≥ 1`；`type=3` 必须是**存在的节点 ID** | `0` 或悬空 → 永不触发 + Warn | 策划 |
| C4 | **每个事件至少 1 个选项**（`EventOption` 里 `EventId` 有对应行） | 只显示「关闭」按钮（体验差，Warn） | 策划 |
| C5 | 选项数 `1~3` | 超 3 → UI 溢出（建议工具拦截） | 策划 |
| C6 | `TitleText` / `BodyText` **禁英文逗号 / 英文双引号 / 换行** | **整表 Json 损坏**（最致命） | 写手 + 程序 |
| C7 | `TitleText` 非空；`BodyText` 建议非空 | 空标题 → UI 难看 | 策划 |
| C8 | 所有 `Repeatable` / `Enable` 逐行填满 | 空值 → 导出期抛异常并**中断整批导出** | 程序/策划 |
| C9 | 行数 ≥1（顶层硬约束） | 空表 → 无导出项/中断整批 | 程序/策划 |

---

## 6. 与其他表的关系

| 关系 | 说明 |
| --- | --- |
| → `EventOption` | `EventOption.EventId = Event.Index`（1~3 行/事件）；**导出器不校验外键**，悬空选项会被忽略 + Warn |
| ← `SavePayload` | `firedEvents`（`List<uint>`）记录已处置的一次性事件 ID |
| ← 进度数据 | `runs` / `maxClearedDepth` / `familyTreeNodes`（均已存在） |

---

## 7. 待确认问题

| # | 问题 | 影响 | 需谁确认 | 状态 |
| --- | --- | --- | --- | --- |
| Q1 | 列名用 **`TitleText`/`BodyText`**（本文，中文原文）还是 `TitleKey`/`TextKey`（需补本地化表）？ | 写手与本地化 | **用户** + 程序 | **待确认** |
| Q2 | `Index` 是否统一从 **1001** 起（避开其它表 ID 段）？ | 存档可读性 | 策划 | 待确认（本文按 1001 起） |
| Q3 | 是否需要 **`Era` 列**（按时代筛事件）？ | 内容节奏 | **用户** | **待确认**（本轮不做） |
| Q4 | `Priority` 范围 `0~99` 是否够？ | 排序粒度 | 策划 | 待确认 |
| Q5 | 是否要 **`OncePerSave` 之外的"每局一次"** 语义（可重复事件每次回局外都弹是否过密）？ | 可重复事件节奏 | **用户** | **待确认** |

---

## 8. 变更记录

| 日期 | 变更 | 变更人 |
| --- | --- | --- |
| 2026-10-21 | 首版（W5.1）：定 **8 列**（`Index`/`TriggerType`/`TriggerValue`/`Repeatable`/`Priority`/`TitleText`/`BodyText`/`Enable`）；`TriggerType` **只有 1/2/3**（用户裁决）+ `TriggerValue` 逐类型语义表；**文本列用中文原文（`*Text`）而非本地化 key**（现有面板不挂 `LocalizedText`，Q1）；写手红线：禁英文逗号/引号/换行；C1~C9 约束（含"每事件至少 1 选项、选项 ≤3"）；Q1~Q5 待确认。xlsx 由程序用 EPPlus 建，本次不建表。 | 策划 |
