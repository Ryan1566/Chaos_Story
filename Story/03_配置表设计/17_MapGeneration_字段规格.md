# MapGeneration 字段规格（统一连续地形参数）

> 状态：已实现（源表/导出/运行接入/最终构建恢复已核）
> Owner：策划
> 最后更新：2026-10-24
> 关联：`Story/02_系统策划/22_连续地形与结构地图生成.md`、`Story/03_配置表设计/00_配置表清单.md`、`Story/03_配置表设计/01_配置表填写规范.md`、`Assets/Excel/Chaos_excel/MapGeneration_地图生成参数.xlsx`

## 1. 表定位与产物

| 项目 | 契约 |
| --- | --- |
| 前缀／文件名 | MapGeneration／MapGeneration_地图生成参数.xlsx；顶层唯一 |
| 页签与范围 | MapGenerationSheet0；内容范围A1:AX9，50列，6表头＋3条正式数据 |
| 导出 | 原ExcelTool/ExportExcelConfigs生成MapGenerationConfig.cs与MapGenerationConfig.json；禁止手改生成物 |
| 运行时 | 程序手写NaturalMapParameters消费同名字段；不以它替代原Exporter产物 |
| 主题来源 | presetId→MapPreset.biomeId→MapBiome；不增加重复主题列，不新增战斗倍率 |
| 当前状态 | 原Exporter已导出3行/50字段，源表→最终Json150项零差异；NaturalMapParameters已读取接入，3000组批测已完成；最新增量3000组已复测；Windows64构建0错误0警告、测试数据恢复已收口，见程序10文 |

## 2. 六行表头与第七行起数据

- 第1行：A1=类名、B1=MapGeneration；第2行：A2=是否读取、B2=IsConfig。
- 第3行每列中文备注；第4行字段名，逐列非空且唯一；第5行全部client；第6行精确小写类型。
- 第7～9行分别为参数Id1/2/3；数据行禁止空数字、空bool；不加额外说明页、空列或尾部空行。
- Id、presetId为uint；enable为bool；buildingPool为string；frameBudgetMs、mainRouteMinLength、mainRouteMaxLength为float；其余43列为int。字符串禁止英文逗号、英文双引号和换行。

## 3. 正式字段清单（大小写与顺序均为契约）

默认值列按Id1／Id2／Id3顺序；只列一个值表示三行相同。

