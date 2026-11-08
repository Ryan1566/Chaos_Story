# B_运行时UI素材清单（数据盘点）

> 产出方：美术需求数据盘点员（runtime-ui）
> 盘点范围：`Assets/Scripts/Runtime/MVP/View/`（41 个 .cs）、`Scripts/Runtime/3C/`、`GameBase/UIManager/`、`GameBase/DialogueManager/`、`GameBase/LoadingManager/`、`Assets/Resources/UIPanels/`（20 个 prefab）、`Assets/Resources/UI/`、`Assets/Art/UI/`（164 张 png）
> Unity 2022.3.57f1c2 / 2D / 参考分辨率 1920×1080（部分 Canvas 1366×768 验收）
> 所有代码位置均为真实行号（逐文件读取后记录），未做推测。

---

## 0. 硬约束（写需求时必须整段照搬）

1. **新素材一律不得含任何可读文字、数字、字母、标点、伪文字字形、货币/星级等符号**，全部留空；仅允许纯几何形状。代码里的文字一律由 TMP 运行时绘制，素材里出现任何字形都会与 TMP 叠印。
2. 代码里用 `Graphic` / `Image` + 纯色拼出来的元素，**若只是色块则判「不需要新素材」**；确需贴图的一律判「需要但极简（仅圆角/描边）」，不加大面积纹理、不加渐变、不加高光。
3. 本工程**没有任何一处**把 Sprite 用作"带文字/带数字的成品图"；所有数值、计数、时辰、键位都是 TMP 文本叠加。素材侧只需提供容器（底/框/环/条/点）。
4. **复杂形状件（圆形扇区、圆环、多段圆、旋转连线）新素材只保留「边框 + 极浅淡基础纹理」，不加明显装饰**——因为它们会旋转、缩放、着色、按 4°/20 段细分，装饰会被拉伸、错位、随旋转抖动。
5. 现有 `DefaultSlices.png` 已证明正确方向：**纯白纯几何平涂、无描边无渐变、靠 `Image.color` 运行时着色**。新轮盘/环件必须沿用该范式（白色或近白中性灰，单色，可着色）。
6. 尺寸口径：**代码里写死的 `sizeDelta` 是 1920×1080 参考分辨率下的像素**；prefab 根全部 `stretch` 到父 Canvas，因此「根尺寸」记「全屏 stretch」。

### 0.1 编码与读取真值（后续同学注意）
- `Assets/Scripts/**` 为混合编码：GBK(936) / UTF-8+BOM / UTF-8 无 BOM 共存。`read` 工具遇 GBK 会报 invalid UTF-8，需 `[Text.Encoding]::GetEncoding(936).GetString([IO.File]::ReadAllBytes(path))`。
- 已确认 GBK 文件：`DialogueManager.cs`、`DialogueData.cs`（其余 View 均为 UTF-8）。

### 0.2 三条最重要的资产侧事实（先看这个再看表）
- `Assets/Art/UI/` 164 张 png 中 **仅 26 张被引用，138 张零引用**。所谓"缺素材"多数其实是"素材已有但没接线"（见 §5）。
- `Assets/Resources/UIPanels/` 20 个 prefab 中 **17 个是空壳**（根 + 1 背景 Image + 1 TMP + BasePanel），其面板视觉 100% 由 C# 现场拼装。
- **任务书提到的 `SkillKey_CooldownRing`（技能冷却进度环）在本工程不存在**：`Scripts` 下 0 命中，prefab/asset 下 0 命中；`Art/UI/09_Run/SkillKey_CooldownRing.png` 文件存在但零引用。真实的"环形进度"只有 `RunCampPanel` 的 24 时辰楔形环。详见 §3 与 §5。

---

## 1. 运行时 UI 总览

| 运行时界面 | 所属脚本:行号 | 触发场景 | 根尺寸 | 是否需要新素材 |
|---|---|---|---|---|
| 通用 UI 构件工厂（Rect/Text/Button） | `GameplayUi.cs:9` / `:21` / `:38` | 全局（被 30+ 处调用） | 由调用方给 | 是（按钮底极简） |
| 深度 HUD（右上角） | `RunDepthHud.cs:153-189` | 局内 | Canvas 1920×1080，内容 520×40 + 按钮 180×48 | 否（纯文本+纯色按钮） |
| 交互提示条（底部居中） | `RunInteractionHud.cs:16-33` | 局内 | Canvas 1920×1080，Prompt 1000×70 | 否（纯文本） |
| 队伍血量 HUD（左侧） | `RunTeamHud.cs:157-254` | 局内 | Canvas 1920×1080，行 400×30 | 是（G-HUD-02 条） |
| 快捷物品栏 / 行囊槽 | `RunBagSlotView.cs:21-34` | 局内 | 槽尺寸由父级定，Icon 24×24 | 是（G-HUD-05 槽 + G-SLOT） |
| 地形/阵营 Toast | `RunTerrainToast.cs:191-236` | 局内 | Canvas 1920×1080，文本 900×48 | 否（纯文本+alpha 淡出） |
| **快捷选择轮盘** | `RadialMenuView.cs:28-95` | 局内（长按/滚轮唤起） | 根 620×620，环层 600×600 | **是（G-HUD-06/07，重点）** |
| 局内地图 | `RunMapView.cs:30-41`、`:114-132` | 局内 | Viewport 1052×410，图例层 1100×700 | 是（G-HUD-04 框 + G-BTN） |
| 局内地图节点标记 | `RunMapMarkerView.cs:15-122` | 局内 | Dot 10×10，头像框 36×36 | 是（G-HUD-09，程序化图形） |
| 队伍详情面板 | `RunTeamView.cs:101`、`:246-313` | 局内 | 1100×475 | 是（G-PANEL-03 + G-BAR-01/02） |
| 行囊 / 菜单（三页签） | `RunMenuPanel.cs:30-201` | 局内 | 1100×700 | 是（G-PANEL-01 + G-TAB-01） |
| 扎营 + 时辰轮盘 | `RunCampPanel.cs:43-253` | 局内 | 800×600，轮盘 440×440 | 是（G-PANEL-03 + G-HUD-07 楔形环） |
| 局内内容/收集卡 | `RunContentView.cs:18-105`、`:257-298` | 局内 | 920×620 | 是（G-CARD-01 + G-SLOT-02） |
| 局内结算 | `RunSettlementPanel.cs:33-141` | 局内 | 800×600 | 是（G-PANEL-03） |
| 事件面板 | `EventPanel.cs:30-133` | 局外/局内均触发 | 800×600 | 是（G-PANEL-03） |
| 大地图（俯瞰） | `WorldMapPanel.cs:183-309`、`:405-501` | 大地图 | Viewport anchor (0.025,0.15)-(0.72,0.86) | 是（G-HUD-08 + G-PANEL-01） |
| 大地图地点标记 + 详情卡 | `WorldLocationsView.cs:54-63`、`:295-330`、`:522-530` | 大地图 | 标记 30/38，卡片 380 宽 | 是（G-HUD-08 重点） |
| 大地图势力地名标签 | `WorldMapFactionLabelView.cs:33-46` | 大地图 | 纯文本 + 描边材质 | 否 |
| 村落世界内 HUD 标签 | `VillageHud.cs:36` | 大地图（世界空间） | 纯文本 | 否 |
| 村落面板 | `VillagePanel.cs:76-175` | 局外 | 全屏（stretch），行 30 高 | 是（G-ROW-01 + G-BTN-01） |
| 村政面板 | `VillageOfficePanel.cs:26-39` | 局外 | 880×560 | 是（G-PANEL-03） |
| 派遣（门客差事） | `DispatchPanel.cs:17-30`、`:314-320`、`:843` | 局外 | 全屏，卡片 112 高 | **是（已在用 DispatchVisuals，需补）** |
| 家族树 | `FamilyTreePanel.cs:26-137` | 局外 | TreeGraph 1300×800，节点 172×144 | **是（已在用 FamilyTreeVisuals，需补）** |
| 市场 | `MarketPanel.cs:30-149` | 局外 | 800×600 | 是（G-PANEL-03 + G-ROW-02） |
| 门客 | `RetainerPanel.cs:30-177` | 局外 | 全屏，行 58 高 | 是（G-ROW-01） |
| 行囊（背包） | `InventoryPanel.cs:47-60`、`:211-302` | 局外 | 全屏，行 46 高 | 是（G-SLOT-02 + G-ROW-02） |
| 时代进度 | `EraProgressPanel.cs:83-174` | 局外 | 全屏（ContentBox 运行时建） | 是（G-PANEL-03） |
| 政务任务 | `OfficeTasksView.cs:16-38`、`:145-154` | 局外 | 1120×760 | 是（G-PANEL-01 + G-TAB-01） |
| 设置面板（5 页） | `SettingPanel.cs:322` + `SettingsPage/*.cs` | 局外 | 全屏（stretch），键位行 1398.4×96 | 部分（主要靠 prefab 已接） |
| 存档 / 读档 | `SLPanel.cs:301` + prefab | 局外 | Box 900×400 | 部分（prefab 已接） |
| 主菜单 | `MainMenuPanel.cs:55` + prefab | 局外 | 全屏，按钮 200×60 | 部分（prefab 已接） |
| 暂停 | `PausePanel.cs:13-15` | 全局 | 480×270 卡 | 是（G-PANEL-07 + G-PANEL-03） |
| 对话 | `GameBase/DialogueManager/DialogueManager.cs:14-20`、`:303` | 剧内 | Inspector 绑定（prefab） | 部分（需补对话框底） |
| 加载 | `GameBase/LoadingManager/LoadingController.cs:46-49`、`:239-242` | 加载 | 条 1200×44（Fill 1192×30） | 是（G-BAR-03） |
| 面板入场/退场动画 | `GameBase/UIManager/UIPanelAnimator.cs:51-76` | 全局 | 组件驱动 | **否（纯代码动画）** |

> 不计入美术需求：`Scripts/Runtime/GM/GMPanel.cs`（内部调试面板，4 个 Image / 12 个运行时节点，纯色 + TMP，发布不启用）。

---

## 2. 分界面明细

### 2.1 通用 UI 构件工厂（GameplayUi.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| `Rect()` 空白容器 | `GameplayUi.cs:9` | 纯 `RectTransform` | 无 | 调用方给定 | 否 | — |
| `Text()` 文本 | `GameplayUi.cs:21` | `TextMeshProUGUI` | 字体 `RunTeamHud.ResolveFont()`（MSYH SDF），内边距 offsetMin(12,4)/offsetMax(-12,-4) | 调用方给定 | 否 | — |
| `Button()` 按钮底 | `GameplayUi.cs:38` | `Image` + `Button` | **纯色 `new Color(.18f,.32f,.48f,1)`**，`sprite = null`（`:43`） | 调用方给定 | 是（极简：圆角 + 1px 内描边） | `G-BTN-01` |
| 按钮文字 | `GameplayUi.cs:38` 内部 | TMP 子节点 | 白 | 撑满 | 否 | — |

