# MVP分层改造与验收

> 状态：已实现（兼容项与验证边界见§7）
> Owner：程序
> 最后更新：2026-10-22
> 关联：`Story/README.md`、`Story/04_架构/04_MVP契约与落地路线.md`（冻结历史草稿，只读参考）、`Story/05_程序/06_W5收尾实现与验收.md`、`Story/02_系统策划/21_W5收尾与数值评审.md`

本轮按用户要求，将当前交互/数据展示接入 `Assets/Scripts/Runtime/MVP` 的 Model／Presenter／View 分工。View构建控件、渲染快照和转发输入；Presenter组织意图与显示、调用业务；Model承载交易/编辑/结算状态。既有 `GameBase/Meta` 业务服务继续复用，不以批量移动目录作为完成标准。本文按磁盘源码及 `Assets/QA/MVP/` 证据记录，不改冻结的04架构文档，也不把其中ModelHub、IModel迁移等历史设想写成本轮产出。

## 1. 目录地图与职责

以下路径以主工程 `Assets/` 为基准；`ConfigData`、`ModelData` 继续保留生成物身份。

```text
Scripts/Runtime/MVP/
├─ Presenter/
│  ├─ PresenterBase.cs            PresenterBase<TView>／IPanelView<TState,TIntent>
│  ├─ Gameplay/
│  │  ├─ GameplayPresenter.cs     PanelIntent／PanelAction／IGameplayView<TState>
│  │  ├─ InventoryPresenter.cs
│  │  ├─ MarketPresenter.cs
│  │  ├─ RetainerPresenter.cs
│  │  ├─ DispatchPresenter.cs
│  │  ├─ EventPresenter.cs
│  │  ├─ FamilyTreePresenter.cs
│  │  ├─ VillagePresenter.cs
│  │  ├─ EraProgressPresenter.cs
│  │  ├─ CampPresenter.cs
│  │  ├─ SettlementPresenter.cs
│  │  └─ RunHudPresenter.cs        TeamHudPresenter／DepthHudPresenter
│  ├─ Startup/MainMenuPresenter.cs
│  ├─ SaveSlots/SaveSlotsPresenter.cs
│  ├─ WorldMap/WorldMapPresenter.cs
│  └─ Settings/SettingsPresenter.cs
├─ Model/
│  ├─ Gameplay/
│  │  ├─ MarketModel.cs            交易资格快照与买卖业务入口
│  │  ├─ RetainerRecruitmentModel.cs 招募防重、保存与失败回滚
│  │  ├─ RunSettlementModel.cs     单局结算选择、已应用、待保存事务
│  │  ├─ RunControlModel.cs        战斗对象暂停/输入适配
│  │  └─ RunHudModel.cs            队伍/进度HUD数据快照
│  ├─ Startup/MainMenuModel.cs
│  ├─ SaveSlots/SaveSlotsModel.cs
│  ├─ WorldMap/WorldMapModel.cs
│  ├─ Settings/SettingsEditingModel.cs
│  ├─ ConfigData/                 Excel生成配置DTO，禁止手改
│  └─ ModelData/                  导出生成DTO，禁止手改
└─ View/                         控件/布局/动画/输入转发/渲染
```

Presenter均为普通C#类，不继承MonoBehaviour，不持有Button/Transform/TMP控件或直接执行文件IO；主菜单、存档、世界地图及设置的页面导航在Presenter中调用UIManager。Model没有UIManager、PanelType或View引用。`RunControlModel`是Unity战斗对象适配器，`RetainerRecruitmentModel`使用Time/JsonUtility，因此**不能声称所有Model均已脱离Unity运行时依赖**。

## 2. 接入范围

已接入Presenter的16个交互/数据View组件：

| View组件 | Presenter | 状态/业务权威 |
| --- | --- | --- |
| InventoryPanel | InventoryPresenter | InventoryService快照 |
| MarketPanel | MarketPresenter | MarketModel→MarketService/InventoryService |
| RetainerPanel | RetainerPresenter | RetainerRecruitmentModel→RetainerService与SaveProgress |
| DispatchPanel | DispatchPresenter | DispatchService，保留队伍1～4人的资格/选择规则 |
| EventPanel | EventPresenter | EventService当前队列、Resolved状态与已保存结果 |
| FamilyTreePanel | FamilyTreePresenter | FamilyTreeService |
| VillagePanel | VillagePresenter | VillageService |
| EraProgressPanel | EraProgressPresenter | MetaProgressService／EraConfigService |
| RunCampPanel | CampPresenter | CampService／RunControlModel |
| RunSettlementPanel | SettlementPresenter | RunSettlementModel.Current跨覆盖事务 |
| RunTeamHud | TeamHudPresenter | RunHudModel队伍快照 |
| RunDepthHud | DepthHudPresenter | RunHudModel进度快照 |
| MainMenuPanel | MainMenuPresenter | MainMenuModel；导航由Presenter处理 |
| SLPanel | SaveSlotsPresenter | SaveSlotsModel；保存成功后Presenter替换世界地图根 |
| WorldMapPanel | WorldMapPresenter | WorldMapModel；选择/预览与开面板导航分工 |
| SettingPanel | SettingsPresenter | SettingsEditingModel编辑事务与兼容门面 |

