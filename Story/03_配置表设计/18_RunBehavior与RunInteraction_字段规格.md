# RunBehavior与RunInteraction字段规格

> 状态：已实现（原管线导出、运行接入及118项源表对照通过）
> Owner：策划
> 最后更新：2026-10-24
> 关联：`Story/02_系统策划/23_局内寻路战斗与基础交互.md`、`Story/03_配置表设计/01_配置表填写规范.md`、`Story/03_配置表设计/00_配置表清单.md`

## 1. 通用导出约定

源文件分别为`Assets/Excel/Chaos_excel/RunBehavior_局内行为参数.xlsx`与`Assets/Excel/Chaos_excel/RunInteraction_局内交互文案.xlsx`。单页签，第1行类名、第2行IsConfig装饰、第3行中文备注、第4行字段、第5行全部client、第6行类型，第7行即正式数据。数字使用真正数值，字符串不含英文逗号、英文双引号或换行，无空列/尾部空行。

原菜单`ExcelTool/ExportExcelConfigs`生成`RunBehaviorConfig`/`RunInteractionConfig`及同名Json。源表落盘不等于运行消费；生成类、Json和正式输入动作由程序维护，禁止手改生成物。

## 2. RunBehavior：角色行为参数

页签`RunBehaviorSheet0`，范围A1:X8，24字段×2数据行共48项。Id1=Enemy，Id2=Ally；role大小写固定且唯一，不靠行序识别角色。1uint、1string、2int、20float。空间单位米，时间秒，速度米/秒，加速度米/秒²；searchRadius为整数米，maxExpandedNodes为节点数量。当前地图1格=1米，若网格比例变化必须进行换算，不能把配置米当任意格数。

| 列 | 字段 | 类型 | Enemy | Ally | 含义与校验 |
| --- | --- | --- | --- | --- | --- |
| A | Id | uint | 1 | 2 | 正数且唯一的配置主键。 |
| B | role | string | Enemy | Ally | 角色行用途，必须非空且唯一；仅这两个值。 |
| C | moveSpeed | float | 3 | 5 | 移动/追击速度，>0。 |
| D | patrolSpeed | float | 1.5 | 2 | 巡逻速度，>0；Ally保留相同消费者参数接口，不强行新增巡逻玩法。 |
| E | acceleration | float | 30 | 30 | 水平加速度，>0。 |
| F | jumpForce | float | 11 | 11 | 沿用现有控制器起跳速度语义，>0；不是新伤害或战斗倍率。 |
| G | repathInterval | float | 0.4 | 0.4 | 路径重新规划间隔，>0秒。 |
| H | waypointTolerance | float | 0.18 | 0.18 | 路径点到达容差，>0米。 |
| I | stuckSeconds | float | 2 | 2 | 连续无导航进展的卡住判定，>0秒。 |
| J | recoverySeconds | float | 8 | 8 | 连续失败至允许安全恢复的阈值，≥stuckSeconds；不是每8秒固定传送。 |
| K | searchRadius | int | 8 | 8 | 起终点附近合法导航锚点搜索半径，>0米。 |
| L | maxExpandedNodes | int | 4096 | 4096 | 单次寻路展开节点上限，>0整数。 |
| M | detectRange | float | 8 | 8 | 目标发现距离，>0米。 |
| N | loseRange | float | 12 | 12 | 目标丢失距离，≥detectRange。 |
| O | memorySeconds | float | 2 | 2 | 丢失目标后记忆持续时间，>0秒。 |
| P | attackRange | float | 1.25 | 2.5 | 攻击范围，>0米；还要满足高度/遮挡条件。 |
| Q | attackWindup | float | 0.25 | 0.12 | 命中前摇，>0秒。 |
| R | attackRecovery | float | 0.2 | 0.15 | 命中后摇，>0秒。 |
| S | attackCooldown | float | 0.8 | 0.6 | 相邻攻击起手最短间隔，≥attackWindup+attackRecovery。 |
| T | attackHeight | float | 1.8 | 1.8 | 攻击目标高度容差，>0米。 |
| U | patrolWait | float | 0.6 | 0.6 | 巡逻节点等待时间，>0秒。 |
| V | returnTolerance | float | 0.6 | 0.6 | 返回/到位容差，>0米。 |
| W | leashPadding | float | 0 | 0 | 真实活动区域边界允许外扩，≥0米；0为不外扩，不能误拒绝。 |
| X | interactionRange | float | 2.5 | 2.5 | 受控角色公共交互距离，>0米；正式交互读取Ally行，Enemy行不启用敌人交互。 |