> **结论**：`GameplayUi.Button` 是全工程被复用最多的按钮视觉（局内地图按钮、深度 HUD 菜单按钮、扎营按钮、结算按钮…），但它是**无 sprite 的纯蓝灰色块**，与 `Art/UI/01_Buttons/Button_Md_Normal.png`（设置/存档/主菜单在用）风格完全脱节。这是 §4 重复项表的第一条。

### 2.2 深度 HUD（RunDepthHud.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 独立 Canvas | `RunDepthHud.cs:153-161` | `Canvas` + `CanvasScaler` | sortingOrder=100，1920×1080，match 0.5 | 全屏 | 否 | — |
| 深度文字（第 N 层） | `RunDepthHud.cs:173-189` | TMP | 色 (0.96,0.97,1,1)，字号 26，TopRight | 520×40，位置 (-24,-24) | 否 | — |
| "行囊 / 菜单" 按钮 | `RunDepthHud.cs:166-171` | `GameplayUi.Button` | 纯色 (.18,.32,.48,1) | 180×48，位置 (-24,-80) | 是（极简） | `G-BTN-03` |

> **重要**：深度 HUD **没有环形、没有进度条**。它是"一行动态文本 + 一个按钮"。`Art/UI/03_Sliders_Bars/Bar_Fill_Depth.png` 与 `09_Run/SceneProgress.png` 零引用 → 现有"深度 HUD 条"的素材从未被使用。若美术想给深度做进度条，属于**新增设计**，不是补绘。

### 2.3 交互提示 HUD（RunInteractionHud.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| Canvas | `RunInteractionHud.cs:16-20` | `Canvas` | sortingOrder=80 | 全屏 | 否 | — |
| 提示文本（"按 E 交互"等） | `RunInteractionHud.cs:21-22` | TMP | 可用 白 / 不可用 (0.8,0.8,0.8)（`:33`），字号 26 | 1000×70，底部居中 (0,72) | 是（极简：半透明圆角条） | `G-OVL-02` |

> 现状是**裸文本浮在场景上**，无底板；在亮地形上可读性差。建议给一张极简 1000×70 的暗色半透明圆角底（α≈0.55），无任何装饰。

### 2.4 队伍血量 HUD（RunTeamHud.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 独立 Canvas | `RunTeamHud.cs:157-165` | `Canvas` | sortingOrder=99，1920×1080 | 全屏 | 否 | — |
| 标题"队伍血量" | `RunTeamHud.cs:171-177` | TMP | (0.85,0.9,1,1)，20 号 | 400×26，位置 (20,-72) | 否 | — |
| 行底（槽） | `RunTeamHud.cs:193` | `Image` 纯色 | **(0.06,0.07,0.09,0.55)** | 400×30（`RowWidth=400`/`RowHeight=30`，`:37-38`） | 是 | `G-HUD-02` |
| 血条填充 | `RunTeamHud.cs:207` | `Image` 纯色（宽度按比例，非 fillAmount） | 绿 **(0.3,0.85,0.4,0.55)** | 高度 30 内 | 是（极简：可九宫拉伸的条芯） | `G-HUD-02` / `G-BAR-01` |
| 血量分档色 | `RunTeamHud.cs:250-254` | 运行时改 color | >50% 绿、>25% 黄 (0.95,0.8,0.25,.55)、否则红 (0.9,0.35,0.25,.55)、倒地 (0.55,0.2,0.2,0.5) | — | 否（靠 tint） | — |
| 行距/数量 | `RunTeamHud.cs:35`、`:104` | 位置计算 | 最多 `MaxRows=4` 行，y = -104 - i*36 | 行距 36 | 否 | — |

> 行距 36 vs 行高 30 → 行间 6px 空隙。素材须做成**可九宫格拉伸**（左右端不缩放），因为条宽随血比例变化。

### 2.5 快捷物品栏 / 行囊槽（RunBagSlotView.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 槽底 | `RunBagSlotView.cs:21` | `gameObject.AddComponent<Image>()` | 三态：未解锁 (0.10,0.12,0.14)／有物品 (0.20,0.33,0.39)／空 (0.15,0.19,0.23)（`:33`） | 由父级 Grid 定 | 是 | `G-HUD-05` + `G-SLOT-01/03` |
| 物品图标 | `RunBagSlotView.cs:22` | `Image` | 物品 sprite，preserveAspect | 24×24，位置 (0,-3) | 否（用物品图） | — |
| 数量/名称文本 | `RunBagSlotView.cs:24` | TMP | 13 号；未解锁字色 (0.4,0.43,0.47)（`:34`） | 撑满 | 否 | — |

### 2.6 地形 / 阵营 Toast（RunTerrainToast.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| Canvas | `RunTerrainToast.cs:191-205` | `Canvas` + `CanvasGroup` | sortingOrder=101，raycaster 关闭 | 全屏 | 否 | — |
| 地形/阵营名文本 | `RunTerrainToast.cs:212-229` | TMP | **(1,0.96,0.82,1)**，34 号 | 900×48，顶部居中 (0,-96) | 是（极简：可选横向渐隐条） | `G-OVL-03` |
| 淡入淡出 | `RunTerrainToast.cs:179-188` | `CanvasGroup.alpha` | 停留 2.6s + 淡出 0.4s（`:26-32`） | — | 否 | — |

> 纯文本 + alpha 动画，**无任何底板**。若加底板必须用"上缘/下缘渐隐"的柔和条，避免硬边压在画面上。

### 2.7 快捷选择轮盘（RadialMenuView.cs + RadialMenuSkin.cs + RadialWedgeGraphic.cs）★ 本次重点

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 轮盘根 | `RadialMenuView.cs:28` | `GameplayUi.Rect` | 无 | **620×620** | 否 | — |
| 内环层 Primary | `RadialMenuView.cs:29` | `RectTransform` 容器 | — | **600×600** | 否 | — |
| 外环层 Secondary | `RadialMenuView.cs:30` | 同上 | — | **600×600** | 否 | — |
| 单片扇区 Entry | `RadialMenuView.cs:33` | `RectTransform`，`localRotation = Euler(0,0,-angle)`（`:77`） | — | **600×600**（整张画布，靠 alpha 形状出扇形） | 否 | — |
| 扇区图形 Background | `RadialMenuView.cs:35` | `AddComponent<Image>()` | **`DefaultSlices.png` 的 Primary_N / Secondary_N 切片**（`RadialMenuSkin.cs:8` 载 `UI/RadialMenu/DefaultSkin`） | 600×600 | **是（G-HUD-06）** | `G-HUD-06` |
| 扇区着色 | `RadialMenuSkin.cs:22-24`、`:31` | `Image.color` tint | normal **(.07,.14,.23,.96)**、selected **(.7,.48,.12,.96)**、disabled **(.13,.14,.16,.95)**；未配 selected/disabled 变体时 = normal + tint | — | 否（靠 tint） | — |
| 标签根 | `RadialMenuView.cs:37` | `RectTransform` | 无 | **100×90** | 否 | — |
| 物品图标 | `RadialMenuView.cs:38` | `Image` | 物品 sprite，preserveAspect | **30×30**，位置 (0,17) | 否 | — |
| 物品名 | `RadialMenuView.cs:40` | TMP | 17 号 | **112×38**，位置 (0,-12) | 否 | — |
| 数量 | `RadialMenuView.cs:41` | TMP | 15 号 | **90×20**，位置 (0,-34) | 否 | — |
| 快捷键角标 | `RadialMenuView.cs:42` | TMP | 17 号 | **50×22**，位置 (0,43) | 否（但需要一个键帽底，见下） | `G-CTRL-05` |
| 键帽底 | 无 | — | **代码里没有键帽底板**（只有裸文字） | — | 是（极简 50×22 方块） | `G-CTRL-05` / `Keycap_Square`（已有零引用） |
| 相对指针 | `RadialMenuView.cs:45` | `Image` | `DefaultSlices.png` 的 `Pointer` 切片（**4×4 实心方块**） | **8×8** | 是（G-HUD-07） | `G-HUD-07` |
| 指针着色 | `RadialMenuSkin.cs:26` | `Image.color` | **(1,.86,.4)** | — | 否 | — |
| 底部提示 | `RadialMenuView.cs:49` | TMP | 20 号 | **700×60**，位置 (0,-320) | 否 | — |
| 展开/收起动画 | `RadialMenuView.cs:89-95` | `localScale` + `MoveTowards` | 160ms（`/ .16f`）；primary `Lerp(1, .58, blend)`，secondary `Lerp(.65, 1, blend)`，`SetActive(blend>0)` | — | 否 | — |
| 整体自适应缩放 | `RadialMenuView.cs:95` | `_root.localScale` | `Min(1, Min(size.x/730, size.y/740))` → 1080p 下 = 1.0 | — | 否 | — |
| 片数上限 | `RadialMenuSkin.cs:7` | 常量 | `MaxEntries = 12` | — | — | — |
| 扇区角度 | `RadialMenuView.cs:71` | 计算 | `i * 360f / count` | — | — | — |

> 详细几何参数见 §3.1（含逐像素实测的内外半径、3° 视觉间隙、中心孔直径）。

### 2.8 局内地图（RunMapView.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 状态文本 | `RunMapView.cs:30`（绘制在 `:133-137`） | TMP | 18 号（白） | 390×36，位置 (24,0) | 否 | — |
| 缩小按钮 "−" | `RunMapView.cs:31` | `GameplayUi.Button` | 纯色 (.18,.32,.48) | 50×34，位置 (430,0) | 是（极简） | `G-BTN-05` |
| 放大按钮 "+" | `RunMapView.cs:32` | 同上 | 同上 | 50×34，位置 (488,0) | 是（极简） | `G-BTN-05` |
| "全图" | `RunMapView.cs:33` | 同上 | 同上 | 110×34，位置 (550,0) | 是（极简） | `G-BTN-05` |
| "定位队伍" | `RunMapView.cs:34` | 同上 | 同上 | 150×34，位置 (672,0) | 是（极简） | `G-BTN-05` |
| "图例" | `RunMapView.cs:35` | 同上 | 同上 | 200×34，位置 (850,0) | 是（极简） | `G-BTN-05` |
| 地图视口 | `RunMapView.cs:36-38` | `Image` + `RectMask2D` | **纯色 (.025,.035,.05,1)** | **1052×410**，位置 (24,-44) | **是（G-HUD-04 圆形/异形框，当前是直角矩形）** | `G-HUD-04` |
| 地图底图 | `RunMapView.cs:40-41` | `RawImage` | `model.Terrain` **运行时生成的 Texture**（非美术资产） | 撑满视口，可缩放 1~6×（`:76`） | 否 | — |
| 节点标记容器 | `RunMapView.cs:99` | `RectTransform` | — | 12×12 | — | — |
| 图例浮层遮罩 | `RunMapView.cs:117` | `Image` 纯色 | **黑 (0,0,0,.88)** | **1100×700**（挂 `transform.parent` 全屏） | 是 | `G-PANEL-07` |
| 图例框 | `RunMapView.cs:119` | `Image` 纯色 | **(.08,.12,.17)** | **630×640** | 是 | `G-PANEL-03` |
| 图例标题 | `RunMapView.cs:120` | TMP | 28 号 | 500×44，位置 (40,-18) | 否 | — |
| 7 条图例标记 | `RunMapView.cs:121-130` | `Image` 纯色块 | 12×12 色块（黄/蓝/红/青/浅黄/橙） | **12×12**，位置 (56,-100-i*45) | 是（应换成 §2.9 的 5 种 Glyph 小尺寸版） | `G-HUD-09` |
| 7 条图例文字 | `RunMapView.cs:121` | TMP | 20 号 | 500×44，位置 (100,-77-i*45) | 否 | — |
| 地形说明 | `RunMapView.cs:120-130` | TMP | 19 号 | 574×105，位置 (28,-420) | 否 | — |
| "关闭图例" | `RunMapView.cs:120-130` | `GameplayUi.Button` | 纯色 | 270×44，位置 (180,-560) | 是（极简） | `G-BTN-02` |