`RunTerrainToast`本来是纯展示，本轮保留。设置子页/设置行仍是SettingPanel的子View，并未每一行都生成一个Presenter；16不是所有MonoBehaviour总数，也不是16个独立系统Model。各Presenter的State/Row是展示快照，常用IReadOnlyList/数组拷贝；不是全部深度不可变类型，View不能借其中集合反向修改业务状态。

## 3. 新接口用法与生命周期

实际公共接口在 `Scripts/Runtime/MVP/Presenter/PresenterBase.cs`：

```csharp
public interface IPanelView<TState, TIntent>
{
    event Action<TIntent> Intent;
    void Render(TState state);
}
```

Gameplay接口扩展了 `ShowMessage(string,bool)` 与 `ClosePanel()`，`PanelIntent`携带 `PanelAction` 和可选uint Id。派遣/交易/结算按钮先发送意图，由Presenter调用服务或Model，再将State交回Render。布局、字体、颜色、按钮创建和动画仍在View。

以现有MarketPanel为例，生命周期调用关系如下（省略控件构建）：

```csharp
// View的显示入口：在控件准备好之后创建/挂载。
_presenter = new MarketPresenter(new MarketModel());
_presenter.Attach(this);

// 买入按钮只发强类型意图；不直接扣钱或保存。
Intent?.Invoke(new PanelIntent(PanelAction.Buy, itemId));

// OnExit同步退订；缓存面板再次显示创建新Presenter。
_presenter?.Dispose();
_presenter = null;
```

`PresenterBase.Attach`已挂载时不重复订阅，Disposed后重新Attach抛ObjectDisposedException；`Dispose`可重复调用，OnDetach后在finally清View引用。OnAttach抛异常也会Dispose清理。`GameplayPresenter`在Attach订阅Intent并首次Render，Detach成对退订；Dispose后的旧View意图无效。

必须维持的时序：

- `BuildUi/Awake`只负责一次性的控件建立与按钮绑定；Presenter在面板显示路径挂载，构造函数不能调用View。
- `OnExit`发生在退场动画之前，必须立即退订，不把Dispose延迟到动画完成。`OnDestroy`兜底，调用需幂等。
- UIManager的Push覆盖旧面板也会触发OnExit，Pop恢复会重新OnEnter；**隐藏Presenter不等于结束业务事务**。控件的点击回调发送当前Intent，不捕获已Dispose的旧Presenter。
- `HideAllPanelsForGameplay`直接SetActive，不把所有OnDisable当成重置业务的信号；正式切场景销毁仍需清理Presenter/订阅。
- Render/Refresh只渲染，不再次招募/抽失败保留项/推进clears/重置待保存状态；显式业务入口才开始新事务。

## 4. 业务状态与失败规则

### 4.1 招募与交易

`RetainerRecruitmentModel`承载busy与同帧防重，保存前保留payload快照；招募失败或SaveProgress失败保持原业务数据，异常也回滚。当前会话使用已选槽，无会话原型仍走槽3。Presenter只映射状态和反馈，不在View内使用JsonUtility回滚。

`MarketModel`提供报价/阻断原因与Buy/Sell入口，复用MarketService的容量、钱财、买失败退款及保存规则。Presenter渲染CanBuy/CanSell与原因；按钮灰显仅是反馈，Model/Service执行时仍必须复核，不能只依赖UI状态。

### 4.2 结算事务跨覆盖保留

`RunSettlementModel.Begin(win)`按RunSeed、runs和Win识别当前事务；同局重复Begin不重新初始化。Current保存物品数量快照、已选/失败保留集合、`_applied`、PendingSave和Completed。Presenter重建时取同一个Current，覆盖后仍保留选择与待保存提示。

首次Confirm只应用一次入库/通关推进，随后设置PendingSave并保存。锁盘失败保留已应用的内存事务，下一次Confirm只重试保存；不能套用“招募保存失败立即回滚”来重做结算。保存成功才Completed；再次Confirm被拒绝。Presenter先关闭旧结算页，再调用返回局外，防止事件关闭后露出已完成结算。

当前失败保留比例、随机口径、时代/深度推进及物品入库失败处理均沿用已有行为。本轮不借分层改造调整数值/Repeatable，也不扩展触发条件。

