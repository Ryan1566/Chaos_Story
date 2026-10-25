# MainScene实体村庄实现与验收

> 状态：已实现并验收（95基础流程＋48增量、25契约、224配置、新构建通过；原数据恢复及导入后哈希复核完成）
> Owner：程序（主代理实现；策划按实际证据归档）
> 最后更新：2026-10-25
> 关联：`Story/02_系统策划/24_实体村庄与工作生产.md`、`Story/03_配置表设计/19_VillageScene与VillageText_字段规格.md`、`Story/05_程序/08_EnterScene正式玩法入口迁移.md`、`Story/05_程序/11_局内寻路战斗与交互实现及验收.md`

## 1. 正式行为与旧入口替代

开始→选择/读取存档→LoadingScene→MainScene实体村庄。读档后没有WorldMap根面板，村庄HUD提供移动、近地块提示、设置和主菜单；默认M打开/关闭辅助世界地图，默认E在附近地块打开原村庄建造管理面板。实际提示与消费使用当前重绑定动作。地图不是村庄主菜单上的可点击入口，设置覆盖时地图键不能穿透。

派遣由手动打开的地图进入，沿用编队→LoadingScene→WorldScene三图→结算；结算后返回MainScene，地图关闭，存档中的建筑重建为场景实体。08记录的“读档后局外地图根面板/结算回EnterScene”属于历史，当前入口以本文为准。旧07/08的免费重复收取行为被24方案的工作周期与待收获规则替代，未改Building原八行数值。

## 2. 实现位置与生命周期

以下程序路径以主仓库Assets为基准。

| 位置 | 职责 |
| --- | --- |
| `Scenes/MainScene.unity`、`Scripts/Editor/VillageSceneTool/VillageSceneBuilder.cs` | 持久化固定96×6地面、复合碰撞与4个VillagePlot锚点；场景进入构建列表。 |
| `Scripts/Runtime/GameBase/Village/VillageSceneFlow.cs` | 已选会话、构建场景和配置预检；经Loading进入MainScene；就绪后清旧面板；失败回存档页，返回主菜单先保存再退出。 |
| `VillageSceneBootstrap.cs`、`VillagePlot.cs`、`VillageVisuals.cs` | 读取配置、校验固定地形/地块，创建玩家和生产NPC、重建建筑外观、近地块提示及相机跟随；销毁时撤销岗位与就绪状态。 |
| `Scripts/Runtime/MVP/View/VillageHud.cs`、`Presenter/Gameplay/VillageWorldPresenter.cs`、`Model/Gameplay/VillageWorldModel.cs` | HUD渲染和意图、定期状态刷新、地块/设置/返回主菜单操作。Presenter销毁时Dispose；HUD不是局外根面板。 |
| `Scripts/Runtime/NPC_AI/Production/VillageBehaviorTree.cs`、`VillageProductionNpc.cs` | 独立生产NPC行为树，不加入门客、伤害或战斗统计。 |
| `Scripts/Runtime/GameBase/Village/VillageWorkforce.cs` | 每建筑一个岗位所有者；仅真实Working所有者可推进生产，非所有者不能释放他人岗位。 |
| `Scripts/Runtime/GameBase/Meta/VillageService.cs` | 真实生产累计、待收获领取、正式命令事务保存和失败回滚、建造小数秒累计。 |
| `Scripts/Runtime/GameBase/SaveManager/SaveSlotData.cs` | BuildingSave新增productionSeconds、pendingOutput、pendingResourceType，旧档缺字段默认为0。 |

持久相机栈复用现有世界/UI相机，停用重复预览世界相机/监听器；正式断言验证一个世界相机、一个AudioListener并保留UI相机。派遣离村后生产NPC清理、生产停止，返回重建四工人，不残留战斗门客。无会话直接进入MainScene会返回EnterScene。

增量复核：原VillageWorldModel的UIManager/PanelType面板导航已移到VillageWorldPresenter；Model保留Read、CanManage就近业务复核及返回主菜单服务门面。HUD设置/主菜单标签由状态快照按当前语言刷新；incremental.txt验证英/日刷新、单行布局及设置/M/E/主菜单导航，相关结构遗留已修复。