> **视口是硬边直角矩形 + RectMask2D**，没有任何边框贴图。任务里说的"小地图圆形框"在当前实现里**不存在**：`Art/UI/09_Run/MinimapFrame.png` 存在但零引用。是否改成圆形属设计决策，数据侧如实记录："当前直角，圆形框素材可用但未接"。

### 2.9 局内地图节点标记（RunMapMarkerView.cs + RunMapGlyph）★ 程序化复杂形状

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 圆点 Dot | `RunMapMarkerView.cs:15`、`:29` | `RunMapGlyph(Circle)` → **VertexHelper 20 段三角扇**（`:120`） | Player (1,.83,.16) / Ally (.25,.65,1) / Enemy (1,.23,.23) / Npc (.35,.95,.74)（`:36-47`） | **10×10**（Ladder/Objective 时 **18×18**，`:29`） | **是（G-HUD-09 极简圆点 + 光晕）** | `G-HUD-09` |
| 圆点光晕 | `RunMapMarkerView.cs:81` | 同上，同一 mesh 两圈 | 同色 α=**0.15** | 直径 = 尺寸 ×0.75 | 是（或代码保留） | `G-HUD-09` |
| 菱形头像框 | `RunMapMarkerView.cs:16-17` | `RunMapGlyph(Diamond)`，4 顶点手绘（`:101-108`） | 同色 | **36×36** | 是 | `G-HUD-09` |
| 菱形头像图 | `RunMapMarkerView.cs:18` | `RunMapGlyph(Portrait)`，菱形 UV 裁切 | 人物立绘 | **30×30**（Player 偏移 (0,27)，`:31`） | 否 | — |
| 梯子 | `RunMapMarkerView.cs:73-75` | 2 竖 + 4 横 Quad 拼接 | (.92,.82,.55) | 18×18 | 是（建议给单张小图） | `G-HUD-09` |
| 核心目标 | `RunMapMarkerView.cs:78` | 菱形 + 0.24 边长深色方块 | (1,.45,.16) + (.12,.09,.05) | 18×18 | 是 | `G-HUD-09` |
| 宝箱 | `RunMapMarkerView.cs:92-93` | 2 个 Quad | 默认 (.92,.69,.33) | 18×18 | 是 | `G-HUD-09` |
| 人物 fallback | `RunMapMarkerView.cs:97-98` | 1 圆（半径 `.13w`）+ 1 Quad（.44w×.18h） | 同色 | 18×18 内 | 否 | — |
| 变暗（倒地） | `RunMapMarkerView.cs:33` | `color *= (.55,.55,.55)` | — | — | 否 | — |

### 2.10 队伍详情面板（RunTeamView.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 面板底 | `RunTeamView.cs:101` | `Image` 纯色 | **(.09,.10,.13,.96)** | **1100×475** | 是 | `G-PANEL-03` |
| 左栏底 | `RunTeamView.cs:31-44` | `Image` 纯色 | (.12,.13,.17) | 左宽 430 | 是（可省，与面板同色调） | `G-PANEL-03` |
| 全身立绘预览 | `RunTeamView.cs:165-166` | `Image` / Spine | 角色立绘 | 左栏内 | 否 | — |
| 头像槽（4 个） | `RunTeamView.cs:189-190` | `Image` 纯色 | SlotColor (.20,.22,.28) | **64×64**，位置 (14+i*74, slotY) | 是 | `G-SLOT-02` |
| 头像 | `RunTeamView.cs:203-204` | `Image` | 人物头像，preserveAspect | **54×54** | 否 | — |
| Spine 预览 | `RunTeamView.cs:211-212` | `Image` | Spine 渲染 | **54×54** | 否 | — |
| 选中标记 | `RunTeamView.cs:235` | `Image` 纯色 | SlotSelectedColor (.34,.42,.24) | 宽 stretch × **4** 高 | 是（极简描边） | `G-SLOT-04` |
| 状态条（3 条：血/体力/内力） | `RunTeamView.cs:225-226` | `Image`，靠 `offsetMax` 裁剪（**非 fillAmount、无 sprite**，注释 `:487-489`） | Health (.86,.24,.24) / Stamina (.30,.78,.34) / Mana (.28,.50,.92) | 撑满，条高由 offsetMin(2,2)/offsetMax(-2,18) 定 | 是 | `G-BAR-01` |
| 右栏卡 | `RunTeamView.cs:246-247` | `Image` 纯色 | CardColor (.16,.18,.23) | **≈618×439** | 是 | `G-CARD-02` |
| XP 轨道 | `RunTeamView.cs:260` | `Image` 纯色 | 黑 α=.55 | 行内 | 是（与 Bar_Track 统一） | `G-BAR-02` |
| 属性行（3 行） | `RunTeamView.cs:279`、`:290` | `Image` 轨道 + TMP | 轨道黑 .55，AccentColor (1,.78,.22) | 行高 **38** | 是 | `G-ROW-01` |
| "+" 加点按钮 | `RunTeamView.cs:298-299` | `GameplayUi.Button` | 纯色 | **48×30** | 是（极简） | `G-BTN-05` |
| 技能占位区 | `RunTeamView.cs:313` | `Image` 纯色 | 黑 α=.35 | 右栏下部 | 是（此处正是 G-HUD-01 技能键该出现的地方） | `G-HUD-01` |

> **关键缺口**：`RunTeamView.cs:313` 的 `SkillsPlaceholder` 是"技能区占位"，而 `Art/UI/09_Run/SkillKey_Ready/Locked/Cooldown/CooldownRing.png` 四张全在但零引用。技能键视觉的真正归属在此，不在深度 HUD。

### 2.11 行囊 / 菜单（RunMenuPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 全屏遮罩 | `RunMenuPanel.cs:53` | `Image` 纯色 | **黑 .78** | 全屏 | 是 | `G-PANEL-07` |
| 内容框 | `RunMenuPanel.cs:54-55` | `Image` 纯色 | **(.055,.075,.10,1)** | **1100×700** | 是 | `G-PANEL-01` |
| 三页签（行囊/地图/队伍） | `RunMenuPanel.cs:184` | `GameplayUi.Button` | 选中 **(.28,.49,.62)** / 未选 **(.12,.20,.28)** | 顶部 | 是 | `G-TAB-01` |
| 三区块 | `RunMenuPanel.cs:65`/`:93`/`:97` | 容器 | — | 均 1100×475，位置 (0,-110) | 否 | — |
| 拖拽幽灵 | `RunMenuPanel.cs:199-201` | `Image` 纯色 | **(.28,.48,.58,.9)** | **110×60** | 是（极简，需半透明 + 圆角） | `G-OVL-04` |

### 2.12 扎营 + 时辰轮盘（RunCampPanel.cs）★ 第二处真实环形件

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 主面板 | `RunCampPanel.cs:130-136` | `Image` 纯色 | **(0.10,0.11,0.14,0.98)** | **800×600** | 是 | `G-PANEL-03` |
| 标题"扎营" | `RunCampPanel.cs:139` | TMP | (0.96,0.97,1,1)，34 号 | 顶部 | 否 | — |
| 状态文本 | `RunCampPanel.cs:145` | TMP | 22 号 | — | 否 | — |
| 提示文本 | `RunCampPanel.cs:151` | TMP | **(1,0.85,0.3,1)**，20 号 | — | 否 | — |
| "完全治疗（1 药）" | `RunCampPanel.cs:157-163` | `GameplayUi.Button` | 纯色 (.2,.42,.72) / 禁用 (.22,.24,.28)（`:281`） | **300×56**，位置 (-170,200) | 是 | `G-BTN-02` |
| "补充减员（2 药）" | `RunCampPanel.cs:157-163` | 同上 | 同上 | **300×56**，位置 (170,200) | 是 | `G-BTN-02` |
| "调整时辰"入口 | `RunCampPanel.cs:157-163` | 同上 | 同上 | **340×48**，位置 (0,292) | 是 | `G-BTN-02` |
| "关闭" | `RunCampPanel.cs:157-163` | 同上 | 同上 | **300×56**，位置 (0,30) | 是 | `G-BTN-02` |
| 时辰层遮罩 | `RunCampPanel.cs:183` | `Image` 纯色 | **(0.05,0.06,0.09,1)** | 全屏 | 是 | `G-PANEL-07` |
| 标题"调整时辰" | `RunCampPanel.cs:173-206` | TMP | 30 号 | 700×38，位置 (0,-10) | 否 | — |
| 轮盘 tip | `RunCampPanel.cs:173-206` | TMP | (0.68,0.72,0.80,1)，16 号 | 700×26，位置 (0,-48) | 否 | — |
| 轮盘容器 | `RunCampPanel.cs:206` | `RectTransform` | — | **440×440**，位置 (0,-12) | 否 | — |
| **24 时辰楔形片** | `RunCampPanel.cs:220-221` | **`RadialWedgeGraphic`（程序化 mesh）** | **(0.10,0.18,0.28,0.96)** | 内 r=126 / 外 r=205，扫角 13°，步进 15° | **是（G-HUD-07 环件，极简）** | `G-HUD-07` |
| 整点按钮（24 个） | `RunCampPanel.cs:228-229` | `GameplayUi.Button` | 纯色，白色 17 号文字 | **34×30**，半径 166 | 是（极简键帽） | `G-CTRL-05` |
| 12 时辰名 | `RunCampPanel.cs:234-242` | TMP | 白 | **64×26**，半径 214 | 否 | — |
| 当前时辰 | `RunCampPanel.cs:247` | TMP | (0.72,0.76,0.84,1)，15 号 | 236×26，位置 (0,46) | 否 | — |
| 预览时辰 | `RunCampPanel.cs:248` | TMP | 白，20 号 | 236×32，位置 (0,14) | 否 | — |
| 原因/消耗说明 | `RunCampPanel.cs:249` | TMP | (1,0.85,0.3,1)，14 号 | 210×48，位置 (0,-38) | 否 | — |
| "确认切换" | `RunCampPanel.cs:251-253` | `GameplayUi.Button` | 纯色 (.2,.42,.72)（`:323`） | **210×44**，位置 (-118,18) | 是 | `G-BTN-01` |
| "返回扎营" | `RunCampPanel.cs:251-253` | 同上 | 同上 | **210×44**，位置 (118,18) | 是 | `G-BTN-02` |