所有float须为有限值，拒绝NaN/Infinity；除leashPadding外数字均>0。必须校验前后摇之和不超过冷却、loseRange≥detectRange、recoverySeconds≥stuckSeconds。当前两行前后摇之和0.45/0.27，分别小于0.8/0.6，关系均合法。角色属性、敌人HP/伤害与地形倍率仍走原角色/MapBiome通路，本表没有HP、伤害、奖励列。

## 3. RunInteraction：独立五语言文案

页签`RunInteractionSheet0`，范围A1:G16，7字段×10数据行共70项，其中五语言共50条。表不复用MapLoading，不改变旧本地化行。

| 列 | 字段 | 类型 | 规则 |
| --- | --- | --- | --- |
| A | Id | uint | 固定1～10，正数且唯一。 |
| B | key | string | 以下10个稳定键，非空且唯一；运行时按key读取。 |
| C | zhCN | string | 简体中文，必填。 |
| D | enUS | string | 英语，必填。 |
| E | zhTW | string | 繁体中文，必填。 |
| F | jaJP | string | 日语，必填。 |
| G | koKR | string | 韩语，必填。 |

| Id/key | zhCN | enUS | zhTW | jaJP | koKR |
| --- | --- | --- | --- | --- | --- |
| 1 Prompt | {0}：{1} | {0}: {1} | {0}：{1} | {0}：{1} | {0}: {1} |
| 2 Unavailable | 暂不可交互 | Unavailable | 暫不可互動 | 現在は操作できません | 지금은 상호작용할 수 없습니다 |
| 3 Completed | 已查看 | Inspected | 已查看 | 確認済み | 확인 완료 |
| 4 Npc | 路人 | Passerby | 路人 | 通行人 | 행인 |
| 5 Interactive | 交互物 | Interactive object | 互動物 | 操作対象 | 상호작용 대상 |
| 6 Inspect | 查看 | Inspect | 查看 | 確認 | 확인 |
| 7 OutOfRange | 距离过远 | Out of range | 距離過遠 | 距離が遠すぎます | 거리가 너무 멉니다 |
| 8 Blocked | 前方有遮挡 | Obstructed | 前方有遮擋 | 障害物があります | 장애물에 가로막혔습니다 |
| 9 Inactive | 目标已失效 | Target inactive | 目標已失效 | 対象は無効です | 대상이 비활성화되었습니다 |
| 10 ActionName | 交互 | Interact | 互動 | インタラクト | 상호작용 |

Prompt仅含{0}和{1}两个格式占位：{0}=当前Interact绑定显示名称，{1}=本地化操作/目标描述；译文保持占位完整，不把数字替成全角，不填固定键位。ActionName用于设置中的动作名称，不代表键位绑定。Completed仅表示基础查看成功，不承诺对话/奖励业务。无当前候选时隐藏提示；Unavailable是一般条件失败文案，其余原因按实际条件返回，不从文本推导业务。

## 4. 验收与当前状态

源表由Spreadsheets技能的artifact工具生成，已校验字段顺序、client标记、默认值、关系与字符串限制，分块渲染核对。两表总118项数据、50条翻译；源xlsx复读、导出Json对照、正式消费与功能验收分别记录，不能互相替代。运行验收清单见23方案§6；实际导出、编译、流程和恢复由主代理补录。

## 5. 变更记录

| 日期 | 变更 |
| --- | --- |
| 2026-10-24 | 新增RunBehavior固定24字段/2角色与RunInteraction固定7字段/10key五语言源表；保持原生命/伤害/掉落/通关规则，等待原管线导出及消费证据。 |
