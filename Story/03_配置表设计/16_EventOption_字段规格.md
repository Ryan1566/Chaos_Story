# `EventOption` 字段规格说明（事件选项表）

> 状态：已实现
> Owner：策划
> 最后更新：2026-10-22
> 关联：`Story/02_系统策划/16_剧情事件.md`（事件规则与奖励口径）、`Story/03_配置表设计/15_Event_字段规格.md`（事件主表）、`Story/03_配置表设计/10_Item_字段规格.md`（物品）、`Story/03_配置表设计/09_Retainer_字段规格.md`（门客）、`Story/03_配置表设计/08_Effect_字段规格.md`

> **一张表把"选项文本 + 奖励"都装下**：每个事件 1~3 行；**`EventId + OptionIndex` 组合唯一**。
> 奖励支持 4 类（资源 / 物品 / 门客 / 信心）+ 1 类预留（`EffectId`，本期不实现）。

> **两条硬约束**
> 1. **顶层目录 `Assets/Excel/Chaos_excel/` 里任何一张表都必须 ≥ 1 行数据**（只有表头 = 空表）。只有表头 → 无导出项；**页签完全空白 → 抛异常并中断整批导出**，其后所有表都不导出。
> 2. **顶层同英文前缀只能有一张表**（`EventOption_A.xlsx` 与 `_B.xlsx` 会导出同一个 `EventOptionConfig` 并**静默互相覆盖**）。
> 3. 表头固定 **7 行**，**第 7 行就是第一条数据**；表尾不得留空行；第 5 行每列填 `client`；第 6 行类型只能取 **12 类型白名单**且小写。

> ⚠️ `OptionText` 是**中文原文**，**禁英文逗号 / 英文双引号 / 换行**（英文逗号会破坏整表 Json）。

---

## 1. 表定位

| 项目 | 内容 |
| --- | --- |
| 表名（英文前缀） | `EventOption` |
| 源文件名 | `EventOption_事件选项表.xlsx`（顶层） |
| 用途（一句话） | 定义每个事件的 1~3 个选项：**按钮文本 + 选择后发放的奖励** |
| 导出模式 | **Config** → `EventOptionConfig.json` + `ConfigData/EventOptionConfig.cs` |
| 数据量级 | **策划建议** MVP 5~15 事件 × 1~3 选项 = **5~45 行** |
| 关联系统 | M4 剧情事件（局外） |
| 状态 | **已实现**（10行，12列；Json/Config类均存在；本轮仅静态核对，运行证据见20/21报告） |

---

## 2. 表头填写区（7 行）