> **轮盘与时辰环是两种完全不同的技术路线**：快捷轮盘用"整张 512 切片 + 旋转"，时辰环用"程序化 mesh 逐片 Configure"。素材侧要分别支持：前者给 12 张按片数变化的全画布切片；后者**只需要一张可着色的极简环形底/描边**（因为 mesh 自己画形状）。

### 2.13 局内内容 / 收集卡（RunContentView.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 卡片底 | `RunContentView.cs:51-54` | `Image` 纯色 | **(.055,.075,.10,1)** | **920×620** | 是 | `G-CARD-01` |
| 头像框 | `RunContentView.cs:56-65` | `Image` 纯色 | **(.12,.16,.20)** | **180×180** | 是 | `G-SLOT-02` |
| 头像 | `RunContentView.cs:56-65` | `Image` | 人物立绘，preserveAspect | **168×168** | 否 | — |
| 选项滚动区 | `RunContentView.cs:74-88` | `Image` + `RectMask2D` + `ScrollRect(Clamped,24)` | **(.035,.05,.065)** | 中段 | 是（极简） | `G-PANEL-03` |
| 提示文本 | `RunContentView.cs:95` | TMP | **(1,.83,.42)** | 中段 | 否 | — |
| 关闭按钮 | `RunContentView.cs:97-105` | `GameplayUi.Button` | 纯色 (.18,.32,.48) | **208×60**，右边距 32 | 是 | `G-BTN-02` |
| 选项行 | `RunContentView.cs:257-298` | `Image` 纯色 | 三态：可用 (.20,.33,.39)／已领取 (.11,.14,.16)／不可用 (.15,.17,.19)（`:238-246`） | 高 **96**（`ChoiceHeight`） | 是 | `G-ROW-01` |
| 选项图标框 | `RunContentView.cs:257-298` | `Image` 纯色 | (.16,.22,.27) | **56×56**，位置 (12,-20) | 是 | `G-SLOT-01` |
| 选项图标 | `RunContentView.cs:257-298` | `Image` | 物品图 | **44×44** | 否 | — |
| 选项文字组 | `RunContentView.cs:257-298` | TMP | Name 22 号白／(.58,.61,.65)；Count 18 号 (.85,.88,.92)；State 16 号；Detail 17 号 (.78,.82,.86)／(.46,.49,.53) | — | 否 | — |
| 行动按钮区 | `RunContentView.cs:257-298` | `Image` 纯色 | (.18,.32,.48) | **124×68** | 是 | `G-BTN-01` |

### 2.14 局内结算（RunSettlementPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 结算框 | `RunSettlementPanel.cs:105-114` | `new GameObject(..., typeof(Image))` | **(0.10,0.11,0.14,0.98)** | **800×600** | 是 | `G-PANEL-03` |
| 标题"局内结算" | `RunSettlementPanel.cs:118` | TMP | (0.96,0.97,1,1)，34 号 | 顶部 70 高 | 否 | — |
| 提示（"按 X 继续"） | `RunSettlementPanel.cs:127` | TMP | **(1,0.85,0.3,1)**，20 号 | 顶部 80 高 | 否 | — |
| 伤势行 | `RunSettlementPanel.cs:141` | TMP | **(1,0.55,0.45,1)**，20 号 | — | 否 | — |
| 结算行 | `RunSettlementPanel.cs:33-35` | 计算 | — | 行高 **42** | 是（建议 G-ROW-03） | `G-ROW-03` |

### 2.15 事件面板（EventPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 事件框 | `EventPanel.cs:105-114` | `Image` 纯色 | **(0.10,0.11,0.14,0.98)** | **800×600** | 是 | `G-PANEL-03` |
| 标题 | `EventPanel.cs:118` | TMP | (0.96,0.97,1,1)，34 号 | 顶部 | 否 | — |
| 队列提示 | `EventPanel.cs:127-133` | TMP | **(1,0.85,0.3,1)**，20 号 | 顶部 30 高 | 否 | — |
| 事件正文 | `EventPanel.cs:30-35` | TMP | 白 | 中段 | 否 | — |
| 选项按钮 | `EventPanel.cs:30-35` | `GameplayUi.Button` | 纯色 | 高 **52**，间距 **12** | 是 | `G-BTN-01` |

> 与 `RunSettlementPanel`、`MarketPanel`、`RunCampPanel` **完全同一套参数**（800×600 + 同色 0.10,0.11,0.14,0.98 + 34 号标题 + (1,0.85,0.3) 提示）→ 见 §4 重复项表。

### 2.16 大地图（WorldMapPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 顶部工具条（MetaEntries） | `WorldMapPanel.cs:183-203` | `Image` 纯色 | **(.07,.10,.14,.96)** | 顶部通栏 | 是 | `G-PANEL-01` |
| 提示区 | `WorldMapPanel.cs:203` | TMP | **(1,.85,.5)** | 工具条内 | 否 | — |
| 大地图视口 | `WorldMapPanel.cs:292-299` | `Image` + `RectMask2D` | **(.11,.09,.06,.94)** | anchorMin(.025,.15) / anchorMax(.72,.86) | **是（G-HUD-04 异形框）** | `G-HUD-04` |
| 地图底图 | `WorldMapPanel.cs:338-343` | `Image` | **`Art/PNG/GameWorld_Map/Map_tianxia_zhongyuan.png`**（唯一已接的美术底图） | 按 sprite 宽高比反算 | 否 | — |
| 地点点位（28×28） | `WorldMapPanel.cs:405-434` | `Image` | — | **28×28**（`iconBaseSize=28`，`:56`），选中改 localScale | 是 | `G-HUD-08` |
| 点位图标（有配置时） | `WorldMapPanel.cs:443-448`、`:53` | `Image` + Inspector `kindIcons` | 分类 Sprite | 28×28 | 是（`kindIcons` 当前未配满，见 `:499-501` 兜底） | `G-HUD-08` |
| 点位图标（兜底） | `WorldMapPanel.cs:454`、`:463-493`、`:499-501` | **`CreateCircleSprite()` 运行时生成白圆 Texture** | kind1 (1,.85,.25)／kind2 (.35,.80,1)／default (.75,.75,.78) | 28×28 | 是（同 §2.9 圆点） | `G-HUD-08` |
| 确认弹窗遮罩 | `WorldMapPanel.cs:213-216` | `Image` 纯色 | 黑 **.8** | 全屏 | 是 | `G-PANEL-07` |
| 确认弹窗框 | `WorldMapPanel.cs:213-216` | `Image` 纯色 | **(.09,.13,.18,1)** | **620×230** | 是 | `G-PANEL-03` |
| 弹窗正文 | `WorldMapPanel.cs:213-216` | TMP | 白 | **580×100** | 否 | — |
| 势力地名层 | `WorldMapPanel.cs:275` | `WorldMapFactionLabelView` | 见 §2.17 | — | 否 | — |

### 2.17 大地图势力地名标签（WorldMapFactionLabelView.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 地名标签 | `WorldMapFactionLabelView.cs:33-36` | `GameplayUi.Rect` + `TextMeshProUGUI` | 白字 + 材质描边 `_OutlineColor` 黑 .95（`:46`） | 由调用方给 | 否 | — |

### 2.18 村落世界内标签（VillageHud.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 村落世界空间标签 | `VillageHud.cs:36` | `GameplayUi.Rect` + TMP | 纯文字（世界空间 Canvas） | 调用方给 | 否 | — |

### 2.19 村落面板（VillagePanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 面板底 | prefab（空壳）+ 运行时代码 | `Image`（prefab 内） | 代码未改色 → prefab 默认白，**实机上是一块白** | 全屏 stretch | **是（G-PANEL-01，优先级高：当前无底色）** | `G-PANEL-01` |
| 行 | `VillagePanel.cs:76-83` | `RectTransform` + TMP | 白／(.72,.76,.82)；行高 **30**，宽 -48，位置 (24,y) | 高 **30** | 是 | `G-ROW-01` |
| 子行 | `VillagePanel.cs:110-116` | `RectTransform` + TMP | 高 **28** | 高 **28** | 是 | `G-ROW-03` |
| 主按钮 | `VillagePanel.cs:93`、`:142-150` | `Image` + `Button` | 可用 **(.20,.42,.72)**／禁用 **(.22,.24,.28)**（`:150`） | **180×30**，位置 (0,-34) | 是 | `G-BTN-05` |
| 次按钮（采集） | `VillagePanel.cs:96` | 同上 | 同上 | **180×30**，位置 (200,-34) | 是 | `G-BTN-05` |
| 块底 | `VillagePanel.cs:175` | `Image` 纯色 | **(.18,.21,.26)** | 行内 | 是 | `G-CARD-03` |

### 2.20 村政面板（VillageOfficePanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 遮罩 | `VillageOfficePanel.cs:26` | `Image` 纯色 | 黑 **.68** | 全屏 | 是 | `G-PANEL-07` |
| 政务卡 | `VillageOfficePanel.cs:27-28` | `Image` 纯色 | **(.055,.075,.10,1)** | **880×560** | 是 | `G-PANEL-03` |
| 标签文本 | `VillageOfficePanel.cs:39` | `GameplayUi.Text` | 白 | 卡内 | 否 | — |