## 3. 四岗位与生产时钟

| BuildingId | 建筑/工人 | 地块X | 原产出 |
| --- | --- | --- | --- |
| 2001 | 农田/农夫 | -27 | 物资，Lv1每周期2、Lv2每周期4 |
| 2002 | 工坊/工匠 | -9 | 材料，同上 |
| 2003 | 书斋/学者 | 9 | 图纸，同上 |
| 2004 | 集市/商贩 | 27 | 钱财，同上 |

四建筑周期均为原HarvestSeconds=30秒，农田继续消费原家族树增产/建造时长效果，不新增工人效率、离线收益或经济倍率。

行为树由ReactiveSelector、Guard、Sequence和ActionNode组成。工作可用时获取岗位→走向入口→工作；到入口后隐藏外观并停用刚体模拟，表示简化室内工作。工作8秒后释放岗位并出门休息3秒，然后移动到闲逛点，在到点后待命2～5秒再返回。2～5秒是闲逛分支末端等待，另有实际移动耗时，不保证整个闲逛分支仅耗2～5秒。走向岗位期间状态文字可能已是Working，但只有入内后VillageWorkforce的Working标记生效才产出。

TickProduction校验场景已就绪、有会话、未派遣及岗位所有权，按实际工作dt累加productionSeconds。完整周期转换为pendingOutput并保留余数，pendingResourceType锁定已产批次资源；待收获以uint安全饱和处理。休息、闲逛、未到位、无岗位、离场和离线不补生产。

正式Harvest只领取pendingOutput；零产出拒绝，领取多少扣多少，资源uint容量不足时保留未领取部分，重复点击不会新造一周期产物。建造计时先累计小数再扣完整秒，避免旧逐帧Ceil导致每帧扣一秒；生产进度与建造计时分开。NPC实体位置/行为阶段不落盘，生产小数秒、待收获及资源类型落盘，旧档不凭空补产。

VillageService.Execute对建造、升级、取消、收获统一捕获Payload快照→执行→SaveProgress；保存失败用FromJsonOverwrite恢复同一Payload并刷新界面。源码证明四命令共用事务路径；本轮正式故障注入直接验证的是升级回滚及释放后重试，未把它扩大成四命令各自锁盘全测。

## 4. 已读取的验收证据

证据目录`Assets/QA/Village/`。正式流程使用隔离临时存档、输入动作和带射线检查的合成按钮操作；不是人工鼠标全流程或完整手操战斗证据。

| 文件 | 实际内容与范围 |
| --- | --- |
| `formal.txt` | 95条PASS，含控件可用/射线等辅助断言；`VILLAGE_FLOW_DONE`与`VILLAGE_RELOAD_DONE`齐全，无以FAIL/ABORT开头的失败项。 |
| `contracts.txt` | 25条PASS：原22项默认参数、30唯一key与五语言非空、非法配置拒绝、旧档零生产字段、岗位所有权、行为树抢占、场景保存/碰撞/锚点/无缺脚本；另3项合法groundY/firstPlotX/plotSpacing变更拒绝旧场景。 |
| `config.txt` | VillageScene14字段×1行=14、VillageText7字段×30行=210，共224项按字段类型与源表对照；动作与绑定GUID唯一。浮点比较脚本使用容差，不能解释为所有浮点字节逐位相同。 |
| `incremental_expected_errors.txt` | 运行时地形错位故障注入产生2条预期错误日志（村庄初始化及回槽错误提示）；归档后清空本批次控制台，不作为正常流程未解决异常。 |
| `incremental.txt` | 48条PASS、0项FAIL/ABORT，VILLAGE_INCREMENTAL_DONE；英/日HUD两个按钮刷新/单行、Presenter导航、M/E/设置、运行时地形平移拒Ready→回槽保留会话/明确错误→同档重试四工人及退出。95基础流程保留，不把增量合并成独立玩法场景数量。 |
| `config_incremental.txt` | --check-only读取224typed值/GUID；7个独立内存反例被拒，包括源表额外行、Json字段缺/增、错数字类型、bool冒充uint、Json多行、未知源类型；禁止所有Path.write_text仍完成，旧config.txt哈希保持，未改真实xlsx/Json。 |
| `build.txt` | 新版Windows64 Player构建Succeeded，errors=0、warnings=0，375036742字节，00:00:20.1532646；四正式场景见下节。 |
| `restored.txt` | 新构建后原Slot3、settings及3字体恢复并在最后AssetDatabase导入后再次核对原始SHA256；Slot1哈希不变，最后测试使用隔离存储。 |