### 4.3 事件与设置

事件队列及结果仍以EventService为权威。已选择待继续的事件被Market覆盖后，新的EventPresenter读取Resolved与CurrentResult，不能再给一次奖励。Continue才消费队首；一次性持久化规则保持。

SettingsEditingModel拥有一次编辑事务，EnsureEdit只BeginEdit一次。面板被覆盖而仍在UI栈中时保留Model，Presenter重建后继续使用Pending；真正应用/放弃退出时Finish，再释放Model。Dispose取消重绑定，未完成编辑恢复Applied，音频预览随放弃恢复。

设置行保留原SettingsData绑定，通过SettingsPresenter门面访问Pending、通知更改、预览音量与绑定输入。**这是兼容折中**：该部分尚未全部转换成不可变State＋细粒度Intent，不能宣称设置DTO完全隔离；生成配置DTO未修改。

## 5. 浮层关闭与HUD保护

FamilyTree/Village旧关闭按钮只Hide会留下UI栈；本轮改为发Close意图并正确Pop。营地关闭需要恢复战斗输入/画面。当前代码加入 `BasePanel.ReturnsToGameplayOnClose`，UIManager Pop后按该标记隐藏局外栈；游戏HUD Canvas使用 `GameplayCanvas` 标记，隐藏局外Canvas时跳过它，避免营地关闭同时隐藏HUD。

以上为当前源码改动事实；主代理尚在收尾复测该关闭链路。不得从“能打开营地且输入被禁用”推导“全部关闭/HUD恢复验证已通过”。这项最终验证留在§7。

## 6. 已完成验证与证据范围

证据路径以下均以主工程Assets为基准，已复读原文；操作证据由主代理在Unity中产生。策划文档代理未重复操作Unity。

| 检查 | 实际证据 | 结论与边界 |
| --- | --- | --- |
| Presenter/Model断言 | `QA/MVP/presenter-model.txt`：**Assertions=26；expectedSaveFaults=2；slot=3；fakeViews=true** | 假View与真实服务/文件锁路径通过；不是25个玩家UI测试 |
| 生命周期 | 重复Attach只订阅/Render一次；Refresh一次Render一次；重复Dispose无残留；Disposed不能重新Attach | 同上PASS；覆盖重建另有真实面板证据 |
| 交易 | 20个Buy意图：钱500→300、物品20；20个Sell意图：钱300→400、物品0；不可买状态及拒绝意图保持数据 | 同上PASS；不是20次真人鼠标连点 |
| 事件/招募 | 重复选项一次奖励；P重建恢复结果；Continue只关闭一次；同帧招募一次且写槽3 | 同上PASS |
| 村庄/结算 | 建造开始与取消通过；结算带出上限；两次锁盘确认只应用一次，保持待保存且不关闭；P重建保留选择/提示；解锁重试不重复入库/clears；已完成拒绝再Confirm | 同上PASS，故意保存错误2次需与未知Console错误区分 |
| 既有服务回归 | `QA/MVP/services-regression.txt`：**Assertions=86；expectedIOErrors=1；slot=3** | 86项既有服务断言重跑通过；该1次为刻意保存故障 |
| Settings真实面板 | `QA/MVP/ui-flow.txt`：applied=36、pending=37、audio=37；Market覆盖恢复pendingPreserved=True、appliedUnchanged=True；放弃后appliedRestored=True、audioRestored=True | 实际控件/音频编辑路径通过；与假View断言分开 |
| 菜单/局外View | StartBtn→槽3→WorldMapPanel，root栈1；Inventory rendered=True、textLabels=5；FamilyTree rows=16、Village rows=5；Era面板主代理已确认可渲染 | ui-flow.txt及主代理面板观察；行/按钮数包含关闭或根控件，不等同于16个节点、5种建筑 |
| 正式派遣 | LoadingScene→WorldScene，4人=玩家1＋AI3，stage=1/runs=1；Camp可见且playerInputDisabled=True | ui-flow.txt；不冒充营地关闭恢复已最终验收 |
| 胜利结算与覆盖 | GM清3场；结算选1项，被Market覆盖重建selected=1；确认返回EnterScene，clears=1/runs=1 | ui-flow.txt，正式场景与真实结算面板通过；清怪由GM，不代表完整手工战斗通关 |
| 事件与失败返回 | 1001物资50，Market覆盖后resolved=True；继续root栈1；第二局全倒确认返回EnterScene，clears=1/runs=2/wounded=4 | ui-flow.txt，对应数据落盘由主代理核验 |

本节的Settings/菜单/结算测试使用实际Unity面板和控件，不把它们等同于真人完成所有键鼠/手柄输入路径。预期锁盘错误单列，不能将这些测试描述成“Console全程0 error”。