### 2.21 派遣（DispatchPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 面板底 | `DispatchPanel.cs:314-315` | `Image`，`sprite = null` | 纯色（调色板 Dark/Panel，`：17-30`） | 全屏 stretch | 是 | `G-PANEL-01` |
| 视觉资源包 | `DispatchPanel.cs:320` | `Resources.Load<DispatchVisuals>("UI/DispatchVisuals")` | `Card_Normal/Selected/Disabled`、`DispatchSlot_Empty/Filled/Selected`、`FamilyNodeDetail`、`PortraitFrame_Square` | — | **已接** | `G-CARD-01/02`・`G-SLOT-02/03` |
| 分隔线 | `DispatchPanel.cs:371` | `Image` 纯色 | GoldDim | 1px 通栏 | 是（极简 1px） | `G-ORN-02` |
| 搜索框 | `DispatchPanel.cs:392-396` | `Image` + `Outline` | 底 Panel；描边 GoldDim α=.55，effectDistance (1,-1) | 全宽 | 是 | `G-CTRL-01` |
| 列表视口 | `DispatchPanel.cs:452-453` | `Image` 纯色 | 黑 α=.02 | 中段 | 是（极简内凹底） | `G-PANEL-03` |
| 滚动条 | `DispatchPanel.cs:473-474` | `Image` 纯色 | **(0.06,0.05,0.035,.8)** | 右侧 | 是 | `G-CTRL-03` |
| 差事卡 | `DispatchPanel.cs:511-515`、`:843` | `Image` + `Outline` | Card (.18,.14,.095)／选中 (.31,.235,.125)；描边 GoldDim α=.45、1px | **宽 -10 × 高 112**，步距 **120**（`:787`） | **是（卡面需覆盖件）** | `G-CARD-01/02` |
| 详情卡背景 | `DispatchPanel.cs:578` | `Image` | `_visuals.DetailBackground`（`FamilyNodeDetail`） | 右侧 | 已接 | `G-CARD-04` |
| 头像框 | `DispatchPanel.cs:585-586`、`:594` | `Image` | `_visuals.PortraitFrame`（`PortraitFrame_Square`），缺图色 (0.045,0.035,0.025,.95) | 方框 | 已接 | `G-SLOT-02` |
| 槽位 | `DispatchPanel.cs:690-691` | `Image` + `Outline` | 描边 1px；anchor (x0,0.075)-(x1,0.155) | 横向 stretch | 已接 | `G-SLOT-03` |

> 派遣是**唯一"半个 prefab 化"的局外大面板**：它已有 ScriptableObject 视觉包（7 张 sprite），但底色、分隔线、搜索框、滚动条仍是代码纯色 → 见 §4。

### 2.22 家族树（FamilyTreePanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 背景 | `FamilyTreePanel.cs:50` | `Image` 纯色 | **(.08,.07,.06,1)** | 全屏 | 是（建议给族谱专用纹理底） | `G-PANEL-01` |
| 视觉资源包 | `FamilyTreePanel.cs:49` | `Resources.Load<FamilyTreeVisuals>("UI/FamilyTreeVisuals")` | `FamilyNodeDetail`、`FamilyNode_Max/Unlocked/Locked`、`FamilySection_Normal/Selected` | — | **已接** | `G-CARD-04`・`G-TAB-01` |
| 视口 | `FamilyTreePanel.cs:61` | `Image` + `RectMask2D` | Ink (.13,.115,.095) | 中段 | 是 | `G-PANEL-03` |
| 树画布 | `FamilyTreePanel.cs:63` | `RectTransform` | — | **1300×800** | 否 | — |
| **节点** | `FamilyTreePanel.cs:135`、`:107` | `Image` + `Outline`(2,-2) | 状态 sprite；缺图色 (.24,.20,.14) | **172×144** | **是（G-CARD-04 家族节点专用）** | `G-CARD-04` |
| 详情框 | `FamilyTreePanel.cs:68` | `Image.Type.Sliced` | `DetailBackground`；缺图色 (.19,.16,.12) | 侧栏 | 已接 | `G-CARD-04` |
| 分支按钮 | `FamilyTreePanel.cs:90` | `Image.Type.Sliced` | `SectionSelected/SectionNormal` | 顶部 | 已接 | `G-TAB-01` |
| 节点按钮 | `FamilyTreePanel.cs:182` | `Image` 纯色 | **(.32,.26,.16)** | 节点内 | 是 | `G-BTN-05` |
| **连线** | `FamilyTreePanel.cs:114-117` | `Image` 单色矩形 + **旋转** `sizeDelta=(length,3)`、`Euler(0,0,Atan2)` | Gold (.87,.70,.35)／可用 (.35,.6,.42)／锁定 (.36,.31,.23) | 长 = 两点距离，**粗 3** | **是（G-ORN-02 极简线/带节点圆点）** | `G-ORN-02` |

### 2.23 市场（MarketPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 市场框 | `MarketPanel.cs:127-136` | `Image` 纯色 | **(0.10,0.11,0.14,0.98)** | **800×600** | 是 | `G-PANEL-03` |
| 标题"市场" | `MarketPanel.cs:140` | TMP | (0.96,0.97,1,1)，34 号 | 顶部 | 否 | — |
| 钱数 | `MarketPanel.cs:149` | TMP | **(1,0.86,0.45)**，22 号 | 顶部右 | 否 | — |
| 商品行 | `MarketPanel.cs:30-33` | 计算 | 行高 **32**，步进 **36**；色 Normal (.92,.93,.96)／Dim (.62,.66,.72)／Warn (1,.72,.25)／Ok (.45,.90,.55) | 高 **32** | 是 | `G-ROW-02` |

### 2.24 门客（RetainerPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 行 | `RetainerPanel.cs:125-132` | `RectTransform` + TMP | 行高 **58**，宽 -48，步距 **62**，位置 (24,-8-i*62) | 高 **58** | 是 | `G-ROW-01` |
| 子行 | `RetainerPanel.cs:151-158` | `RectTransform` + TMP | 高 **30** | 高 **30** | 是 | `G-ROW-03` |
| 状态色 | `RetainerPanel.cs:30-34` | 运行时改 color | 可用 (.30,.85,.40)／受伤 (1,.36,.32)／低置信 (1,.82,.25)／正文 (.92,.93,.96)／次要 (.72,.76,.82) | — | 否 | — |
| 列表区 | `RetainerPanel.cs:171-177` | 容器 | offsetMin(0,90)/offsetMax(0,-142) | 中段 | 否 | — |

### 2.25 行囊（背包，InventoryPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 行 | `InventoryPanel.cs:47-48` | 计算 | 行高 **46**，顶部留 8 | 高 **46** | 是 | `G-ROW-02` |
| 品质块（6 档） | `InventoryPanel.cs:55-60`、`:211` | `Image` 纯色 | (0.85,.35,.40) 红／(.55,.62,.45) 绿／(.40,.55,.80) 蓝／(.70,.55,.85) 紫／(.90,.70,.35) 金／(.60,.62,.66) 灰 | **36×36** | **是（应换 IconFrame_* / Slot_Frame_* 品质框，现有 20 张全零引用）** | `G-SLOT-02` |
| 行文本 | `InventoryPanel.cs:222` | TMP | 白 | -96×36 | 否 | — |
| 操作按钮 | `InventoryPanel.cs:299-302` | `GameplayUi.Button` | **(.18,.21,.26)** | **160×48** | 是 | `G-BTN-01` |
| 视口 | `InventoryPanel.cs:150-153` | `Image`（全透明）+ `RectMask2D` | α=0 | 中段 | 否（当前就是隐形的） | — |

### 2.26 时代进度（EraProgressPanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 内容框（运行时建） | `EraProgressPanel.cs:83` | `new GameObject("ContentBox", typeof(RectTransform), typeof(Image))` | 未设色 → **默认白** | 代码内定 | **是（G-PANEL-03，优先级高）** | `G-PANEL-03` |
| 标题"时代进度" | `EraProgressPanel.cs:95` | TMP | 34 号 | 顶部 | 否 | — |
| 进度行 | `EraProgressPanel.cs:102-107` | TMP TopLeft | 22 号 | 列表 | 否 | — |
| 提示 | `EraProgressPanel.cs:110` | TMP | (1,.85,.3)，20 号 | 底部 | 否 | — |
| 按钮 | `EraProgressPanel.cs:172-174` | `GameplayUi.Button` | 纯色 (.2,.42,.72)，22 号文字 | 底部 | 是 | `G-BTN-01` |

### 2.27 政务任务（OfficeTasksView.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 遮罩 | `OfficeTasksView.cs:145-146` | `Image` 纯色 | 黑 **.68** | 全屏 | 是 | `G-PANEL-07` |
| 大卡 | `OfficeTasksView.cs:151-154` | `Image` 纯色 | **(.055,.075,.10)** | **1120×760** | 是 | `G-PANEL-01` |
| 4 页签 | `OfficeTasksView.cs:198-206` | `GameplayUi.Button` | TabColor (.15,.19,.25)／选中 **(.22,.40,.60)** | 高 **52** | 是 | `G-TAB-01` |
| 列表区 | `OfficeTasksView.cs:230-244` | `Image` 纯色 | PanelColor (.09,.115,.15) | 宽 **380** | 是 | `G-PANEL-03` |
| 详情区 | `OfficeTasksView.cs:273-306` | `Image` 纯色 | PanelColor | 右侧 | 是 | `G-PANEL-03` |
| 任务行 | `OfficeTasksView.cs:26-27` | 计算 | RowColor (.13,.16,.21)／选中 **(.22,.36,.52)** | 高 **92**，间距 **8** | 是 | `G-ROW-01` |
| 选项 | `OfficeTasksView.cs:333-336` | 全透明底 | α=0 | 高 **90**，间距 **8** | 否 | — |
| 页头 / 页脚 | `OfficeTasksView.cs:20-23` | 布局 | 高 100 / 56 | — | 否 | — |

### 2.28 设置面板（SettingPanel.cs + SettingsPage/ 6 个文件）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 面板本体 | prefab（**作者化，1.07MB / 143 组件 / 118 sprite**）+ `SettingPanel.cs:322` | Prefab 可视化 | `Panel_Main`、`Panel_Dialog`、`Button_Md_Normal`×57 | 全屏 stretch | 否（已接） | `G-PANEL-01` |
| 设置行 | prefab `SettingsRow` ×29 | `Image`（prefab） | `Art/UI/10_System/SettingsRow.png` | 键位行 **1398.4×96** | 否（已接，但**该 png 在孤儿普查里属"仅本 prefab 引用"**） | `G-ROW-01` |
| 页签 | prefab `Tab_Normal` ×4 | `Image`（prefab） | `Art/UI/02_Controls/Tab_Normal.png` | `Tab_Gamepad` anchor(.32,.15)-(.47,.85) | 否 | `G-TAB-01` |
| 滑条 | prefab `Slider_Track`×5 + `Slider_Fill`×5 + `Slider_Handle_Normal`×1 | `Slider`（prefab） | 上述三张 | anchor(.44,.25)-(.86,.75) | 否 | `G-CTRL-02` |
| 复选框 | prefab `Checkbox_Unchecked`×3 + `Slot_Selected`×8 | `Image`（prefab） | 上述两张 | `Checkmark` inset -20 | 否 | `G-CTRL-04` |
| 键位按钮 | prefab `KeyButton` | `Image`（prefab） | `Button_Md_Normal` | anchor(.32,.12)-(.66,.88) | 否 | `G-CTRL-05` |
| 手柄页行 | prefab `Row_Menu_Pad` | `Image`（prefab） | `Button_Md_Normal` | **100×100** | 否 | `G-CTRL-06` |
| 底部按钮区 | prefab `BottomBar` | `Image`（prefab） | `Panel_Dialog` | anchor(.26,.012)-(.74,.14) | 否 | `G-BTN-02` |
| 5 个页面逻辑 | `SettingsPage/AudioSettingsPage.cs`、`GameplaySettingsPage.cs`、`GraphicsSettingsPage.cs`、`KeybindBindingsPage.cs`、`KeybindSettingsPage.cs`、`SettingsPageBase.cs` | 运行时改属性 | 仅设文本/选项/绑定，**不新建视觉** | — | 否 | — |