| 列 | 字段 | 类型 | 默认值 | 范围/校验要求 | 含义 |
| --- | --- | --- | --- | --- | --- |
| A | `Id` | `uint` | 1／2／3 | ≥1且唯一 | 参数行主键。三行固定1/2/3；不是MapBiome.Id。 |
| B | `presetId` | `uint` | 11／10／12 | 有效MapPreset.Id | 外键到MapPreset.Id；11地表、10地下、12建筑；同一预设启用行唯一。 |
| C | `enable` | `bool` | true | true/false | true参与新生成；false保留行但不参与参数选择。 |
| D | `widthTiles` | `int` | 320 | >0 | 有限地图本地网格宽度；与heightTiles乘积不得超过maxTileCells。 |
| E | `heightTiles` | `int` | 144 | >0 | 有限地图本地网格高度；不等于旧24格房间高度。 |
| F | `edgeMargin` | `int` | 8 | ≥0且保留有效内部 | 地图边缘保留的安全格数；生成及锚点不得越过有效边界。 |
| G | `surfaceBaseY` | `int` | 80 | 0～heightTiles-1且满足上下空间约束 | 本地网格地表基准Y；配合起伏、天空、地下边界校验。 |
| H | `skyClearance` | `int` | 32 | ≥routeHeadroom | 地表上方至少预留的净空高度；地表最高点及此值不得越过上界。 |
| I | `undergroundMinY` | `int` | 8 | ≥edgeMargin | 地下可生成区域的最低本地Y；不得挖穿底部安全边界。 |
| J | `surfaceAmplitude` | `int` | 8 | ≥0 | 地表相对基准的起伏幅度；不授权形成不可跳跃台阶。 |
| K | `surfaceStepLength` | `int` | 12 | ≥1 | 地表形态变化的横向采样/步长；最终可行走轮廓仍受maxStepHeight约束。 |
| L | `maxStepHeight` | `int` | 1 | ≥0且≤角色可跳高度 | 必经路线相邻站立面允许的最大台阶。必须不高于当前玩家可跳高度，当前1格。 |
| M | `routeHeadroom` | `int` | 4 | ≥1且容得下角色 | 受保护通路头顶净空；不能用连通的单格空气代替角色可通行空间。 |
| N | `platformMinWidth` | `int` | 6 | ≥1 | 主路/落脚点的最小稳定平台宽度；生成、结构放置及修复后均需保留。 |
| O | `maxGapWidth` | `int` | 2 | ≥0，仅非主路 | 仅非必经野外段允许的最大可挖缺口；受保护主路不得按此值随机挖断。 |
| P | `maxDropHeight` | `int` | 3 | ≥0 | 必经路线允许的最大落差；需要回程的路线还需验证逆向可达。 |
| Q | `regionWidthMin` | `int` | 40 | ≥platformMinWidth | 兼容战斗区域划分的最小宽度；不是新增完整房间块模板。 |
| R | `regionWidthMax` | `int` | 64 | ≥regionWidthMin | 兼容战斗区域划分的最大宽度；不得小于regionWidthMin。 |
| S | `branchCountMin` | `int` | 1 | ≥0 | 目标支路最少数量；受连通、空间与生成预算限制。 |
| T | `branchCountMax` | `int` | 3 | ≥branchCountMin | 目标支路最多数量；不得小于branchCountMin。 |
| U | `branchLengthMin` | `int` | 20 | ≥1 | 支路目标最短长度格数；不能以不可到达分叉满足数量。 |
| V | `branchLengthMax` | `int` | 60 | ≥branchLengthMin | 支路目标最长长度格数；不得小于branchLengthMin。 |
| W | `loopChance` | `int` | 60 | 0～100 | 合法候选支路回连的概率，单位百分比；不是强制生成60%格子的回环。 |
| X | `caveDepth` | `int` | 24 | ≥0 | 洞穴相对地表的目标深度；雕刻后不得低于undergroundMinY。 |
| Y | `caveHeight` | `int` | 8 | ≥1 | 洞穴通道目标高度；路线上净空下限仍由routeHeadroom保证。 |
| Z | `caveWidth` | `int` | 8 | ≥1 | 洞穴通道目标宽度；边界与结构占用冲突时不能越界。 |
| AA | `chamberRadius` | `int` | 8 | ≥1 | 洞室目标半径；不得将受保护路线或结构承重面平滑抹掉。 |
| AB | `caveEntranceMin` | `int` | 2 | ≥0 | 洞穴与可达区域连接的最少洞口目标数；洞口需适合实际角色通过。 |
| AC | `caveEntranceMax` | `int` | 3 | ≥caveEntranceMin | 最多洞口目标数；不得小于caveEntranceMin。 |
| AD | `caveSmoothPasses` | `int` | 1 | ≥0 | 洞穴平滑迭代次数；0允许不平滑；保护通路与结构接口。 |
| AE | `buildingCountMin` | `int` | 1／0／2 | ≥0 | 手工结构最少放置目标。地表1、地下0、建筑2；不代表新战斗倍率。 |
| AF | `buildingCountMax` | `int` | 2／1／3 | ≥buildingCountMin | 手工结构最多放置目标。地表2、地下1、建筑3。 |
| AG | `buildingMinSpacing` | `int` | 16 | ≥0 | 手工结构之间的最小间距；空间不满足时进入重试/修复报告，不能强行重叠。 |
| AH | `buildingPool` | `string` | Building | 非空资源池键 | 手工结构池名称；当前Building，复用已有资源定义，不解析逗号列表。 |
| AI | `enemiesPerRegion` | `int` | 2 | ≥0 | 每个合法战斗区域的目标敌人数；入口安全区与可达性过滤仍优先。 |
| AJ | `npcCount` | `int` | 1 | ≥0 | NPC生成接口预留点目标数；不等于已实现NPC业务或必有NPC prefab。 |
| AK | `propCount` | `int` | 3 | ≥0 | 杂物接口预留点目标数；不能堵路或占入口安全区。 |
| AL | `interactiveCount` | `int` | 1 | ≥0 | 交互接口预留点目标数；入口/出口等必要流程点独立保证。 |
| AM | `decorationCount` | `int` | 24 | ≥0 | 非阻塞装饰目标数；优先级低于路线、必要交互与战斗。 |
| AN | `contentMinSpacing` | `int` | 3 | ≥0 | 内容候选点最小间距；与物体实际占用尺寸、安全半径共同校验。 |
| AO | `entrySafetyRadius` | `int` | 8 | ≥0 | 入口周边安全半径；敌人及阻塞内容不得落入。 |
| AP | `retryMax` | `int` | 6 | ≥1 | 生成尝试的有界重试预算；失败必须报告并走已定义保底。 |
| AQ | `repairMax` | `int` | 8 | ≥0 | 单次生成可达性修复预算；不能无限挖路或无界重试。 |
| AR | `maxTileCells` | `int` | 262144 | ≥widthTiles×heightTiles | 单张地图网格安全上限；默认262144，320×144=46080格。 |
| AS | `maxContentObjects` | `int` | 256 | ≥必要对象/点数量 | 内容对象/预留点装配的总数量预算；必要点不能被装饰挤掉。 |
| AT | `workColumnsPerFrame` | `int` | 16 | ≥1 | 协程/分帧处理最多推进的列数；不改变随机分流或最终布局。 |
| AU | `frameBudgetMs` | `float` | 5 | >0 | 每帧处理时间预算毫秒；用于让出执行权，不能以机器快慢决定随机结果或改种子。 |
| AV | `mainRouteMinLength` | `float` | 200 | >0且≤mainRouteMaxLength | 基于角色通行图的入口到出口主路实际世界单位长度下限；不是房间数或地图宽度。 |
| AW | `mainRouteMaxLength` | `float` | 600 | ≥mainRouteMinLength | 同一条实际入口到出口路线的世界单位长度上限；不能用不相关支路或绕圈拼凑长度。 |
| AX | `startOnRightChance` | `int` | 50 | 0～100 | 入口位于右侧的权重百分比；0固定左、100固定右；中间值按独立layout/direction种子支线确定。 |