## 7. 遗留与最终门禁

| 项目 | 当前状态／后续要求 |
| --- | --- |
| 最新编译门禁 | 最后改动已Refresh并等isCompiling=False，CompilationPipeline错误0；正常真实UI回归Console错误0。最终26断言的2条预期锁盘错误已存expected-save-faults.json后清理日志，再检查构建结果 |
| 构建与工程恢复 | 最新Windows构建0错误0警告；独立Player5360读档通过。已退出Play并恢复TestScene、还原动态字体测试缓存；restored.txt确认槽1哈希未变、槽3原档恢复、无.tmp残留；未提交推送 |
| 浮层关闭与HUD | 已通过：两个CloseButton均回到WorldMap根栈1；营地关闭outsideDispatchHidden=True、teamHudVisible=True、depthHudVisible=True、inputRestored=True，随后正式撤退返回EnterScene/playerCount0 |
| ModelHub与IModel | 本轮未实现统一ModelHub，也未迁出Recorder中的旧IModel；新运行时Model按所在事务/面板显式拥有，不能引用历史草稿说已统一托管 |
| 业务服务位置 | GameBase/Meta仍有业务服务，多个Presenter直接复用这些服务；后续可逐模块收敛模型边界，本轮未搬空GameBase |
| 设置行DTO | 保留通过Presenter门面的Pending绑定，不是全部不可变快照；需后续按控件分批细化 |
| Unity适配与可测试性 | RunControlModel仍查找Unity战斗对象，部分Model使用Time/JsonUtility，导航Presenter使用UIManager；尚未全面注入框架接口，不宣称全纯领域单元测试 |
| 状态重置 | RunSettlementModel.Current校验payload引用及单局Seed/runs；SubsystemRegistration清运行态；最终26断言覆盖相同Seed/runs切换到新会话不复用旧事务 |
| 原型表现与数值 | RunTerrainToast保留纯展示；本轮不改Excel生成DTO、配置表结构、数值、事件Repeatable，后续玩法平衡另行裁决 |

## 8. 变更记录

| 日期 | 变更 | 变更人 |
| --- | --- | --- |
| 2026-10-22 | 首版。按当前源码记录Presenter接口、16个View接入、运行时Model和服务复用；登记26项P/M断言、86服务回归及真实面板正式流程证据；明确设置DTO折中、ModelHub未落地及最终编译/构建/浮层关闭/恢复待补。未修改冻结04架构文档、代码或配置。 | 程序记录（策划子代理整理） |

## 9. 最终补充证据

- `boundaries.json`：Presenter17文件、View24文件、手写Model10文件（含原有DialogueRuntimeData）扫描通过；P无控件/直接IO，View无业务服务单例调用，Model无UI栈导航引用。此为源码边界回归守卫，不冒充Normify策略验证或完整依赖图证明。
- `ui-flow.txt`补充：村庄和家族树CloseButton均正确Pop到根栈1；扎营关闭后隐藏局外Dispatch、保留两种HUD、恢复角色输入，随后撤退返回EnterScene，玩家对象0。
- 实际门客按钮通过Intent→P→Model：门客内存4→5、磁盘5。实际市场按钮成交Crafting plans，钱财500→480→490；先前日志中的item1=0是无关条目的持有数，不能解读为买了Item1。`MarketEnglish.png`是另一次买入后的界面复查（Money480／Slots1/8），九行、名称、结果与关闭按钮完整；截图前聚焦Game View并请求完整Canvas重绘。
- 新增`Scripts/Runtime/MVP/README.md`作为程序目录内使用说明；一次性源码迁移脚本已移除，保留可重复运行的`QA/MVP/check_boundaries.py`及Editor验收类。

最新Windows64 Development Build：Succeeded，0 errors／0 warnings，19.4595585s，401972704 bytes；输出C:/Users/26751/AppData/Local/Temp/ChaosMvpBuild/Project_Chaos.exe。build.json保留原始工具结果。独立headless Player5360退出码0，goods50／firedCount1／pendingCount0／passed=true（player-process.json），仅证明数据读回，不代替成包真人通关。
最终恢复：槽1 SHA256=`DF3EE8B45AB74B073EA8529D237B11D59293C33141988C3BA8C64B2079C82360`，与本轮测试前一致；槽3恢复原始字节，SHA256=`7C3001A4A634AE0CE76DC4EA843C656E1DB5B5904CE5939A8A3C493D3F06F5FD`。恢复后无Slot3.json.tmp，Unity停止Play、TestScene无未保存场景修改。最后编译错误0，Console错误0；预期锁盘错误单列于验收文件。原有W5代码、文档与两张Excel在途改动继续保留，未提交、推送或改变分支。