> 设置面板是全工程**唯一完全美术化**的界面，可作为"目标完成度"的参照：底色、按钮、行、滑条、页签全部有贴图。

### 2.29 存档 / 读档（SLPanel.cs + prefab）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 主框 | prefab | `Image` | `Panel_Main` | **900×400**（Box） | 否 | `G-PANEL-01` |
| 对话框 | prefab | `Image` | `Panel_Dialog` | 全屏 stretch | 否 | `G-PANEL-03` |
| 存档记录列表 | prefab `RecordsList` | 容器 | — | 高 **200**，横向 stretch，anchor y=.5 | 否 | — |
| 记录单元 | `Resources/UIPanels/SubUI_Prefab/RecordCell.prefab`（20.6KB / 2 sprite） | `Image` | `Card_Normal`、`Card_Selected` | — | 否 | `G-CARD-03` |
| 返回 / 删除按钮 | prefab `ReturnBtn` / `DeleteBtn` | `Image` | `Button_Md_Normal` | **146.01×130.24** | 否 | `G-BTN-01` |
| 底部按钮条 | prefab `BottomBar` | `Image` | `Panel_Dialog` | anchor(.26,.012)-(.74,.14) | 否 | `G-BTN-02` |
| 确认组 | prefab `ConfirmGroup`（OkBtn/CancelBtn/ConfirmText） | `Image` + TMP | `Button_Md_Normal` | 百分比锚定（见 §0 表） | 否 | `G-BTN-02` |
| 逻辑 | `SLPanel.cs:301` | 运行时 | 仅数据填充 | — | 否 | — |

### 2.30 主菜单（MainMenuPanel.cs + prefab）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 背景板 | prefab | `Image` | `Panel_Main` | 全屏 stretch | 否 | `G-PANEL-01` |
| 4 个主按钮 | prefab StartBtn/SettingBtn/UpdateBtn/QuitBtn | `Image` + TMP | `Button_Md_Normal` | **200×60** | 否 | `G-BTN-01` |
| 按钮列表容器 | prefab `BtnList` | 容器 | — | **205.44×389.97** | 否 | — |
| 按钮文字 | prefab `Text (TMP)` | TMP | 白 | **130×60** | 否 | — |
| 标题 | prefab `Title`（anchor .5,1） | TMP | 白 | 0×0（自适应） | 是（若需印章式标题，`TitleSeal.png` 已有零引用） | `G-ORN-04` |
| 逻辑 | `MainMenuPanel.cs:55` | 运行时 | 仅按钮回调 | — | 否 | — |

### 2.31 暂停（PausePanel.cs）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 全屏遮罩 | `PausePanel.cs:13` | `GetComponent<Image>()` 改色 | **黑 .7** | 全屏 | 是 | `G-PANEL-07` |
| 暂停卡 | `PausePanel.cs:14-15` | `Image` 纯色 | **(.055,.075,.10,1)** | **480×270** | 是 | `G-PANEL-03` |

> 只有 30 行、两个元素，是"最该被 prefab 化"的界面。

### 2.32 对话（GameBase/DialogueManager/）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 对话面板 | `DialogueManager.cs:14-20`（Inspector 绑定） | Prefab 引用 | 由场景配置 | 由 prefab 定 | **是（`09_Run/DialogueBox.png` 已有零引用）** | `G-OVL-01` |
| 角色名 | `DialogueManager.cs:195` | TMP | `character.nameColor`（数据驱动） | — | 否 | — |
| 头像 | `DialogueManager.cs:198` | `Image` | `character.avatar` | — | 否 | — |
| 选项按钮 | `DialogueManager.cs:303` | **`Instantiate(optionButtonPrefab)`** | 由 prefab 定 | — | **是（`DialogueOption_Normal/Hover.png` 已有零引用）** | `G-BTN-02` |
| 继续指示器 | `DialogueManager.cs:14-20` 的 `continueIndicator` | 引用对象（可能带动画） | — | — | 是（极简三角/箭头） | `G-CTRL-07` |
| 打字机节奏 | `DialogueData.cs:27-28` | 数据 0.05s／2s | — | — | 否 | — |

> 对话是**唯一"全 Inspector 绑定、零运行时构建"**的界面 → 它的素材缺口纯粹是"没做"，不是"代码绕开了 prefab"。

### 2.33 加载（LoadingController.cs + LoadingScene.unity）

| 视觉元素 | 代码位置(文件:行) | 构建方式 | 当前 Sprite 或颜色 | 尺寸 | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|
| 背景图 | `LoadingController.cs:46`（`NodeBgImage="BgImage"`） | `Image`（场景内） | `Resources/UI/Loading/Loading_01~03.png` 3 张轮播 | 全屏 stretch | 否（已有 3 张） | `G-PANEL-07` |
| 交叉淡入图 | `LoadingController.cs:47`（`FadeImage`）、`:649-750` | 双 `Image` alpha 交叉 | 同上 | 全屏 stretch | 否 | — |
| 进度条轨道 | `LoadingController.cs:48`（`NodeProgressFill="ProgressBar/Fill"`） | 父 `ProgressBar` 的 `Image` | `Art/UI/10_System/Loading_BarTrack.png`（场景引用） | **1200×44**，位置 (0,150) | 是（轨道需按 1200×44 重做成九宫） | `G-BAR-03` |
| 进度条填充 | `LoadingController.cs:239-242`、`:762` | `Image.Type.Filled` + Horizontal + OriginLeft，`fillAmount=Clamp01` | `Loading_BarFill.png`（场景引用） | **1192×30**（Fill 的 sizeDelta (-8,-14)） | 是（填充需可水平裁切的纯色/极简纹理） | `G-BAR-03` |
| 阶段文字 | `LoadingController.cs:49`（`NodeStageText="StageText"`） | TMP | 白 | **1000×90**，位置 (0,80) | 否 | — |
| 兜底背景色 | `LoadingController.cs:77` | 代码 fallback | **(0.10,0.11,0.13,1)** | 全屏 | 否 | — |
| 失败态背景/条色 | `LoadingController.cs:78-79` | 代码 fallback | **(0.28,0.07,0.07)** / **(0.90,0.25,0.20)** | — | 否 | — |

> **加载界面没有沙漏、没有进度弧**。`10_System/Loading_Hourglass.png` 与 `Loading_TipBar.png` 存在但零引用。任务书里"加载沙漏/进度弧"在本工程属于**未实现**；真实件是"水平填充条"，且它的 fill 只能是水平裁切，不能是弧形图（代码硬编码 `FillMethod.Horizontal`）。

---

## 3. 复杂形状专项（非矩形 / 带曲线 / 带动画）

> **通用结论（每个复杂件都适用，必须写进需求）**：
> **因为形状复杂且可能带动画，新素材只保留边框 + 极浅淡基础纹理，不加明显装饰。**
> 具体原因：① 轮盘扇区是"整张方形图 + 旋转"，任何装饰都会随 15°/30° 步进错位；② 环形件由 mesh 逐片绘制或按 `fillAmount` 裁切，装饰会被切掉一半；③ 标记件在 1~6× 缩放下会被放大 6 倍，装饰会糊；④ 大量件靠 `Image.color` 着色（同一张图要变黄/蓝/红/青），带彩装饰会串色。

