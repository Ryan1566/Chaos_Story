# `Building` 字段规格说明（建筑表 · 每级一行）

> 状态：评审中
> Owner：策划
> 最后更新：2026-10-17
> 关联：`Story/02_系统策划/07_村庄系统.md`（建造/收获规则）、`Story/03_配置表设计/08_Effect_字段规格.md`（建造时长/收获产量效果）、`Story/02_系统策划/06_资源与库存.md`（4 类资源）、`Story/03_配置表设计/01_配置表填写规范.md`、`Assets/Excel/Chaos_excel/Building_建筑表.xlsx`

> **关键约定：每个建筑的每个等级占一行**（"每级一行"）。
> 因此 **`Index` 是行 ID（唯一），`BuildingId` 才是"建筑身份"** —— 同一建筑的 L1/L2 两行 **`BuildingId` 相同、`Level` 不同**。
> `Effect`（类型 6 建造时长 / 类型 7 收获产量）的 `Target` **指向 `BuildingId`**（如 `2001` 农田），不是指向行 `Index`。

> **两条硬约束（写给策划自己看）**
> 1. **顶层目录 `Assets/Excel/Chaos_excel/` 里任何一张表都必须 ≥ 1 行数据**（只有表头 = 空表）。只有表头 → 无导出项；**页签完全空白 → 抛异常并中断整批导出**，其后所有表都不导出。
> 2. **顶层同英文前缀只能有一张表**（`Building_A.xlsx` 与 `Building_B.xlsx` 会导出同一个 `BuildingConfig` 并**静默互相覆盖**）。
> 3. 表头固定 **7 行**，**第 7 行就是第一条数据**；表尾不得留空行；第 5 行每列填 `client`；第 6 行类型只能取 **12 类型白名单**且小写。

---

## 1. 表定位

| 项目 | 内容 |
| --- | --- |
| 表名（英文前缀） | `Building` |
| 源文件名 | `Building_建筑表.xlsx`（顶层） |
| 用途（一句话） | 定义村庄 **4 个建筑 × 2 个等级**的建造时长、建造消耗、收获周期与产出 |
| 导出模式 | **Config** → `BuildingConfig.json` + `ConfigData/BuildingConfig.cs` |
| 数据量级 | **8 行**（4 建筑 × 2 级） |
| 关联系统 | M1 村庄（建造/升级/收获）、`Effect`（类型 6/7 加成） |
| 状态 | **评审中**（规格已出；⚠️ xlsx **由程序用 EPPlus 建**，本次不建表） |

---

## 2. 表头填写区（7 行）

