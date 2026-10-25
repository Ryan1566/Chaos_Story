# VillageScene与VillageText字段规格

> 状态：已实现（原导出/运行接入及224项配置对照通过，证据见程序12）
> Owner：策划
> 最后更新：2026-10-25
> 关联：`Story/02_系统策划/24_实体村庄与工作生产.md`、`Story/03_配置表设计/12_Building_字段规格.md`、`Story/03_配置表设计/00_配置表清单.md`、`Story/03_配置表设计/01_配置表填写规范.md`

## 1. 源表与导出

`Assets/Excel/Chaos_excel/VillageScene_村庄场景配置.xlsx`与`Assets/Excel/Chaos_excel/VillageText_村庄界面文案.xlsx`各一个页签。第1行类名、第2行IsConfig装饰、第3行中文备注、第4行属性名、第5行全部client、第6行类型、第7行正式数据。无空列、空页签、尾部空数据行，字符串无英文逗号/双引号/换行。

原`ExcelTool/ExportExcelConfigs`生成VillageSceneConfig和VillageTextConfig及对应Json，禁止手改生成物。本规格不改变BuildingConfig字段和八行成本/建造秒/产量/HarvestSeconds。

## 2. VillageScene（14字段/1行）

页签VillageSceneSheet0，A1:N7。类型为1uint、3int、9float、1string，数据共14项。米制场景当前1格=1米；位置允许负数和0，其余尺寸/速度/时长要求有限正数。

| 列 | 字段 | 类型 | 默认 | 约束与用途 |
| --- | --- | --- | --- | --- |
| A | Id | uint | 1 | 唯一正数主键，当前单行1。 |
| B | widthTiles | int | 96 | >0，固定村庄地面宽格数，横向以0居中。 |
| C | groundDepth | int | 6 | >0，地面向下厚度，不能解释为场景总高度。 |
| D | playerSpawnX | float | -38 | 有限且在村庄有效地面范围内，玩家出生横坐标；Y按实际角色支撑放置。 |
| E | groundY | float | 0 | 有限，地面顶面世界Y；0为合法默认值。 |
| F | workerCount | int | 4 | >0且本轮固定4，对应BuildingId2001～2004一岗一人；不按门客名额计算。 |
| G | workerMoveSpeed | float | 2 | >0，工人移动米/秒。 |
| H | workSeconds | float | 8 | >0，每次连续实际工作时长，单位秒；不替代Building.HarvestSeconds周期。 |
| I | restSeconds | float | 3 | >0，每次休息秒数，休息不生产。 |
| J | wanderMinSeconds | float | 2 | >0，闲逛到点后的Idle等待秒数下界，移动耗时另计。 |
| K | wanderMaxSeconds | float | 5 | ≥wanderMinSeconds，到点等待秒数上界，不保证整个闲逛分支只耗2～5秒。 |
| L | plotSpacing | float | 18 | >0，固定地块中心间距米；不修改建造成本。 |
| M | firstPlotX | float | -27 | 有限，首地块横坐标；后续每块加plotSpacing，四块均应处于有效地面。 |
| N | groundTilePool | string | Surface | 非空已有地面资源池键；不是可任意拼接的文件路径或数组。 |

四地块按BuildingId2001、2002、2003、2004映射到-27、-9、9、27米。宽度/位置还须给建筑、玩家和NPC的真实碰撞体留足边界，不只验证中心点。所有浮点拒绝NaN/Infinity，尺寸/持续时间不能为0；不允许为通过字段校验偷偷增减岗位数或经济数值。

## 3. VillageText（7字段/30行）

页签VillageTextSheet0，A1:G36。Id(uint)、key(string)、zhCN/enUS/zhTW/jaJP/koKR(string)，字段顺序固定；第7～36行为Id1～30。共210数据项，其中五语言150条。原Id1～22及模板不变，Id23～30追加名称。key非空且唯一，运行按key查找，Id只是稳定行主键。