正式流程直接覆盖：读槽3进入实体村庄、实际移动与落地、M开关及重绑定、设置覆盖屏蔽；地块2001打开原界面建造/自动倒计时完成；NPC入门工作、休息不推进、实际工作周期形成原产量；领取仅转移已产出、重复收取不多发；升级保存失败资源/状态回滚、重试及取消50%返还；其余三生产职业建成并进入各自建筑；四人派遣、三图、离村不产、结算回村；停止Play再读档后四建筑/实体重建，离线不增加进度、不丢已有部分周期；保存返回主菜单并清工人/HUD。

主流程使用临时资源和完成建筑等测试夹具，三图属于正式入口回归，不作为村庄经济平衡或全手操战斗验收。95不是95种独立玩法组合。截图`VillageConstruction.png`、`VillageWorking.png`、`VillageReturn.png`、`VillageLoaded.png`用于对应阶段外观留档；它们不会证明完整建筑美术或全部语言界面。

## 5. 构建与恢复

构建输出`C:/Users/26751/AppData/Local/Temp/ChaosVillageBuild/Chaos.exe`。构建场景为EnterScene、WorldScene、LoadingScene、MainScene；本报告确认BuildPipeline成功，不据此宣称已发布或独立可执行文件完整流程实测。

最新构建后主代理已恢复测试引起的字体缓存变化，并在AssetDatabase最终导入后重新核对原Slot3.json、Config/settings.json及三份字体的SHA256，全部与测试前备份一致，证据见`Assets/QA/Village/restored.txt`。Slot1 SHA256保持`A87BC9BC14D4E6D488F4676DE743BE028224F41541FF45B080F128F2F1ABE639`。字体映射占用通过卸载该字体资产及子资产释放，未改权限或删除文件。

formal记录的隔离目录为`C:\Users\26751\AppData\Local\Temp\ChaosVillageIsolated-b4919084d43946e391c171db89a3b396`，夹具只在该新目录创建槽3。隔离重载中的建筑和资源是测试数据，不是原玩家档的新状态。本文归档过程没有再运行Unity或写玩家存档。

## 6. 保留边界与未覆盖组合

HUD硬编码中文、Model直接导航混入及固定场景一致性缺口已修复并完成对应增量验证。就绪前ValidateSceneLayout校验cell bounds min/max、三个世界角点和四锚点；25契约与48增量分别证明配置变更拒旧场景及实际运行时地形错位回槽/重试。新版构建使用上节新数字，已替换旧基础构建证据。

配置检查工具最后由主代理接管完成，config_incremental.txt为最终验收；此前DSH尝试因max_tokens失败，不能登记为DSH实现成功。反例只在内存夹具中构造，没有删除或修改真实源表/Json；.model-desk本地记录仅忽略，不涉及删除事项。

本轮实体村庄入口、基础建造/生产和往返链路已按上述证据完成。建筑外观使用占位几何、工人复用现有外观；入门隐藏仅模拟室内，没有室内场景。无离线生产、无NPC招募/经济/位置持久化，均为批准范围边界。

尚未逐屏验收五语言全部UI、四正式命令分别锁盘、极端资源上限/所有dt切分及独立Player完整交互流程；不能把英/日两个HUD按钮验证、源码存在或一次构建扩大为这些组合都测过。当前批准实现范围已按列明证据完成，原数据恢复及导入后复核完成，无待删除事项。Project_Chaos与Chaos_excel仍由用户提交推送，本轮没有提交或推送。

## 7. 变更记录

| 日期 | 变更 |
| --- | --- |
| 2026-10-25 | 按实际QA、源码与截图归档实体村庄；95基础＋48增量、25契约/224配置、英日HUD/场景错位恢复及新版四场景构建；配置工具最终主代理接管，原数据恢复与导入后复核完成。保留旧08历史与明确覆盖边界。 |