| 行 | A | B | C | D | E | F | G | H | I | J | K |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第 1 行（装饰） | `类名` | `Building` | | | | | | | | | |
| 第 2 行（装饰） | `是否读取` | `IsConfig` | | | | | | | | | |
| 第 3 行 中文备注 | 序号 | 建筑ID | 名称key | 等级 | 建造秒数 | 产出资源类型 | 每次产出数量 | 收获周期秒 | 建造消耗资源类型 | 建造消耗数量 | 是否启用 |
| 第 4 行 英文属性名 | `Index` | `BuildingId` | `NameKey` | `Level` | `BuildSeconds` | `OutputResourceType` | `OutputAmount` | `HarvestSeconds` | `CostResourceType` | `CostAmount` | `Enable` |
| 第 5 行 端标记 | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` | `client` |
| 第 6 行 类型 | `uint` | `uint` | `string` | `uint` | `uint` | `uint` | `uint` | `uint` | `uint` | `uint` | `bool` |
| 第 7 行起 | — | — | — | — | — | — | — | — | — | — | **正式数据** |

> 共 **11 列（A~K）**；**中间不得出现空列**（第 4 行任一列为空 → 抛异常并**中断整批导出**）。

---

## 3. 字段清单（逐列）

| 列 | 第 3 行 中文备注 | 第 4 行 属性名 | 第 5 行 端标记 | 第 6 行 类型 | 含义与取值 | 取值范围 | 示例 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| A | 序号 | `Index` | `client` | `uint` | **行 ID，表内唯一**（1 起连续）；**不是建筑身份** | ≥1 | `1` |
| B | 建筑ID | `BuildingId` | `client` | `uint` | **建筑身份**（同一建筑多级共用）；`Effect.Target` 指向它 | `2001~2004` | `2001` |
| C | 名称key | `NameKey` | `client` | `string` | 建筑名**本地化 key**。**禁英文逗号 / 英文双引号 / 换行** | 字母数字下划线 | `building_2001_name` |
| D | 等级 | `Level` | `client` | `uint` | 该行对应的等级（1 起；**一个建筑的行构成 1..N 连续**） | **1~2**（MVP） | `1` |
| E | 建造秒数 | `BuildSeconds` | `client` | `uint` | 建造/升级**基础**耗时（秒）；实际时长 = 本列 − `EffectType=6` 的加成，**clamp ≥1** | 1~3600 | `30` |
| F | 产出资源类型 | `OutputResourceType` | `client` | `uint` | 收获产出哪种资源（1~4，见 §3.2） | **1~4** | `1` |
| G | 每次产出数量 | `OutputAmount` | `client` | `uint` | 每个收获周期产出多少；实际 = 本列 + `EffectType=7` 加成 | ≥1 | `2` |
| H | 收获周期秒 | `HarvestSeconds` | `client` | `uint` | 每 N 秒累积 1 份 | 1~3600 | `30` |
| I | 建造消耗资源类型 | `CostResourceType` | `client` | `uint` | 建造/升级扣哪种资源（1~4） | **1~4** | `1` |
| J | 建造消耗数量 | `CostAmount` | `client` | `uint` | 扣多少（取消时返还 `floor(值×50%)`） | ≥0 | `20` |
| K | 是否启用 | `Enable` | `client` | `bool` | `false` = 该等级不可建造（用于临时关闭某级）。**不能留空** | `true`/`false` | `true` |

### 3.1 与 `Effect` 的接口（**必须按此对接**）

| 效果类型 | `Effect.Target` 填什么 | 作用到哪 |
| --- | --- | --- |
| 类型 **6**（建造时长） | **`BuildingId`**（如 `2001`） | 该建筑**所有等级**的建造时长 |
| 类型 **7**（收获产量） | **`BuildingId`**（如 `2001`） | 该建筑**所有等级**的每次产出 |

> 例：现有 `Effect` 4 = `类型6 / Target 2001 / Value -5` → 农田（`BuildingId=2001`）建造时长 **30 → 25 秒**（L1）、60 → 55 秒（L2）。

### 3.2 资源类型对照表（`uint` + 对照表，**不能用 enum**）

| 值 | 资源 | 存档字段 |
| --- | --- | --- |
| `1` | 物资 | `resGoods` |
| `2` | 材料 | `resMaterial` |
| `3` | 钱财 | `resMoney` |
| `4` | 图纸 | `resBlueprint` |

> 与 `11_FamilyTree_字段规格.md` §3.3 保持一致（**同一套映射，避免两处定义不同**）。

---

## 4. 示例数据（**与 `07_村庄系统.md` §3.1 一致**；数值是策划建议）

| 行 | `Index` | `BuildingId` | `NameKey` | `Level` | `BuildSeconds` | `OutputResourceType` | `OutputAmount` | `HarvestSeconds` | `CostResourceType` | `CostAmount` | `Enable` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 第 7 行 | 1 | 2001 | `building_2001_name` | 1 | 30 | 1（物资） | 2 | 30 | 1（物资） | 20 | `true` |
| 第 8 行 | 2 | 2001 | `building_2001_name` | 2 | 60 | 1（物资） | 4 | 30 | 1（物资） | 60 | `true` |
| 第 9 行 | 3 | 2002 | `building_2002_name` | 1 | 45 | 2（材料） | 1 | 45 | 1（物资） | 40 | `true` |
| 第 10 行 | 4 | 2002 | `building_2002_name` | 2 | 90 | 2（材料） | 2 | 45 | 2（材料） | 20 | `true` |
| 第 11 行 | 5 | 2003 | `building_2003_name` | 1 | 45 | 3（钱财） | 5 | 60 | 1（物资） | 50 | `true` |
| 第 12 行 | 6 | 2003 | `building_2003_name` | 2 | 90 | 3（钱财） | 10 | 60 | 3（钱财） | 80 | `true` |
| 第 13 行 | 7 | 2004 | `building_2004_name` | 1 | 120 | 4（图纸） | 1 | 120 | 2（材料） | 30 | `true` |
| 第 14 行 | 8 | 2004 | `building_2004_name` | 2 | 180 | 4（图纸） | 1 | 90 | 4（图纸） | 5 | `true` |

> 4 建筑：`2001` 农田（物资）/ `2002` 工坊（材料）/ `2003` 集市（钱财）/ `2004` 书斋（图纸）。

---

## 5. 数据约束与校验规则

| # | 约束 | 违反后果 | 谁来把关 |
| --- | --- | --- | --- |
| C1 | `Index` 表内唯一、非空、从 1 连续 | 排查困难 | 程序/策划 |
| C2 | `(BuildingId, Level)` **组合唯一** | 同建筑同级两行 → 读到哪一行不确定 | 程序（可校验） |
| C3 | 同一 `BuildingId` 的 `Level` **从 1 连续**（1,2,3…无跳号） | 升级链断裂（L1 之后直接找 L3） | 程序/策划 |
| C4 | `BuildingId` 全局唯一（一个建筑一个 ID，不同于行 `Index`） | `Effect.Target` 指向错建筑 | 策划 |
| C5 | `BuildSeconds ≥ 1`、`HarvestSeconds ≥ 1` | 0 秒 → 一瞬间完成/疯狂产出 | 策划 |
| C6 | `OutputAmount ≥ 1`（0 产出无意义，要停用请改 `Enable`） | 建了不产出 | 策划 |
| C7 | `OutputResourceType` / `CostResourceType` ∈ `1~4` | 找不到资源 | 策划 |
| C8 | `NameKey` **禁英文逗号 / 英文双引号 / 换行** | 英文逗号会**破坏整表 Json** | 策划 |
| C9 | 所有 `Enable` 逐行填满 | 空值 → 导出期抛异常并**中断整批导出** | 程序/策划 |
| C10 | 行数 ≥1（顶层硬约束） | 空表 → 无导出项/中断整批 | 程序/策划 |

---

## 6. 与存档 / 其他表的关系

| 数据 | 在表里 | 在存档里 | 说明 |
| --- | --- | --- | --- |
| `BuildingId` | ✅ | ✅ `BuildingSave.id` / `buildingType` | MVP 每建筑 1 实例，两者同值 |
| `Level`（当前） | ✅（各级一行） | ✅ `BuildingSave.level`（0 = 未建） | 存档只存"当前级"，升级目标级 = 当前+1 |
| `BuildSeconds` / `CostAmount` | ✅ | ❌ | 建造时读表（含 `Effect` 调整） |
| `HarvestSeconds` / `OutputAmount` | ✅ | ❌ | 收获周期读取；`pendingCharges`/`harvestAccum` 落存档（见 `07_村庄系统.md` §6.2） |
| `NameKey` | ✅ | ❌ | 显示时回查表 |

---

## 7. 待确认问题

| # | 问题 | 影响 | 需谁确认 | 状态 |
| --- | --- | --- | --- | --- |
| Q1 | **建筑清单**（农田/工坊/集市/书斋）与**建筑 ID 段 2001~2004** 是否认可？ | 与 `Effect.Target=2001` 的既有约定绑定 | **用户** + 策划 | **待确认** |
| Q2 | 是否需要 **≥3 级**（当前 2 级）？ | 内容量与养成深度 | **用户** | **待确认** |
| Q3 | 消耗是否要**多种资源**同时扣（当前每级只支持 1 种）？ | 经济设计；若要多资源 → 需拆 `BuildingCost` 关联表 | **用户** + 策划 | **待确认**（本文按单资源） |
| Q4 | 是否要"**建造前置**"（如仓库需先有农田 L1）？ | 建造顺序 | **用户** | **待确认**（MVP 不做） |
| Q5 | `OutputResourceType` 与 `CostResourceType` 的 1~4 映射是否与程序 `ResourceKind` 一致 | 点错资源 | 程序 + 策划 | **待确认**（同 `11_` Q1） |

---

## 8. 变更记录

| 日期 | 变更 | 变更人 |
| --- | --- | --- |
| 2026-10-17 | 首版（W1b）：定 **11 列**（`Index`/`BuildingId`/`NameKey`/`Level`/`BuildSeconds`/`OutputResourceType`/`OutputAmount`/`HarvestSeconds`/`CostResourceType`/`CostAmount`/`Enable`）；**"每级一行"** 的落地方式：`Index` 是行 ID、`BuildingId` 是建筑身份（`Effect.Target` 指向它，与现有 `Effect 4 → 2001` 对齐）；8 行示例（4 建筑 × 2 级）；资源类型 1~4 对照表；C1~C10 约束；Q1~Q5 待确认。xlsx 由程序用 EPPlus 建，本次不建表。 | 策划 |