| 元素 | 形状描述 | 精确尺寸(像素) | 内/外半径或圆角 | 分片数或角度 | 是否带动画 | 备注 |
|---|---|---|---|---|---|---|
| **快捷轮盘·内环扇区（Primary）** | 圆环扇区（annular sector），中心孔透明，纯白平涂 | 切片画布 **512×512**，显示 **600×600** | 内 r **67.5**（画布）→ **79.1**（屏显）；外 r **154.6** → **181.2**；环宽 ≈ **102** | 按 `count` 动态 = **360/N**，N≤**12**；实测视觉扫角 = `360/N − 3.0°`（12 片=30°→27.0°；6 片=56.9°；4 片=87.1°；2 片=177.5°） | 是（160ms scale 1→0.58，`RadialMenuView.cs:89-95`） | 现有 `DefaultSlices.png` Primary_1..12 已是正确范式（纯白无描边）；**新素材必须保持"纯白/近白中性灰 + 无描边 + 无渐变"**，靠 tint 上色 |
| **快捷轮盘·外环扇区（Secondary）** | 同上，更外一层 | 同上 | 内 r **157.5** → **184.6**；外 r **240.0** → **281.3**；环宽 ≈ **96.7** | 同上 | 是（160ms scale 0.65→1） | 与内环外缘间距 **3.4px**（181.2 vs 184.6），新素材不得向外扩张 |
| **快捷轮盘·中心孔** | 完整圆形空心 | 直径 ≈ **158**（79.1×2） | r ≈ **79.1** | — | 是（随两层环一起缩放） | 中心孔内放 `TimeCurrent`/Hint 类文本；孔边**不需要描边**，现状是直接透明 |
| **快捷轮盘·中心指针** | 小方块（代码用 4×4 sprite 显示为 8×8） | **8×8**（sprite 原生 4×4） | 直角，无圆角 | 1 | 是（跟随选择旋转） | 建议做成小三角/菱形指针（8×8 或 12×12），单色可着色 tint (1,.86,.4) |
| **快捷轮盘·整环外描边（缺失）** | 目前**没有**外环描边/底托 | 建议 **600×600** | 外 r 281.3 处一圈 1~2px | 1 圈 | 否 | 新增件；单色细描边，禁用花纹（会被 6 档缩放拉糊） |
| **扎营·24 时辰楔形环** | 程序化 mesh 圆环扇区（`RadialWedgeGraphic`，4° 一步细分） | 容器 **440×440** | 内 r **126** / 外 r **205**（环宽 79） | **24 片**，片步进 **15°**，每片扫角 **13°**（间隙 **2°**） | 否（静态，点选高亮改 tint） | 键位/时辰名在 **r=166**（按钮）与 **r=214**（时辰名）两个不同半径上；素材只需一张环形底或纯描边 |
| **扎营·环外时辰名圈** | 24 个 34×30 键帽 + 12 个 64×26 文本，沿 r=166 / r=214 排布 | 键帽 **34×30**，名 **64×26** | r **166** / r **214** | 24 / 12 | 否 | 键帽需极简方块（可复用 `Keycap_Square`，已有零引用） |
| **局内地图·圆形标记 Dot** | 20 段三角扇实心圆 + 外圈光晕 | **10×10**（梯子/目标 **18×18**） | 实心 r = 尺寸×0.5；光晕 r = 尺寸×**0.75**，α=**0.15** | **20 段**（每段 18°） | 是（缩放 + 变暗 ×0.55） | 光晕与实心同色不同 alpha → 新素材若做成单图，必须自带软边，否则 6× 放大后 20 段折线可见 |
| **局内地图·菱形头像框** | 正方形旋转 45° 的菱形框 | **36×36** | 对角 36，边 ≈ 25.5 | 1（4 顶点手绘） | 是（头像随标记缩放） | 只需描边菱形，中间留给头像图 |
| **局内地图·菱形头像（裁切）** | 菱形 UV 裁切方图 | **30×30** | 对角 30 | 1 | 是 | 不用新素材（用人物立绘） |
| **局内地图·梯子标记** | 2 竖 + 4 横，共 6 个矩形拼成 | 18×18 | 竖条宽 = 0.12×18 ≈ **2.2**；横条高 ≈ **1.4** | 2 竖 + 4 横 | 是（缩放） | 建议直接给一张 18×18 单色小图，替代 6 个 Quad |
| **局内地图·核心目标标记** | 菱形 + 中心 0.24 边长深色方块 | 18×18 | 中心块 ≈ 4.3×4.3 | 1 + 1 | 是（缩放） | 单色可着色为橙 (1,.45,.16) |
| **局内地图·宝箱标记** | 2 个矩形（箱体 + 箱盖） | 18×18 | 直角 | 2 | 是（缩放） | 同上，建议单图 |
| **大地图·地点圆点（兜底）** | 运行时生成的白色实心圆 Texture | **28×28** | r = 14 | 代码生成圆 | 是（选中 localScale） | 28×28 时贴图圆会有明显像素边，建议给 64×64 再缩 |
| **大地图·地点标记（分类图）** | 分类 Sprite（13 张已存在） | **28×28**（`iconBaseSize`） | 由素材定 | 1 | 是（缩放 + 悬停放大） | `Resources/UI/WorldLocations/*.png` 13 张已就位，只需按 28×28 重绘构图 |
| **家族树·节点连线** | 细长矩形按两点角度旋转 | 长 = 两点距离，**粗 3** | 直角，无圆角 | 每对父子 1 条 | 是（旋转角度动态） | **必须单色**（会被染成金/绿/褐三态）；两端建议各带 3px 圆点收口，避免旋转后断头 |
| **加载·进度条填充** | 水平裁切（`FillMethod.Horizontal`，左起） | **1192×30**（轨道 1200×44） | 圆角 **≤ 15**（高 30 的一半） | 1 | 是（`fillAmount` 0→1） | **不能做成弧/圆**，代码硬编码水平填充；填充分辨率要够 1192 宽，纹样必须沿水平方向平铺 |
| **加载·进度条轨道** | 横向长条 + 内凹边 | **1200×44** | 圆角 ≤ **22** | 1 | 否 | 已接 `Loading_BarTrack.png`，但尺寸按 1200×44 需九宫 |
| **队伍血条（RunTeamHud / RunTeamView）** | 矩形条，靠宽度或 `offsetMax` 裁剪 | 行 **400×30**（HUD）／条高约 **20**（TeamView） | 圆角 **≤ 15 / ≤ 10** | 1 | 是（宽度随比例 0→满） | **不能用 `Image.Type.Filled`**：`RunTeamView.cs:487-489` 明确是 offsetMax 裁剪；素材必须左右端可九宫不缩放 |
| **面板入场动画（全局）** | 缩放 0.85→1 或 1.15→1、滑入 40px、Fade、逐项 stagger | 全屏 | — | — | **是**（`UIPanelAnimator.cs:51-76`；`PanelAnimType` 8 种） | **不需要任何素材**；但素材的九宫 border 必须留够（≥16px），否则缩放时边饰会变形 |
| **Toast / 提示淡入淡出** | alpha 0→1→0 | 900×48（地形 Toast） | — | — | 是（2.6s + 0.4s） | 只需可整体半透明的底；不要在底上画固定 alpha 的高光 |
| **`SkillKey_CooldownRing`（任务书要求）** | **本工程不存在** | — | — | — | — | `Scripts` 0 命中、prefab 0 命中；`Art/UI/09_Run/SkillKey_CooldownRing.png` 文件在但零引用。真实"技能冷却"视觉**未实现**，只有 `RunTeamView.cs:313` 的 `SkillsPlaceholder` 空块。详见 §5 |

**复杂形状件合计：19 件**（含 1 件"不存在/未实现"的如实登记）。

---

## 4. 运行时 vs 预制体 重复项

> 判据：同一视觉语义在 prefab 里有贴图、代码里又用纯色重画一遍，且两者**在同一屏可能同时出现或仅隔一次点击**。

| 视觉元素 | 预制体侧（有贴图） | 代码侧（纯色重画） | 冲突表现 | 统一建议 |
|---|---|---|---|---|
| **面板底板** | `00_Panels/Panel_Main.png`（SettingPanel / SLPanel / MainMenuPanel 在用） | 17 个空壳 prefab + 代码 `Image` 纯色：`(.10,.11,.14,.98)`（EventPanel/MarketPanel/RunCampPanel/RunSettlementPanel 四处完全相同）、`(.055,.075,.10,1)`（RunMenuPanel/RunContentView/PausePanel/VillageOfficePanel/OfficeTasksView）、`(.09,.13,.18)`（WorldMap 弹窗） | 同一款"深蓝黑面板"有 **3 套色值 + 1 张 png**；从大地图确认框（620×230）到扎营（800×600）到菜单（1100×700）底色差 1~2 个色阶，肉眼可辨 | 统一到 `G-PANEL-01`（全屏大板）/ `G-PANEL-03`（600~900 中板）；**代码侧改为 `sprite = Panel_Sub / Panel_Main`（九宫）**，颜色只做轻微 tint，色值收敛到 2 个 |
| **按钮** | `01_Buttons/Button_Md_Normal.png`（SettingPanel ×57、SLPanel ×4、MainMenuPanel ×4） | `GameplayUi.Button` 纯色 `(.18,.32,.48,1)`（30+ 处）；另有 `.2,.42,.72`（扎营/村落/时代）、`.22,.18,.125`（WorldLocations）、`(.18,.32,.48)`（RunContent 行动区） | **最严重的一条**：设置界面是墨金风贴图按钮，进局内后所有按钮变成**亮蓝灰方块**，风格断裂 | 统一到 `G-BTN-01/02/03/05`；`GameplayUi.Button` 增加可选 `Sprite` 参数并默认用 `Button_Md_Normal` 九宫；蓝色作为 tint 保留语义（主操作=金/朱砂，次操作=墨） |
| **卡片** | `05_Cards/Card_Normal/Selected/Disabled.png`（Dispatch、RecordCell 在用） | `RunContentView` 选项行、`DispatchPanel` 差事卡、`RunTeamView` 右栏卡、`VillagePanel` 块底、`OfficeTasksView` 任务行 —— 全部纯色 | 派遣里"卡片有贴图"，收集卡的"卡片是色块"，两种卡片在同一存档内都能看到 | 统一到 `G-CARD-01/02/03`；收集卡改用 `Card_Normal` + 品质 tint |
| **槽位 / 品质框** | `04_Slots/Slot_Selected.png`（SettingPanel ×8）、`Checkbox_Unchecked.png`（×3）、`Slot_Frame_Common/Epic/Legendary/Rare.png`、`IconFrame_*.png`、`QualityBar_*`（**20 张里 18 张零引用**） | `RunBagSlotView` 三态色块、`RunTeamView` 头像槽 64×64、`RunContentView` IconFrame 56×56、`InventoryPanel` 品质块 36×36（6 色硬编码） | 背包/行囊里的"品质"用 6 个纯色方块表达，而美术已经画好 4 档品质框 + 品质条，**一张没用** | 统一到 `G-SLOT-01..05`；`InventoryPanel.cs:55-60` 的 6 色降为 4 档（Common/Rare/Epic/Legendary）并直接换 `Slot_Frame_*` |
| **进度条 / 血条** | `03_Sliders_Bars/Bar_Track.png`、`Bar_Fill_*.png`（11 张零引用）、`Bar_Gloss`、`Bar_PhaseTick` | `RunTeamHud` 行底+填充纯色；`RunTeamView` sprite-less 裁剪条；`LoadingScene` 已用 `Loading_BarTrack/Fill`；`RunDepthHud` 无条 | 加载条有贴图，局内血条是纯色块；两者只隔一次场景切换 | 统一到 `G-BAR-01..04`（血/体力/内力 = `G-BAR-01`；XP/深度 = `G-BAR-02`；加载 = `G-BAR-03`；Boss/阶段 = `G-BAR-04`） |
| **全屏遮罩** | `06_Overlays/` 14 张（`Warn_*`、`Toast_*`、`Tooltip` 等，**全部零引用**） | 代码黑遮罩：`.68`（VillageOffice/OfficeTasks）、`.7`（Pause）、`.78`（RunMenu）、`.8`（WorldMap 确认）、`.88`（RunMap 图例）、`.98`（扎营时辰层 `0.05,0.06,0.09` 近似不透明） | **同一语义 6 个 alpha 值**；遮罩厚度不一会导致"层数感"混乱 | 统一到 `G-PANEL-07`：一层 0.68~0.72 的通用遮罩 + 需要聚焦时再叠一层 0.35 |
| **标题栏 / 标题文本** | `00_Panels/Panel_TitleBar.png`（零引用） | 所有面板标题为裸 TMP：34 号（Event/Market/Settlement/Camp/Era/MainMenu 标题）、30 号（RunContent/RunMap 图例）、28 号（RunMap 图例标题） | 20+ 个界面标题**没有任何标题栏承载**，字号 28/30/34 三档混用 | 统一到 `G-PANEL-01` 的标题槽位 + `G-ORN-02` 分隔线；字号收敛为 30（一级）/22（二级） |
| **滚动条** | `02_Controls/Scrollbar_Track.png` + `Scrollbar_Thumb.png`（Track 仅 SettingPanel 用 1 次，Thumb 零引用） | `DispatchPanel.cs:473-474` 纯色滚动条；`RunMapView`/`RunContentView` 只有 `RectMask2D` 无滚动条 | 派遣有滚动条（纯色），设置也有（贴图），两条滚动条样式不同却同屏上下切换 | 统一到 `G-CTRL-03`；一个 prefab 化的 Scrollbar 复用给所有列表 |
| **页签** | `02_Controls/Tab_Normal.png`（SettingPanel ×4）、`Tab_Selected.png`（零引用） | `RunMenuPanel`（.28,.49,.62 / .12,.20,.28）、`OfficeTasksView`（.22,.40,.60 / .15,.19,.25）、`FamilyTreePanel`（用 `FamilySection_*`） | 三套页签、三种选中色，其中设置页的 `Tab_Selected.png` 甚至没接线 | 统一到 `G-TAB-01`；选中态用 `Tab_Selected` 九宫，色差用 tint 表达语义（局内=蓝、局外=金） |
| **头像框** | `05_Cards/PortraitFrame_Square/Bust/Round.png`（Square 仅 Dispatch 用） | `RunTeamView` 头像槽 64×64 纯色、`RunContentView` 头像框 180×180 纯色、`RunMapMarkerView` 菱形框（程序化） | 一个角色在"派遣/队伍/收集/地图"四个界面有四种头像框 | 统一到 `G-SLOT-02`；菱形地图框单独走 `G-HUD-09`（形状不同，允许例外） |
| **提示 / 浮层条** | `06_Overlays/Toast_Info/Success/Warning/Fail.png`、`Tooltip.png`、`Warn_*.png`（全部零引用） | `RunTerrainToast`（裸文本）、`RunInteractionHud`（裸文本）、`RunContentView` 提示（裸文本） | 美术画了 4 种 Toast 底 + Tooltip，代码**一个都没用**，全靠裸文字 | 统一到 `G-OVL-01..05`；最少先接 `Toast_Info` 与 `Tooltip` 两张 |
| **记录行（列表行）** | `SubUI_Prefab/RecordCell.prefab`（`Card_Normal/Selected` 2 sprite） | `VillagePanel`（30/28 高）、`RetainerPanel`（58/30 高）、`MarketPanel`（32 高）、`InventoryPanel`（46 高）、`OfficeTasksView`（92 高）、`RunSettlementPanel`（42 高）、`RunContentView`（96 高） | **7 个界面 7 种行高、7 种纯色**，全是 `RectTransform` 手工摆 | 统一到 `G-ROW-01/02/03`（高 58 / 32 / 92 三档即可覆盖），代码侧改用九宫行底 |
| **对话选项** | `09_Run/DialogueOption_Normal/Hover.png`、`DialogueBox.png`（零引用） | `DialogueManager.cs:303` 克隆 `optionButtonPrefab` | 对话是**唯一完全走 prefab**的界面，但它的 prefab 素材本身缺（两张选项图零引用） | 直接把 `DialogueOption_*` 接进 `optionButtonPrefab`；走 `G-BTN-02` + `G-OVL-01` |
| **键帽** | `06_Overlays/Keycap_Square/Wide.png`、`10_System/KeybindChip.png`（零引用） | 轮盘快捷键角标（裸 TMP 50×22）、扎营整点按钮（34×30 纯色）、设置键位（`Button_Md_Normal`） | 同是"按键"，三处三种形态，其中两处裸文字 | 统一到 `G-CTRL-05`；`Keycap_Square` 正好 50×22 可用 |