## 4. 三行身份与差异

| Id | presetId | 主主题（从旧表读取） | buildingCountMin | buildingCountMax |
| --- | --- | --- | --- | --- |
| 1 | 11 | Surface／biome2 | 1 | 2 |
| 2 | 10 | Underground／biome1 | 0 | 1 |
| 3 | 12 | Building／biome3 | 2 | 3 |

三预设均320×144。不同阶段以主主题偏重规划自然地形与手工结构；不是三个完全割裂的坐标/碰撞系统。除上表Id/preset与建筑数量外，其他字段值相同。主路始终保护，maxGapWidth只作用于非必经野外段。

## 5. 跨字段约束与异常

| 规则 | 处理要求 |
| --- | --- |
| 宽高与预算 | widthTiles×heightTiles使用安全整数检查；超maxTileCells拒绝该配置，不先分配巨量数组 |
| 有效边界 | 2×edgeMargin不能占满宽高；最高地表＋skyClearance不能越界；undergroundMinY不能低于底部安全边界 |
| 成对范围 | regionWidth、branchCount、branchLength、caveEntrance、buildingCount的Min≤Max；负数量/非法概率须报告字段与行 |
| 角色真实能力 | maxStepHeight不得高于当前玩家可跳高度；路线检查需使用实际角色碰撞体与运动限制，不能只做区域BFS |
| 必经主路 | 洞穴雕刻、平滑、结构/内容装配后仍需可达；不得用maxGapWidth破坏保护区 |
| 预算与确定性 | frameBudgetMs/workColumnsPerFrame只决定何时yield；机器快慢不得改变布局、重试种子或内容顺序 |
| 缺表/缺预设 | 报告明确来源与降级，不静默将错误行当作完整正常图；具体保底结果由程序记录 |
| 实际主路长度 | 在角色可通行图上累计入口至出口路线通行边的世界长度，验证200≤L≤600；不得按房间数量、区域宽度或无关分支总长替代 |
| 入口方向与镜像 | layout/direction独立种子支线按权重决策；入口右侧时整图几何、结构、锚点及朝向使用同一镜像，方向改变不额外扰动layout/content随机序列 |
| Building资源池缺失 | 标明缺池/缺候选与结构数量降级，不生成空引用；不能伪称手工结构验收完成 |