| Id | key | 简中默认 | 格式参数 |
| --- | --- | --- | --- |
| 1 | MapAction | 世界地图 | 设置动作名称，不是实际绑定键。 |
| 2 | MapHint | {0}：世界地图 | {0}=当前地图动作绑定显示。 |
| 3 | BuildHint | {0}：管理地块 | {0}=当前交互动作绑定显示。 |
| 4 | VillageTitle | 村庄 | 无。 |
| 5 | Building | 建造中 | 施工状态，含升级中；不作建筑名称。 |
| 6 | Working | 工作中 | NPC实际到位Working。 |
| 7 | Idle | 待命 | NPC待命。 |
| 8 | Wander | 闲逛 | NPC闲逛。 |
| 9 | Rest | 休息 | NPC休息。 |
| 10 | Build | 建造 | 操作名称。 |
| 11 | Upgrade | 升级 | 操作名称。 |
| 12 | Harvest | 收获 | 操作名称，仅已有产物可领取。 |
| 13 | Cancel | 取消建造 | 取消施工操作，升级也走同规则。 |
| 14 | Level | 等级 {0} | {0}=当前等级。 |
| 15 | Queue | 建造剩余 {0} 秒 | {0}=真实剩余建造秒数。 |
| 16 | Production | 待收获 {0} | {0}=待收获数量。 |
| 17 | PressToBuild | {0}：建造 | {0}=当前交互动作绑定显示。 |
| 18 | NoWorker | 无工人工作 | 当前无实际Working工人，不推断永久缺员。 |
| 19 | Producing | 生产进度 {0}/{1} 秒 | {0}=累计工作秒余数；{1}=Building.HarvestSeconds。 |
| 20 | NoYield | 暂无产出 | 无待收获，禁止即时发一周期资源。 |
| 21 | SaveFailed | 保存失败：{0} | {0}=保存失败原因；不把失败展示为成功。 |
| 22 | MapDisabled | 当前无法打开世界地图 | 当前状态不可打开，非永久取消功能。 |
| 23 | Building2001 | 农田 | 2001产物资的建筑名称。 |
| 24 | Worker2001 | 农夫 | 2001岗位名。 |
| 25 | Building2002 | 工坊 | 2002产材料的建筑名称。 |
| 26 | Worker2002 | 工匠 | 2002岗位名。 |
| 27 | Building2003 | 书斋 | 2003产图纸的建筑名称。 |
| 28 | Worker2003 | 学者 | 2003岗位名。 |
| 29 | Building2004 | 集市 | 2004产钱财的建筑名称。 |
| 30 | Worker2004 | 商贩 | 2004岗位名。 |

8个名称key均有五语言译文，不含格式占位。名称根据原Building八行实际产出映射，修正旧07中文字面顺序相反的问题，不通过改OutputResourceType或其它数值迎合旧名。建筑和工人显示消费这些名称key，不直接显示building_2003_lv1等内部NameKey。

五语言同一key必须包含相同的格式参数集合，其它key不含格式参数。Prompt相关实际键位不写死M/E；重绑定后取当前显示，数值由状态格式化后传入。完整英/繁/日/韩译文以源表为准，已提供而不是空回退占位。

## 4. 与旧村庄规则的关系

VillageScene只管实体场景布局与NPC行为节奏，Building仍管成本/时长/产出/周期。本轮修复VillageService旧Harvest未检查周期而直接AddResource的问题：仅实际Working累计满原HarvestSeconds才形成待收获；生产小数秒与待收获量由程序加入存档，不放入只读Config、不持久化NPC实体坐标。旧07方案中即时反复领取行为由24方案明确替代。

建造Tick原每秒驱动不能直接按帧CeilToInt扣uint秒；生产累计/建造小数处理由程序保证不同dt切分等价。没工人、休息或离线不补产。收获不改变农田原家族树增产，不新增工人效率/价格/倍率。

## 5. 源表与原导出/运行核对

源表阶段：两表由artifact工具生成并渲染，zipXML独立复读350个表头+数据单元格与契约一致，0差异；正式数据224项、五语言150项。追加时旧VillageText A1:G28共196个单元格的值与样式XML完全保持，新增56项逐项一致；格式占位集合匹配，无公式错误或导出禁用字符串。

运行阶段：2026-10-25实际读取`Assets/QA/Village/config.txt`，确认原导出VillageScene14项、VillageText210项按字段类型与源表一致，输入动作/绑定GUID唯一。浮点对照脚本采用rel_tol=1e-6、abs_tol=1e-7，其它值直接比较；不称浮点二进制逐位相等。contracts.txt确认30唯一key及150条五语言非空；formal.txt确认正式加载MainScene、固定地块/生产/读档重建和地图重绑定。源码按当前LocalizationManager语言选择五列，不据此声称全部语言UI逐屏实测。

实际Validate还拒绝宽小于48/大于512或奇数、非整数groundY、groundDepth不在2～32、工人数不在1～16、地块间距小于12与超边界等技术非法参数；默认岗位仍4个，不新增多工人经济。ValidateSceneLayout在Ready前进一步核cell bounds、三个世界角点和四锚点，25契约含3项合法参数变更拒绝旧场景；48增量含真实运行时地形错位回槽及重试。

最终配置工具由主代理接管，config_incremental.txt记录--check-only下224typed值/GUID通过，7个独立内存反例被拒，禁止Path.write_text仍完成且旧config.txt哈希不变，未变更真实源表/生成Json。HUD英/日两个按钮的语言刷新/单行布局已动态验证，未扩大为全部五语言逐屏UI。新版构建与原数据恢复及导入后复核见`Story/05_程序/12_MainScene实体村庄实现与验收.md`。源表内容未在文档收口阶段修改。

## 6. 变更记录

| 日期 | 变更 |
| --- | --- |
| 2026-10-24 | 用户授权实体村庄，固定14场景字段与22key五语言；两源表落盘，Building成本/产量/周期保持。 |
| 2026-10-24 | 保留原22key，追加Building2001/Worker2001～Building2004/Worker2004八key五语言；VillageText改30行/210项，两源表总224项。 |
| 2026-10-25 | 回填原导出224typed值/唯一GUID、30key五语言非空、25契约/48增量与新版构建；只读工具由主代理最终接管验收，区分源表复读、浮点容差及英日HUD与全UI覆盖范围；恢复及导入后复核完成，见程序12。 |