---

## 5. 系统界面与"已有素材但未接线"清单

### 5.1 已存在的 Art/UI 资产与其真实使用状态

| 素材 | 文件 | 使用状态 | 应归属复用组 |
|---|---|---|---|
| `Panel_Main` / `Panel_Dialog` | `00_Panels/` | 仅 SettingPanel / SLPanel / MainMenuPanel | `G-PANEL-01` / `G-PANEL-03` |
| `Panel_Sub` / `Panel_TitleBar` / `Panel_ListBody` / `Panel_Main_Titled` | `00_Panels/` | **零引用** | `G-PANEL-03` / `G-PANEL-01` |
| `Button_Md_Normal` | `01_Buttons/` | SettingPane/SLPanel/MainMenuPanel | `G-BTN-01` |
| `Button_Md_Hover/Pressed/Disabled`、`Button_Lg_*`、`Button_Round_*` | `01_Buttons/` | **零引用（12 张）** | `G-BTN-01..05` |
| `Tab_Normal` / `Scrollbar_Track` / `Scrollbar_Thumb` / `Switch_*` / `Dropdown_*` | `02_Controls/` | 仅 Tab_Normal、Scrollbar_Track 各 1 处 | `G-TAB-01` / `G-CTRL-01..07` |
| `Bar_*`（11 张）/ `Slider_*`（5 张） | `03_Sliders_Bars/` | Slider_* 仅 SettingPanel；**Bar_* 全零引用** | `G-BAR-01..04` / `G-CTRL-02` |
| `Slot_*`、`IconFrame_*`、`QualityBar_*`、`QualitySeal`、`Checkbox_*` | `04_Slots/` | 仅 Slot_Selected、Checkbox_Unchecked 各少量 | `G-SLOT-01..05` / `G-CTRL-04` |
| `Card_*`、`PortraitFrame_Square`、`ClanCard_*`、`BuildingCard_*` | `05_Cards/` | 仅 Card_Normal/Selected/Disabled、PortraitFrame_Square | `G-CARD-01..04` |
| `Toast_*`、`Tooltip`、`Warn_*`、`Keycap_*`、`EventBar_*` | `06_Overlays/` | **全部 14 张零引用** | `G-OVL-01..05` / `G-CTRL-05` |
| `Ornament_CornerCloud/Divider/Meander/Seal` | `07_Ornaments/` | **全部 4 张零引用** | `G-ORN-01..04` |
| `DepthCell_*`、`EraBadge_*`、`EraNode_*`、`FamilyNode*`、`FamilySection_*`、`DispatchSlot_*`、`Market*`、`Meta_*`、`Tag_*`（32 张） | `08_Meta/` | 仅 `FamilyNodeDetail`、`DispatchSlot_Empty/Filled/Selected`、`FamilyNode_Max/Unlocked/Locked`、`FamilySection_Normal/Selected` 被两个 ScriptableObject 引用 | `G-SYS-01..04` |
| `SkillKey_Ready/Locked/Cooldown/CooldownRing`、`MinimapFrame`、`ConsumableBar`、`PartyBar_Active/Idle`、`SceneProgress`、`Campfire`、`CampOption_*`、`MapNode_*`、`LootRow_*`、`DialogueBox`、`DialogueOption_*`、`Button_Pause`（24 张） | `09_Run/` | **全部零引用** | `G-HUD-01..09` |
| `Loading_BarTrack/BarFill` | `10_System/` | **仅 LoadingScene 引用** | `G-BAR-03` |
| `Loading_Hourglass`、`Loading_TipBar`、`KeybindChip`、`KeybindConflict`、`SaveSlot_*`、`TitleSeal`、`SettingsRow`（13 张） | `10_System/` | 仅 `SettingsRow` 被 SettingPanel 用 | `G-SYS-01..04` / `G-BAR-03` |
| `DefaultSlices.png`（Primary_1..12 / Secondary_1..12 / Pointer） | `Resources/UI/RadialMenu/` | **轮盘唯一在用** | `G-HUD-06/07` |
| `Loading_01~03.png` | `Resources/UI/Loading/` | 加载背景 3 张轮播 | `G-PANEL-07` |
| `WorldLocations/*.png`（13 张） | `Resources/UI/WorldLocations/` | 大地图地点图标 | `G-HUD-08` |
| `Art/PNG/GameWorld_Map/Map_tianxia_zhongyuan.png` | `Art/PNG/` | 大地图底图 | — |

### 5.2 明确"未实现 / 不存在"的件（避免需求写空）

| 任务书提到的件 | 实际情况 | 证据 |
|---|---|---|
| `SkillKey_CooldownRing` 技能冷却进度环 | **代码不存在**；仅 `Art/UI/09_Run/SkillKey_CooldownRing.png` 文件存在且零引用；`RunTeamView.cs:313` 只有 `SkillsPlaceholder` 空块 | grep `SkillKey`/`CooldownRing` 在 `Scripts`、`*.prefab`、`*.asset` 全部 0 命中 |
| 深度 HUD 环形 / 条 | **不存在**；`RunDepthHud` 只有一行 TMP 文本（520×40）+ 一个按钮（180×48） | `RunDepthHud.cs:166-189` |
| 小地图圆形框 | **不存在**；`RunMapView` 视口是 1052×410 直角矩形 + `RectMask2D`；`MinimapFrame.png` 零引用 | `RunMapView.cs:36-38` |
| 加载沙漏 / 进度弧 | **不存在**；只有 `Image.Type.Filled + Horizontal` 的水平条（1192×30）+ 3 张全屏轮播图 | `LoadingController.cs:239-242`、`:762`；`fillAmount` 全工程仅此处使用 |
| 旋转扫描 / 描边生长动画件 | **不存在**；全工程无 `Image.Type.Filled` 以外的进度型动画，无 `Animator` 驱动的 UI 描边；唯一"旋转"是轮盘扇区 `localRotation` 与家族树连线 `localRotation` | `RadialMenuView.cs:77`、`FamilyTreePanel.cs:114-117` |
| UI 动画（入场/退场） | **存在但纯代码**：`UIPanelAnimator` 支持 None/Fade/Slide×4/Scale×2，参数 0.85/1.15/40px/0.05~0.2s | `PanelAnimType.cs:10-33`、`UIPanelAnimator.cs:51-76` |

### 5.3 界面计数与规模

- 盘点界面：**34 个**（局内 15：通用构件、深度 HUD、交互提示、队伍 HUD、快捷栏、地形 Toast、快捷轮盘、局内地图、地图节点、队伍详情、行囊菜单、扎营时辰、内容卡、结算、事件；大地图 3；局外 13；系统/加载/对话 3）。
- 运行时构建的视觉节点合计：`AddComponent<Image>()` 共 **82 处**、`GameplayUi.Rect` 共 **180+ 处**、`new GameObject(...)` 共 **60+ 处**，分布在 20 个 View 脚本中。
- 最重的三个脚本：`DispatchPanel.cs`（14 个 Image / 1241 行）、`OfficeTasksView.cs`（11 个 Image / 27 个 Rect / 751 行）、`WorldLocationsView.cs`（9 个 Image / 16 个 Rect / 891 行）。
- 唯一零运行时构建的界面：`SettingPanel`（143 组件全在 prefab）、`MainMenuPanel`、`SLPanel`、`DialogueManager`（全 Inspector 绑定）。

---

## 6. 备注：给 Lead 的三点提醒

1. **不要把"缺素材"当成主因**：`Art/UI` 里 138 张零引用资产已覆盖条/槽/卡/浮层/装饰/系统全套，真实缺口是"接线"（`GameplayUi.Button` 无 sprite 参数、血条纯色、遮罩纯色），需求文档应同时给出"接线清单"与"重绘清单"两张。
2. **轮盘与时辰环要分开写需求**：两者技术路线不同（切片旋转 vs 程序化 mesh），前者需要 12 张按片数变化的全画布切片（严格 512×512、纯白、中心孔透明、外沿按 r=240 收口），后者只需要一张可着色的环形底/描边（内 126 / 外 205）。
3. **所有复杂形状件必须"边框 + 极浅淡基础纹理"**：19 件里有 11 件靠 `Image.color` 着色（同一张图要变 7 种颜色）、7 件会 1~6× 缩放、5 件会旋转。带彩装饰会串色、6× 放大后折线可见、旋转后断头。这一条要写进美术执行标准的硬约束，而不是每件的备注里。