### 5.1 当前运行校验的进一步限制

以上字段表表达基本含义；当前`NaturalMapParameters.Validate(MapTraversalProfile)`在分配前还实施以下约束。配置需要同时满足跨字段公式与实际玩家/AI能力，不能仅按某一列的宽松下限填写。

| 参数组 | 当前源码限制 |
| --- | --- |
| 尺寸/格子 | width160～2048、height96～1024；width×height≤maxTileCells；maxTileCells≤2097152；还需装得下边缘/入口安全区/洞口坡道/地下主通路 |
| 轮廓/净空 | edgeMargin≥4；skyClearance≥24；undergroundMinY≥3；caveDepth≥caveHeight＋4；surfaceBaseY－surfaceAmplitude－caveDepth－chamberRadius≥undergroundMinY |
| 角色路线 | maxStepHeight≥1且≤实际可跳高度；maxDropHeight≥1且≤角色MaxDrop；routeHeadroom≥角色Height；maxGapWidth≥0且小于角色MaxJumpDistance |
| 平台/区域 | surfaceStepLength≥platformMinWidth≥3；regionWidthMin≥20且Max≥Min |
| 支路/洞穴 | branchCountMax≤16；branchLengthMin≥8且Max≤widthTiles；洞口Min≥2、Max≤6；caveHeight/caveWidth≥routeHeadroom；chamberRadius≥3；平滑0～8 |
| 结构/内容 | buildingCountMax≤12；每区域敌人1～20；contentMinSpacing≥1；entrySafetyRadius≥6且4倍半径<widthTiles；maxContentObjects1～4096 |
| 工作预算 | retryMax1～30、repairMax0～32、workColumnsPerFrame1～128、frameBudgetMs1～30且非NaN；主路长为有限值，Min≥1、Max≥Min；右侧概率0～100 |

5ms是协作让出执行权的目标，不是所有配置读取或Unity原生实例化都能中断的硬帧时长上限；具体超时与冷准备耗时见22方案验收快照，最终程序报告继续补录。

## 6. 交付与验证记录

本轮已用bundled artifact-tool制作初版47列xlsx，6段渲染覆盖原参数；随后在末尾追加AV～AX三列，并单独渲染检查新增范围。保存后重导入确认A1:AU9原47列值完整不变、新9格值正确，最终50列/3行，无公式错误。初版独立ZIP/XML复读为9行/47字段/141数据格，C7为原生bool类型b、值1（原Exporter支持1/0）；该初版证据不冒充追加后50字段已导出。本表没有公式、图表、额外源表或生成物手改。

本轮已通过原Exporter生成最终MapGenerationConfig。以ZIP/XML直接读取xlsx并对最终Json逐字段比较：**3行、50字段、150项，0差异**，字段名/顺序与client标记一致。程序已接入NaturalMapParameters、NaturalMapBuild/Plan v2与装配流程；三主题各1000组通过、确定性3000组与装饰隔离300组均0失败。运行范围及最终构建恢复见22方案§8与程序10文；完整NPC仍为占位，不能把当前接口/锚点通过写成完整NPC玩法实现。

## 7. 变更记录

| 日期 | 变更 |
| --- | --- |
| 2026-10-24 | 按本轮明确字段契约新增47列/3行源表与规格；统一320×144、主题复用旧预设映射；不新增未消费字段、不改战斗倍率；源表核查与程序导出/验收分开记录。 |

| 2026-10-24 | 在初版47列末尾追加mainRouteMinLength(float)=200、mainRouteMaxLength(float)=600、startOnRightChance(int)=50，三行一致，现A1:AX9共50列。原47列值保持；定义实际角色通行路线世界长度、整图镜像与独立layout/direction支线，程序重导出/运行验证待补。 |

| 2026-10-24 | 最终源表→原管线Json复核3×50=150项零差异；补当前Validate限制与运行接入状态，批测/正式场景证据见22。保留初版记录；最终指纹/镜像实体增量、编译构建恢复由程序收口，不承诺严格5ms。 |

| 2026-10-24 | 程序最终构建/恢复已完成：Windows64构建0错误0警告，测试存档/设置/字体已精确恢复。字段与运行验收状态收口，完整证据见22方案§8及程序10文。 |