| 行 | A | B | C | D | E | F | G | H | I | J | K | L |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第 1 行（装饰） | `类名` | `EventOption` | | | | | | | | | | |
| 第 2 行（装饰） | `是否读取` | `IsConfig` | | | | | | | | | | |
| 第 3 行 中文备注 | 序号 | 事件ID | 选项序号 | 选项文本 | 给资源类型 | 给资源数量 | 给物品ID | 给物品数量 | 给门客ID | 信心变化 | 效果ID | 是否启用 |
| 第 4 行 英文属性名 | `Index` | `EventId` | `OptionIndex` | `OptionText` | `GiveResourceType` | `GiveResourceAmount` | `GiveItemId` | `GiveItemCount` | `GiveRetainerId` | `ConfidenceDelta` | `EffectId` | `Enable` |
| 第 5 行 端标记 | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` |
| 第 6 行 类型 | `uint` | `uint` | `uint` | `string` | `uint` | `uint` | `uint` | `uint` | `uint` | `int` | `uint` | `bool` |
| 第 7 行起 | — | — | — | — | — | — | — | — | — | — | — | **正式数据** |

> 共 **12 列（A~L）**；中间不得有空列（第 4 行任一列为空 → 抛异常并**中断整批导出**）。
> ⚠️ `ConfidenceDelta` 是 **`int`**（可以为负），其余数量列都是 `uint`（负值会被拒绝/视为 0）。

---

## 3. 字段清单（逐列）

| 列 | 第 3 行 中文备注 | 第 4 行 属性名 | 第 5 行 端标记 | 第 6 行 类型 | 含义与取值 | 取值范围 | 示例 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 序号 | `Index` | `client` | `uint` | 选项行唯一 ID（1 起连续） | ≥1 | `1` |
| B | 事件ID | `EventId` | `client` | `uint` | **外键 → `Event.Index`** | 存在的事件 ID | `1001` |
| C | 选项序号 | `OptionIndex` | `client` | `uint` | 该事件内的选项顺序（决定按钮**从上到下**的顺序） | `1~3` | `1` |
| D | 选项文本 | `OptionText` | `client` | `string` | 按钮文字（**中文原文**）；**禁英文逗号/英文双引号/换行** | ≤20 字 | `收下这份礼` |
| E | 给资源类型 | `GiveResourceType` | `client` | `uint` | `0`=不给资源；`1`物资 / `2`材料 / `3`钱财 / `4`图纸 | `0~4` | `1` |
| F | 给资源数量 | `GiveResourceAmount` | `client` | `uint` | 给多少（`GiveResourceType=0` 时忽略） | ≥0 | `50` |
| G | 给物品ID | `GiveItemId` | `client` | `uint` | `0`=不给物品；否则 **`Item.Index`** | `0` 或存在的物品 ID | `1` |
| H | 给物品数量 | `GiveItemCount` | `client` | `uint` | 给几个（`GiveItemId=0` 时忽略）；受 `Item.StackLimit` 与格位约束 | ≥0 | `2` |
| I | 给门客ID | `GiveRetainerId` | `client` | `uint` | `0`=不给门客；否则 **`Retainer.Index`（模板 ID = `protoId`）**，按模板实例化 1 名门客 | `0` 或存在的模板 ID | `3` |
| J | 信心变化 | `ConfidenceDelta` | `client` | **`int`** | 对**全部门客**信心加减（clamp `0~100`）；`0`=不变 | `-100~+100` | `10` |
| K | 效果ID | `EffectId` | `client` | `uint` | `0`=无；**本期预留不实现**（选项是瞬发、`Effect` 是常驻聚合，需新增 `grantedEffects` 存档字段 → `16_剧情事件.md` Q2） | `0`（本期恒 0） | `0` |
| L | 是否启用 | `Enable` | `client` | `bool` | `false` = 该选项不显示。**不能留空** | `true`/`false` | `true` |

### 3.1 奖励的叠加与执行顺序

| # | 规则 |
| --- | --- |
| P1 | 一个选项**可以同时**给资源 + 物品 + 门客 + 信心（各列非 0 即生效） |
| P2 | 执行顺序固定：**资源 → 物品 → 门客 → 信心**（效果互相不依赖，顺序只影响提示顺序） |
| P3 | **单项失败不影响其它项**：例如物品因格位满失败 → 提示「物品未能放入（格位已满）」，其它奖励照给 |
| P4 | 全部为空（资源/物品/门客/信心都是 0）的选项 = **纯对话选项**（合法，只推进/关闭） |

### 3.2 资源类型对照表（与其它表一致）

| 值 | 资源 | 存档字段 |
| --- | --- | --- |
| `1` | 物资 | `resGoods` |
| `2` | 材料 | `resMaterial` |
| `3` | 钱财 | `resMoney` |
| `4` | 图纸 | `resBlueprint` |

> 与 `11_FamilyTree_字段规格.md` §3.3、`12_Building_字段规格.md` §3.2 **同一套映射**，不要另定。

---

## 4. 示例数据（**对应 `15_Event_字段规格.md` §4 的事件 1001**）

| 行 | `Index` | `EventId` | `OptionIndex` | `OptionText` | `GiveResourceType` | `GiveResourceAmount` | `GiveItemId` | `GiveItemCount` | `GiveRetainerId` | `ConfidenceDelta` | `EffectId` | `Enable` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第 7 行 | 1 | 1001 | 1 | 收下这份礼 | 1 | 50 | 0 | 0 | 0 | 0 | 0 | `true` |
| 第 8 行 | 2 | 1001 | 2 | 请他喝一杯 | 0 | 0 | 1 | 2 | 0 | 0 | 0 | `true` |
| 第 9 行 | 3 | 1001 | 3 | 邀他入伙 | 0 | 0 | 0 | 0 | 3 | 5 | 0 | `true` |
| 第 10 行 | 4 | 1002 | 1 | 仔细查看 | 0 | 0 | 4 | 1 | 0 | 0 | 0 | `true` |
| 第 11 行 | 5 | 1002 | 2 | 交还族长 | 2 | 30 | 0 | 0 | 0 | 10 | 0 | `true` |

> 事件 1001「第一次出征」有 3 个选项（拿物资 / 拿药品 / 拉人入伙），完整演示了 4 类奖励中的 3 类；1002 演示"物品 + 资源 + 信心"。

---

## 5. 数据约束与校验规则

| # | 约束 | 违反后果 | 谁来把关 |
| --- | --- | --- | --- |
| C1 | `Index` 唯一、非空、从 1 连续 | 排查困难 | 程序/策划 |
| C2 | `(EventId, OptionIndex)` **组合唯一** | 同一事件两个"第 1 选项" → 顺序不确定 | 程序（可校验） |
| C3 | 每个 `EventId` 的 `OptionIndex` 从 1 连续且 **≤3** | UI 溢出 / 按钮顺序错乱 | 策划 + 工具 |
| C4 | `EventId` 必须存在于 `Event.Index`（**导出器不校验外键**） | 悬空选项被忽略 + Warn | 策划 |
| C5 | `OptionText` **禁英文逗号 / 英文双引号 / 换行** | **整表 Json 损坏** | 写手 |
| C6 | `GiveResourceType` ∈ `0~4`；`GiveItemId`/`GiveRetainerId` 为 `0` 或存在的 ID | 悬空 → 跳过该项 + Warn | 策划 |
| C7 | `GiveResourceAmount`/`GiveItemCount` ≥0（`uint`，不填负数） | 负值会被当成超大数（危险） | 策划 |
| C8 | `ConfidenceDelta` ∈ `-100~+100`（`int`） | 超界 → clamp（不报错但语义混乱） | 策划 |
| C9 | `EffectId` **本期恒 0**（预留） | 填非 0 → 本期不生效（Warn） | 策划 |
| C10 | 所有 `Enable` 逐行填满 | 空值 → 导出期抛异常并**中断整批导出** | 程序/策划 |
| C11 | 行数 ≥1（顶层硬约束） | 空表 → 无导出项/中断整批 | 程序/策划 |

---

## 6. 与其他表 / 系统的关系

| 关系 | 说明 |
| --- | --- |
| ← `Event` | `EventId = Event.Index`；一个事件 1~3 行 |
| → `Item` | `GiveItemId` = `Item.Index`；数量受 `Item.StackLimit`（默认 99）与**格位 8**（可提升，硬上限 24）约束 |
| → `Retainer` | `GiveRetainerId` = 模板 `Index`（= 存档 `protoId`）；实例化 1 名门客 |
| → `Effect` | `EffectId` 外键**预留给后续**（需 `grantedEffects`，Q2） |
| 存档 | 选项本身**不落盘**；只有"事件已处置"记在 `firedEvents` |

---

## 7. 待确认问题

| # | 问题 | 影响 | 需谁确认 | 状态 |
| --- | --- | --- | --- | --- |
| **Q1** | **"给门客"是直接白给 1 名**（本文）还是**生成一个可招募的候选**？ | 价值与体验 | **用户** | **待确认**（本文按"直接加入"） |
| **Q2** | **"加信心"作用范围**：全部门客（本文）／当前队伍／指定门客？ | 奖励语义 | **用户** | **待确认** |
| **Q3** | 选项是否需要"**代价**"（扣资源/降信心）而不是只有奖励？ | 选择深度 | **用户** | **待确认**（本文只支持正向 + 负信心） |
| **Q4** | `EffectId` 是否本期就做（需新增 `grantedEffects` 存档字段）？ | 事件能否永久改数值 | **用户** + 程序 | **待确认**（本文预留不实现） |
| **Q5** | `OptionText` 是否也改成 `*Key`（跟随 `Event` 的 Q1）？ | 本地化一致性 | **用户** + 程序 | **待确认** |
| **Q6** | 是否需要"**选项仅显示条件**"（如某些门客才可选）？ | 内容复杂度 | **用户** | **待确认**（MVP 不做） |

---

## 8. 变更记录

| 日期 | 变更 | 变更人 |
| --- | --- | --- |
| 2026-10-21 | 首版（W5.1）：定 **12 列**（`Index`/`EventId`/`OptionIndex`/`OptionText`/`GiveResourceType`/`GiveResourceAmount`/`GiveItemId`/`GiveItemCount`/`GiveRetainerId`/`ConfidenceDelta`/`EffectId`/`Enable`）；奖励 **4 类落地**（资源/物品/门客/信心）+ `EffectId` **预留**；`ConfidenceDelta` 用 **`int`**（可负），其余 `uint`；执行顺序与"单项失败不回滚"口径；5 行示例；C1~C11 约束；Q1~Q6 待确认。xlsx 由程序用 EPPlus 建，本次不建表。 | 策划 |

| 2026-10-22 | 同步源表/导出产物已存在的状态；实际配置与奖励快照见21报告。未改字段、源表或数值，未新增运行验收结论。 | 策划 |
