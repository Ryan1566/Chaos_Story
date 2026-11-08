# A · UI 预制体素材清单（数据盘点）

> **文档定位**：本文件是数据底稿，不是正式美术需求文档。正式需求由 Lead 基于本文件合成（见同级 `01_美术执行标准与首轮需求.md`）。
> **盘点对象**：25 个 UI 预制体 —— `Assets/Prefabs/UI/SettingRows/` 5 个 + `Assets/Resources/UIPanels/` 19 个 + `UIPanels/SubUI_Prefab/RecordCell.prefab`。
> **数据来源**：① 全量控件树 dump `Assets/tmp_uidump_prefabs.txt`（421 行，UTF-8，含 RectTransform / Image.sprite / TMP 文本）；② `Assets/Art/UI` 200+ 张 png 的实测像素与 `spriteBorder`（System.Drawing 读像素 + 9 张 meta）；③ 各面板 `*Panel.cs` 代码常量；④ Unity 编辑器 MCP（端口 7891）核验 CanvasScaler 与组件。
> **盘点时间**：2026-11-08。

---

## 0. 硬约束（美术规范，先读）

<!-- 以下为硬约束，任何素材产出与复用判定都必须遵守 -->

1. **素材内不得出现任何可读文字**：汉字、字母、数字、标点、伪文字字形、纹章文字、货币符号、星级符号 —— 全部留空。只有纯几何形状允许。
2. 因此本清单中**所有 TMP / Text 节点一律判为「不需要新素材」**（文字由运行时文本渲染，素材只提供其底衬）。
3. **同类控件必须归并**：所有功能面板底板 = 1 张 `G-PANEL-01`；所有 Tab = 1 组 4 态 `G-TAB-01`；所有设置行/列表行 = 1 张 `G-ROW-01`。不得因面板不同而各画一张。
4. 复用组编号**只能使用 Lead 已定义的 `G-*` 编号**，不得自创前缀。清单中所有 `建议复用组` 均落在给定编号内；无对应项者显式标注「待 Lead 决策」，不自造组号。
5. **「是否需要新素材」三态定义**
   - **需要**：新规范下必须重绘，且该视觉元素仅本预制体使用。
   - **需要但共用**：新规范下必须重绘，但可归并到某个复用组由多预制体共用（**本清单绝大多数条目属于此态**）。
   - **不需要**：纯文字节点 / 允许纯色的全屏遮罩 / 运行时代码几何绘制。
   - 说明：库内即使已存在同名文件（如 `Panel_Main.png`），也视为**需按新规范重制**，旧图只作**尺寸与九宫格基准**；其「当前 Sprite」列记录的是**接入现状**，不代表美术合格。
6. **【归并冲突】** 标记 = 同一控件在多个预制体里用了不同 Sprite，或尺寸与精灵原始比例严重不符。全部冲突在 §3 汇总与 §5 列出，需 Lead 定案。

### 0.1 条数口径

- 逐预制体的「本预制体新增素材条数（去重后）」= **该预制体涉及的复用组个数**（组内多态不重复计）。
- §3 汇总表「条数」列 = **该复用组内的独立图片文件数**（如 Tab 四态 = 4）。
- 全局合计见 §3 表脚：**26 组 / 49 张**，另有 1 项（世界地图底图）无对应复用组、待 Lead 决策。

---

## 1. 全局基准

### 1.1 画布与缩放

| 项目 | 实测值 | 依据 |
|---|---|---|
| 参考分辨率 | 1920 × 1080 | MainScene 三个 CanvasScaler（GM_Canvas / Canvas / Canvas）均为 ScaleWithScreenSize、referenceResolution 1920×1080、matchWidthOrHeight = 0 |
| 运行时新建 UI | 同样 1920×1080 | `DamageFloaters.cs:69`、`RunItemWheelDriver.cs:60`、`VillageHud.cs:19`、`RunTeamHud.cs:164`、`RunDepthHud.cs:160`、`RunTerrainToast.cs:201`、`VillageSceneBuilder.cs:56` |
| 全屏面板根 | sizeDelta 0×0 + anchors (0,0)-(1,1) → 解析为 1920×1080 | 所有 19 个 UIPanels 预制体 |
| 其他场景 | EnterScene 2 个、LoadingScene 1 个（match=0.5）、TestScene 2 个，均 1920×1080 | 场景核验 |
| 设置行基准行高 | **96**（根 1400×96）；SettingPanel 内行实例实际高 **134** | 预制体根 + `SettingsRow.png` 原始高 |

> **注意（所有 5 个 SettingRow）**：预制体内部子节点 RectTransform 为**退化占位值**（Label 14×100、Button 32×76、Prev 8×56、Slider 42×50、ValueText −15×100 等），实际布局由 `VerticalLayoutGroup` + `LayoutElement` 在运行时计算（根被拉伸为 1398.4×96）。**美术请勿按占位像素出图**，以「复用组基准尺寸」为准。

### 1.2 九宫格基准表（用于新素材切图）

| 复用组 | 基准文件（现状） | 原始像素 | spriteBorder (L,B,R,T) |
|---|---|---|---|
| G-PANEL-01 | `00_Panels/Panel_Main.png` | 748×560 | 36,36,36,36 |
| G-PANEL-02 | `00_Panels/Panel_Main_Titled.png` | 748×560 | 36,86,36,36（上边含标题栏） |
| G-PANEL-03 | `00_Panels/Panel_Sub.png` | 496×292 | 32,32,32,32 |
| G-PANEL-04 | `00_Panels/Panel_ListBody.png` | 554×292 | 32,32,32,32 |
| G-PANEL-05 | `00_Panels/Panel_Dialog.png` | 700×408 | 36,86,36,36 |
| G-PANEL-06 | `00_Panels/Panel_TitleBar.png` | 748×50 | 170,0,170,0 |
| G-TAB-01 | `02_Controls/Tab_Normal.png` | 252×68 | 24,0,24,0 |
| G-BTN-02 | `01_Buttons/Button_Md_Normal.png` | 234×70 | 12,11,12,11 |
| G-ROW-01 | `10_System/SettingsRow.png` | 1634×134 | 32,0,32,0 |
| G-OVL-05 | `10_System/KeybindChip.png` | 182×48 | 建议 24,0,24,0 |

---

## 2. 逐预制体盘点

### 预制体：`Assets/Prefabs/UI/SettingRows/SettingRow_Button.prefab`

- **用途**：设置行模板 —— 左侧标签 + 右侧单个操作按钮（"更改绑定""恢复默认"这类行），被 SettingPanel 以 Prefab 实例方式复用。
- **根节点**：1400×96，锚点 (0.5,0.5) 双向居中，pivot (0.5,0.5)；置于 VerticalLayoutGroup 内被 LayoutElement 拉伸为 1398.4×96。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 行容器底板 | `SettingRow_Button/SettingRow_Button` | Image | 内置 UISprite（unity_builtin_extra） | 32×32 | 100×100 占位（运行时 1398.4×96） | 需要但共用 【归并冲突】 | G-ROW-01 |
| 行标签文字 | `.../Label` | Text | — | — | 14×100 占位 | 不需要（纯文本） | — |
| 行内操作按钮底 | `.../Button` | Image | 内置 UISprite（tint FFFFFFF2） | 32×32 | 32×76 占位 | 需要但共用 【归并冲突】 | G-BTN-02 |
| 行内按钮文字 | `.../Button/Label` | Text | — | — | 20×76 占位 | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：2**

---

### 预制体：`Assets/Prefabs/UI/SettingRows/SettingRow_Keybind.prefab`

- **用途**：设置行模板 —— 键位绑定行（一个键盘键 + 一个鼠标键 + 复位按钮），供设置页键位表使用。
- **根节点**：1400×96，锚点 (0.5,0.5)，pivot 居中；运行时拉伸至 1398.4×96。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 行容器底板 | `SettingRow_Keybind/SettingRow_Keybind` | Image | 内置 UISprite | 32×32 | 100×100 占位（运行时 1398.4×96） | 需要但共用 【归并冲突】 | G-ROW-01 |
| 行标签文字 | `.../Label` | Text | — | — | anchors 0–0.27 | 不需要（纯文本） | — |
| 键盘键帽底 | `.../KeyButton` | Image | 内置 UISprite | 32×32 | anchors 0.29–0.51 | 需要但共用 【归并冲突】 | G-OVL-05 |
| 键名文字（键盘） | `.../KeyButton/KeyText` | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 键位图标槽（键盘） | `.../KeyButton/KeyText/Icon` | Image | 无（sprite=null） | — | 0×0 | 不需要（字形由文本渲染，键帽图形由 G-OVL-05 承载） | — |
| 鼠标键帽底 | `.../MouseButton` | Image | 内置 UISprite | 32×32 | anchors 0.53–0.75 | 需要但共用 【归并冲突】 | G-OVL-05 |
| 键名文字（鼠标） | `.../MouseButton/MouseText` | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 键位图标槽（鼠标） | `.../MouseButton/MouseText/Icon` | Image | 无（sprite=null） | — | 0×0 | 不需要 | — |
| 复位按钮底 | `.../ResetButton` | Image | 内置 UISprite | 32×32 | anchors 0.81–0.93 | 需要但共用 【归并冲突】 | G-BTN-03 |
| 复位按钮文字 | `.../ResetButton/Label` | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：3**

---

### 预制体：`Assets/Prefabs/UI/SettingRows/SettingRow_Selector.prefab`

- **用途**：设置行模板 —— 多选一（左"<"、右">"翻页 + 中间当前值），用于枚举型设置（攻防触发方式等）。
- **根节点**：1400×96，锚点 (0.5,0.5)；运行时拉伸至 1398.4×96。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 行容器底板 | `SettingRow_Selector/SettingRow_Selector` | Image | 内置 UISprite | 32×32 | 100×100 占位（运行时 1398.4×96） | 需要但共用 【归并冲突】 | G-ROW-01 |
| 行标签文字 | `.../Label` | Text | — | — | anchors 0–0.42 | 不需要（纯文本） | — |
| 左翻页钮底 | `.../Prev` | Image（Button） | 内置 UISprite | 32×32 | 8×56 占位（anchors 0.44–0.52） | 需要但共用 【归并冲突】 | G-CTRL-07 |
| 左翻页几何指示 | `.../Prev/Label`（内容 "<"） | Text | — | — | 8×56 占位 | 需要但共用（文字形"<"应改几何箭头，见 §0-1） | G-CTRL-02 |
| 右翻页钮底 | `.../Next` | Image（Button） | 内置 UISprite | 32×32 | 8×56 占位（anchors 0.88–0.96） | 需要但共用 【归并冲突】 | G-CTRL-07 |
| 右翻页几何指示 | `.../Next/Label`（内容 ">"） | Text | — | — | 8×56 占位 | 需要但共用 | G-CTRL-02 |
| 当前值文字 | `.../ValueText` | Text | — | — | −15×100 占位 | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：3**

---

### 预制体：`Assets/Prefabs/UI/SettingRows/SettingRow_Slider.prefab`

- **用途**：设置行模板 —— 滑条（音量、鼠标灵敏度）。
- **根节点**：1400×96，锚点 (0.5,0.5)；运行时拉伸至 1398.4×96。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 行容器底板 | `SettingRow_Slider/SettingRow_Slider` | Image | 内置 UISprite | 32×32 | 100×100 占位（运行时 1398.4×96） | 需要但共用 【归并冲突】 | G-ROW-01 |
| 行标签文字 | `.../Label` | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 滑条轨道 | `.../Slider/Background` | Image | 内置 UISprite（tint C7C7C7FF） | 32×32 | 42×50 占位（anchors 0.44–0.86） | 需要但共用 【归并冲突】 | G-CTRL-06 |
| 滑条填充 | `.../Slider/Fill` | Image | 无（纯色 599ED9FF） | — | 锚点自适应 | 需要但共用 | G-CTRL-06 |
| 滑条手柄 | **节点缺失**（预制体内无 Handle） | Image | — | — | — | 需要但共用 【归并冲突】 | G-CTRL-06 |
| 当前值文字 | `.../ValueText` | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：2**

---

### 预制体：`Assets/Prefabs/UI/SettingRows/SettingRow_Toggle.prefab`

- **用途**：设置行模板 —— 开关/勾选（鼠标反转等布尔项）。
- **根节点**：1400×96，锚点 (0.5,0.5)；运行时拉伸至 1398.4×96。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 行容器底板 | `SettingRow_Toggle/SettingRow_Toggle` | Image | 内置 UISprite | 32×32 | 100×100 占位（运行时 1398.4×96） | 需要但共用 【归并冲突】 | G-ROW-01 |
| 行标签文字 | `.../Label` | Text | — | — | anchors 0–0.7 | 不需要（纯文本） | — |
| 勾选框背板 | `.../Toggle/Background` | Image | 内置 UISprite（tint FFFFFFF2） | 32×32 | 64×64 | 需要但共用 【归并冲突】 | G-CTRL-05 |
| 勾选标记 | `.../Toggle/Checkmark` | Image | 内置 UISprite | 40×40 | 44×44（sizeDelta −20×−20） | 需要但共用 | G-CTRL-05 |

**本预制体新增素材条数（去重后）：2**

---

### 预制体：`Assets/Resources/UIPanels/DispatchPanel.prefab`

- **用途**：派遣面板 —— 左侧候选族人卡片列表 / 中部角色预览 / 右侧编队槽位，用于派族人外出探索、采集。
- **根节点**：1920×1080（sizeDelta 0×0 + anchors (0,0)-(1,1) 拉伸），Image sprite=null、color `#14171CF5`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `DispatchPanel` | Image | 无（null，纯色 14171CF5） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 族人卡片底 | （运行时构建）`CardWidget/Root+Image` | Image | 无（GameplayUi 纯色 Image） | — | 锚点自适应，无固定像素 | 需要但共用 | G-CARD-01 |
| 卡片品质帧 | （运行时构建）`CardWidget/Image` 外框 | Image | 无 | — | 同上 | 需要但共用 | G-SLOT-03 |
| 角色头像框 | （运行时构建）`CardWidget` 头像位 | Image | 无 | — | 同上 | 需要但共用 | G-CARD-03 |
| 编队槽底 | （运行时构建）`SlotWidget/Image` | Image | 无 | — | 同上 | 需要但共用 | G-SLOT-01 |
| 槽位选中框 | （运行时构建）`SlotWidget/Outline` | Image | 无 | — | 同上 | 需要但共用 | G-SLOT-02 |
| 角色预览框 | （运行时构建）`PreviewFrame` | Image | 无 | — | 同上 | 需要但共用 | G-CARD-03 |
| 搜索输入框 | （运行时构建）`_search`（TMP_InputField） | Image | 无 | — | 同上 | 需要但共用 | G-CTRL-01 |
| 行内小按钮 | （运行时构建）`Button` | Image | 无（tint .18/.32/.48） | — | 同上 | 需要但共用 | G-BTN-03 |

**本预制体新增素材条数（去重后）：8**

---

### 预制体：`Assets/Resources/UIPanels/EraProgressPanel.prefab`

- **用途**：时代进度面板 —— 展示时代推进阶段与解锁内容。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#0F1217FA`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `EraProgressPanel` | Image | 无（0F1217FA） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 800×600（代码常量 PanelWidth/Height） | 需要但共用 | G-PANEL-01 |
| 时代节点卡 | （运行时构建）时代节点 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-CARD-02 |
| 关闭按钮 | （运行时构建）`rect.sizeDelta=300×54` | Image | 无 | — | 300×54 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：4**

---

### 预制体：`Assets/Resources/UIPanels/EventPanel.prefab`

- **用途**：剧情事件面板 —— 事件正文 + 选项列表。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#0F1217FA`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `EventPanel` | Image | 无（0F1217FA） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 800×600（代码常量） | 需要但共用 | G-PANEL-01 |
| 事件正文/对话框 | （运行时构建）`_optionArea` 上方正文区 | Image | 无 | — | offsetMin(120,72)，锚点自适应 | 需要但共用 | G-PANEL-05 |
| 选项行底 | （运行时构建）选项 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 选项行悬停/选中态 | （运行时构建）选项 | Image | 无 | — | 同上 | 需要但共用 | G-ROW-02 |

**本预制体新增素材条数（去重后）：5**

---

### 预制体：`Assets/Resources/UIPanels/FamilyTreePanel.prefab`

- **用途**：家族树面板 —— 族谱节点图（节点 + 连线 + 详情）。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#14171CF5`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `FamilyTreePanel` | Image | 无（14171CF5） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 族谱画布内区 | `TreeGraph`（运行时） | Image | 无 | — | 1300×800（`FamilyTreePanel.cs:63`） | 需要但共用 | G-PANEL-04 |
| 家族节点卡 | （运行时构建）节点 | Image | 无 | — | 172×144（代码常量） | 需要但共用 | G-CARD-02 |
| 节点头像框 | （运行时构建）节点头像 | Image | 无 | — | 随节点自适应 | 需要但共用 | G-CARD-03 |
| 节点连线 | （运行时绘制） | — | — | — | — | 不需要（代码绘制线条） | — |
| 返回/关闭按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：5**

---

### 预制体：`Assets/Resources/UIPanels/InventoryPanel.prefab`

- **用途**：库存面板 —— 仓库物品行列表 + 升级按钮。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#14171CF5`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `InventoryPanel` | Image | 无（14171CF5） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-PANEL-01 |
| 物品行底 | （运行时构建）行 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 物品格底 | （运行时构建）格子 | Image | 无 | — | 建议 94×94 基准 | 需要但共用 | G-SLOT-01 |
| 格子选中框 | （运行时构建）选中 | Image | 无 | — | 94×94 | 需要但共用 | G-SLOT-02 |
| 品质框 | （运行时构建）品质描边 | Image | 无 | — | 94×94 | 需要但共用 | G-SLOT-03 |
| 升级按钮 | （运行时构建）160×48 | Image | 无 | — | 160×48 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：7**

---

### 预制体：`Assets/Resources/UIPanels/MainMenuPanel.prefab`

- **用途**：主菜单首页 —— 标题 + 开始游戏 / 公告 / 设置 / 退出。
- **根节点**：1920×1080（拉伸），Image = `Assets/Art/UI/00_Panels/Panel_Main.png (748×560)` Sliced。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 面板底板 | `MainMenuPanel` | Image | `00_Panels/Panel_Main.png` | 748×560 | 1920×1080（九宫格拉伸，border 36） | 需要但共用 【归并冲突：748×560 拉满全屏】 | G-PANEL-01 |
| 主标题文字 | `.../Title`（"主菜单" fs78） | Text | — | — | 0×0 + ContentSizeFitter | 不需要（纯文本） | — |
| 主标题印/徽记底 | （无节点，需新增） | Image | 无 | — | 建议 360×360 装饰 | 需要但共用 | G-SYS-04 |
| 按钮列表容器 | `.../BtnList` | 空容器（VerticalLayoutGroup） | — | — | 205.44×389.97，anchors (1,0) | 不需要 | — |
| 菜单按钮底 ×4 | `.../BtnList/StartBtn、UpdateBtn、SettingBtn、QuitBtn` | Image | `01_Buttons/Button_Md_Normal.png` | 234×70 | 200×60 | 需要但共用 【归并冲突】 | G-BTN-02 |
| 菜单按钮文字 ×4 | `.../BtnList/*/Text (TMP)` | Text | — | — | 130×60（fs 30–32.5） | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：3**

---

### 预制体：`Assets/Resources/UIPanels/MarketPanel.prefab`

- **用途**：市场面板 —— 买卖交易列表与购买操作。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#050508B8`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `MarketPanel` | Image | 无（050508B8） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 800×600（代码常量） | 需要但共用 | G-PANEL-01 |
| 商品行底 | （运行时构建）行 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 商品行选中/悬停 | （运行时构建） | Image | 无 | — | 同上 | 需要但共用 | G-ROW-02 |
| 购买按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |
| 价格/数量文字 | （运行时构建） | Text | — | — | — | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：5**

---

### 预制体：`Assets/Resources/UIPanels/OfficeTasksPanel.prefab`

- **用途**：官府任务面板 —— 任务分类 + 任务行 + 领取奖励。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#FFFFFFFF`（**纯白不透，疑似占位遗留，见 §5**）。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `OfficeTasksPanel` | Image | 无（FFFFFFFF） | — | 1920×1080 | 不需要（允许纯色，但颜色需修正） | G-PANEL-07 |
| 面板标题 | **无 Title 节点**（建议补） | Text | — | — | — | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-PANEL-01 |
| 任务分类页签 | （运行时构建）分类 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-TAB-01 |
| 任务行底 | （运行时构建）行 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 领取按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：5**

---

### 预制体：`Assets/Resources/UIPanels/PausePanel.prefab`

- **用途**：局内暂停面板 —— 暂停卡片 + 继续 / 返回主菜单。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#FFFFFFFF`（纯白不透，疑似占位）。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `PausePanel` | Image | 无（FFFFFFFF） | — | 1920×1080 | 不需要（允许纯色，颜色需修正） | G-PANEL-07 |
| 暂停卡片 | （运行时构建）`PauseCard`（`PausePanel.cs:14`） | Image | 无 | — | 480×270 | 需要但共用 | G-PANEL-03 |
| 卡片标题文字 | （运行时构建） | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 继续按钮 | （运行时构建） | Image | 无 | — | 380×64 | 需要但共用 | G-BTN-02 |
| 返回按钮 | （运行时构建） | Image | 无 | — | 380×64 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：2**

---

### 预制体：`Assets/Resources/UIPanels/RetainerPanel.prefab`

- **用途**：族人管理面板 —— 族人列表行 + 头像 + 升级。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#14171CF5`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `RetainerPanel` | Image | 无（14171CF5） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-PANEL-01 |
| 族人行底 | （运行时构建）行 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 族人头像框 | （运行时构建） | Image | 无 | — | 建议 94×94 | 需要但共用 | G-CARD-03 |
| 升级按钮 | （运行时构建） | Image | 无 | — | 180×48 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：5**

---

### 预制体：`Assets/Resources/UIPanels/RunCampPanel.prefab`

- **用途**：局内扎营面板 —— 扎营选项（休息/做饭/整备）+ 轮盘。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#0F1217FA`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `RunCampPanel` | Image | 无（0F1217FA） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 800×600（代码常量） | 需要但共用 | G-PANEL-01 |
| 扎营选项卡底 | （运行时构建）选项卡 | Image | 无 | — | 554×466（`CampOption_*` 同尺寸） | 需要但共用 | G-CARD-01 |
| 扎营轮盘 | （运行时 `RadialWedgeGraphic`） | 自定义 Graphic | — | — | 440×440 | 不需要（代码绘制扇形） | — |
| 休整按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：4**

---

### 预制体：`Assets/Resources/UIPanels/RunContentPanel.prefab`

- **用途**：局内内容容器 —— 承载局内事件/宝箱/交互内容的主体卡片。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#000000B8`（半透明黑）。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `RunContentPanel` | Image | 无（000000B8） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 内容卡片 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-PANEL-03 |
| 内容正文框 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-PANEL-05 |
| 操作按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：3**

---

### 预制体：`Assets/Resources/UIPanels/RunMenuPanel.prefab`

- **用途**：局内主菜单 —— 功能集合（背包/技能/队伍等页签 + 槽位 + 目录格）。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#000000A6`（半透明黑）。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `RunMenuPanel` | Image | 无（000000A6） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 主面板框（带标题栏） | （运行时构建）`Box` | Image | 无 | — | 1100×700（BoxWidth/BoxHeight） | 需要但共用 | G-PANEL-02 |
| 功能页签 | （运行时构建）Tab | Image | 无 | — | 210×42 | 需要但共用 | G-TAB-01 |
| 物品/技能槽底 | （运行时构建）槽 | Image | 无 | — | 156×60 | 需要但共用 | G-SLOT-01 |
| 槽位选中框 | （运行时构建） | Image | 无 | — | 156×60 | 需要但共用 | G-SLOT-02 |
| 目录格 | （运行时构建） | Image | 无 | — | 82×56 | 需要但共用 | G-SLOT-01 |
| 关闭/确认按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：5**

---

### 预制体：`Assets/Resources/UIPanels/RunSettlementPanel.prefab`

- **用途**：局内结算面板 —— 本局收益/损失结算行 + 翻页。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#0F1217FA`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `RunSettlementPanel` | Image | 无（0F1217FA） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 800×600（代码常量） | 需要但共用 | G-PANEL-01 |
| 结算行底 | （运行时构建）行 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 页码条 | （运行时构建）`pageRect` | Text | — | — | 680×28 | 不需要（纯文本） | — |
| 确认按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：4**

---

### 预制体：`Assets/Resources/UIPanels/SettingPanel.prefab`

- **用途**：设置面板 —— 左栏 4 页签（玩法/键位/画面/音频）+ 右侧 5 个滚动页 + 键位绑定子页 + 底部 3 按钮 + 确认弹窗。**全工程最复杂的 UI 预制体（29 个行实例）。**
- **根节点**：1920×1080（拉伸），Image = `00_Panels/Panel_Main.png (748×560)` Sliced。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 面板底板 | `SettingPanel` | Image | `00_Panels/Panel_Main.png` | 748×560 | 1920×1080（九宫格拉伸，border 36） | 需要但共用 【归并冲突：748×560 拉满全屏】 | G-PANEL-01 |
| 面板标题文字 | `.../Title`（"设置" fs64） | Text | — | — | 自动文本 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 748×50 | 需要但共用 | G-PANEL-06 |
| 左栏页签底 ×4 | `MainList/Btn_Gameplay、Btn_Keybind、Btn_Graphics、Btn_Audio` | Image | `01_Buttons/Button_Md_Normal.png` | 234×70 | 布局自适应（列宽 ~205） | 需要但共用 【归并冲突】 | G-TAB-01 |
| 左栏页签选中框 ×4 | `.../Btn_*/Selected` | Image | `04_Slots/Slot_Selected.png` | 94×94 | 12×12（四角定位） | 需要但共用 【归并冲突】 | G-TAB-01 |
| 左栏页签文字 ×4 | `.../Btn_*/Label` | Text | — | — | 布局自适应 | 不需要（纯文本） | — |
| 内容区内区 | `ContentArea` | 空容器 | — | — | anchors (0.22,0.16)-(0.97,0.7) | 需要但共用 | G-PANEL-04 |
| 设置行容器底 ×29 | `Page_*/Viewport/Content/Row_*` | Image | `10_System/SettingsRow.png` | 1634×134 | 宽度自适应，行高 134 | 需要但共用 【归并冲突】 | G-ROW-01 |
| 行内翻页钮 ×4 | 行实例 `Prev、Next` | Image | `Button_Md_Normal.png` | 234×70 | 8×56（占位） | 需要但共用 【归并冲突】 | G-CTRL-07 |
| 行内复位/操作按钮 | 行实例 `ResetButton、Button` | Image | `Button_Md_Normal.png` | 234×70 | 32×76（占位） | 需要但共用 【归并冲突】 | G-BTN-03 |
| 滑条轨道 ×2 | 行实例 `Slider/Background` | Image | `03_Sliders_Bars/Slider_Track.png` | 468×18 | 42×50（占位） | 需要但共用 | G-CTRL-06 |
| 滑条填充 ×2 | 行实例 `Slider/Fill` | Image | `03_Sliders_Bars/Slider_Fill.png` | 468×18 | 锚点自适应 | 需要但共用 | G-CTRL-06 |
| 滑条手柄 ×2 | **节点缺失** | Image | — | — | — | 需要但共用 【归并冲突】 | G-CTRL-06 |
| 勾选框背板 ×2 | 行实例 `Toggle/Background` | Image | `04_Slots/Checkbox_Unchecked.png` | 44×44 | 64×64 | 需要但共用 【归并冲突】 | G-CTRL-05 |
| 勾选标记 ×2 | 行实例 `Toggle/Checkmark` | Image | 内置 UISprite | 40×40 | 44×44 | 需要但共用 | G-CTRL-05 |
| 顶部页签底 ×5 | `Header/BackButton、Tab_Keyboard、Tab_Gamepad、ModelTab_PS、ModelTab_Xbox` | Image | `02_Controls/Tab_Normal.png` | 252×68 | 布局自适应 | 需要但共用 【归并冲突】 | G-TAB-01 |
| 顶部页签选中框 ×5 | `.../Selected` | Image | `Slot_Selected.png` | 94×94 | 12×12 | 需要但共用 【归并冲突】 | G-TAB-01 |
| 键位提示文字 | `Header/ReservedKeyHint` | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 滚动条轨道 | `Scrollbar_Vertical` | Image | `02_Controls/Scrollbar_Track.png` | 16×350 | 16×内容高 | 需要但共用 | G-CTRL-03 |
| 滚动条滑块 | `Scrollbar_Vertical/Handle` | Image | `03_Sliders_Bars/Slider_Handle_Normal.png` | 26×30 | 26×~60 | 需要但共用 【归并冲突：应换 Scrollbar_Thumb 16×92】 | G-CTRL-03 |
| 底部按钮底 ×3 | `BottomBar/ApplyBtn、ResetBtn、ReturnBtn` | Image | `Button_Md_Normal.png` | 234×70 | 146 / 222 / 146 × 141.76 | 需要但共用 【归并冲突】 | G-BTN-02 |
| 底部按钮文字 ×3 | `.../Label` | Text | — | — | — | 不需要（纯文本） | — |
| 确认弹窗遮罩 | `ConfirmGroup` | Image | 无（纯色 0000008C） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 确认弹窗框 | `ConfirmGroup/Box` | Image | `00_Panels/Panel_Dialog.png` | 700×408 | 900×400 【归并冲突：比例失真】 | 需要但共用 | G-PANEL-05 |
| 确认弹窗正文 | `ConfirmGroup/Box/ConfirmText` | Text | — | — | — | 不需要（纯文本） | — |
| 确认弹窗按钮 ×3 | `ConfirmGroup/Box/ConfirmApplyBtn、ConfirmDiscardBtn、ConfirmCancelBtn` | Image | `Button_Md_Normal.png` | 234×70 | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：12**

---

### 预制体：`Assets/Resources/UIPanels/SLPanel.prefab`

- **用途**：存档/读档面板 —— 存档格横向列表（运行时 Instantiate `RecordCell`）+ 删除/返回 + 确认弹窗。
- **根节点**：1920×1080（拉伸），Image = `00_Panels/Panel_Main.png (748×560)` Sliced。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 面板底板 | `SLPanel` | Image | `00_Panels/Panel_Main.png` | 748×560 | 1920×1080（九宫格拉伸，border 36） | 需要但共用 【归并冲突】 | G-PANEL-01 |
| 面板标题文字 | `.../Title` | Text | — | — | 自动文本 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 748×50 | 需要但共用 | G-PANEL-06 |
| 存档列表内区 | `RecordsList` | 空容器（HorizontalLayoutGroup + ContentSizeFitter） | — | — | 宽度 0–1，高 200，当前为空 | 需要但共用 | G-PANEL-04 |
| 存档格底 | （运行时 `RecordCell`） | Image | `05_Cards/Card_Normal.png` | 408×292 | 150×200 【归并冲突】 | 需要但共用 | G-SYS-01 |
| 存档格选中框 | （运行时 `RecordCell/Selected`） | Image | `05_Cards/Card_Selected.png` | 408×292 | 150×200 | 需要但共用 | G-SYS-01 |
| 底部按钮底 ×2 | `BottomBar/DeleteBtn、ReturnBtn` | Image | `Button_Md_Normal.png` | 234×70 | 146×130.24 | 需要但共用 【归并冲突】 | G-BTN-02 |
| 底部按钮文字 ×2 | `.../Label` | Text | — | — | — | 不需要（纯文本） | — |
| 确认弹窗遮罩 | `ConfirmGroup` | Image | 无（0000008C） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 确认弹窗框 | `ConfirmGroup/Box` | Image | `00_Panels/Panel_Dialog.png` | 700×408 | 900×400 【归并冲突：比例失真】 | 需要但共用 | G-PANEL-05 |
| 确认弹窗正文 | `ConfirmGroup/Box/ConfirmText` | Text | — | — | — | 不需要（纯文本） | — |
| 确认弹窗按钮 ×2 | `ConfirmGroup/Box/ConfirmOkBtn、ConfirmCancelBtn` | Image | `Button_Md_Normal.png` | 234×70 | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：6**

---

### 预制体：`Assets/Resources/UIPanels/VillageOfficePanel.prefab`

- **用途**：村庄官府面板 —— 官府办公卡片（建筑/任务入口按钮组）。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#FFFFFFFF`（纯白不透，疑似占位）。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `VillageOfficePanel` | Image | 无（FFFFFFFF） | — | 1920×1080 | 不需要（允许纯色，颜色需修正） | G-PANEL-07 |
| 办公卡片 | （运行时构建）卡片 | Image | 无 | — | 880×560 | 需要但共用 | G-PANEL-03 |
| 卡片标题文字 | （运行时构建） | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 主操作按钮 ×N | （运行时构建） | Image | 无 | — | 260×56 | 需要但共用 | G-BTN-02 |
| 次操作按钮 ×N | （运行时构建） | Image | 无 | — | 240×54 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：2**

---

### 预制体：`Assets/Resources/UIPanels/VillagePanel.prefab`

- **用途**：村庄总览面板 —— 建筑/设施行列表与建造升级。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#14171CF5`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `VillagePanel` | Image | 无（14171CF5） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 面板标题文字 | `.../Title` | Text | — | — | 1872×40 | 不需要（纯文本） | — |
| 标题文字底衬 | （无节点，需新增） | Image | 无 | — | 1872×40 | 需要但共用 | G-PANEL-06 |
| 主面板框 | （运行时构建）面板框 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-PANEL-01 |
| 建筑行底 | （运行时构建）行 | Image | 无 | — | 锚点自适应 | 需要但共用 | G-ROW-01 |
| 建造/升级按钮 | （运行时构建） | Image | 无 | — | 锚点自适应 | 需要但共用 | G-BTN-02 |

**本预制体新增素材条数（去重后）：4**

---

### 预制体：`Assets/Resources/UIPanels/WorldMapPanel.prefab`

- **用途**：世界地图面板 —— 天下中原大地图 + 势力遮罩层 + 地点图标层 + 底栏进入局内。
- **根节点**：1920×1080（拉伸），Image sprite=null、color `#14171CFF`。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 全屏底/遮罩 | `WorldMapPanel` | Image | 无（14171CFF） | — | 1920×1080 | 不需要（允许纯色） | G-PANEL-07 |
| 世界地图底图 | `.../BaseMap` | Image | `Assets/Art/PNG/GameWorld_Map/Map_tianxia_zhongyuan.png` | 4096×3138 | 1409.71×1080（Simple，等比） | **需要**（专用大图）【无对应复用组，见 §5】 | —（待 Lead 决策） |
| 势力遮罩层 | `.../FactionMask` | RawImage | 无（tex=null，WorldFactionMask） | — | 1409.71×1080 | 不需要（运行时渲染蒙版） | — |
| 地点图标层 | `.../IconLayer` | 空容器 | — | — | 1409.71×1080 | 不需要（预制体内为空） | — |
| 时期信息条文字 | `.../InfoBar` | Text | — | — | anchors 越界 −40，宽 1880×70 | 不需要（纯文本） | — |
| 信息条底衬 | （无节点，需新增） | Image | 无 | — | 1880×70 | 需要但共用 | G-PANEL-06 |
| 底栏按钮底 ×4 | `.../BottomBar/ReturnBtn、EnterRunBtn、StageDebugPrev、StageDebugNext` | Image | 内置 UISprite（tint 2E3642FF） | 32×32 | 高 84，宽自适应（anchors 越界 −40） | 需要但共用 【归并冲突】 | G-BTN-02 |
| 底栏按钮文字 ×4 | `.../Label`（fs26） | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：2（另有 1 项无组归属的专用大图待决策）**

---

### 预制体：`Assets/Resources/UIPanels/SubUI_Prefab/RecordCell.prefab`

- **用途**：存档格子 —— 存档缩略图 + 存档名 + 序号 + 时间，由 SLPanel 运行时实例化。
- **根节点**：150×200，Image = `05_Cards/Card_Normal.png (408×292)` Sliced，挂 Button + RecordCell + LayoutElement。CanvasScaler 参考 1920×1080（match=0）。

| 视觉元素 | 节点路径 | 类型 | 当前 Sprite | 当前像素 | 显示尺寸(sizeDelta/锚点换算) | 是否需要新素材 | 建议复用组 |
|---|---|---|---|---|---|---|---|
| 存档格底 | `RecordCell` | Image | `05_Cards/Card_Normal.png` | 408×292 | 150×200 【归并冲突：408×292 压到 150×200】 | 需要但共用 | G-SYS-01 |
| 存档格选中框 | `.../Selected` | Image | `05_Cards/Card_Selected.png` | 408×292 | 150×200（铺满） | 需要但共用 | G-SYS-01 |
| 存档缩略图槽 | （无节点，需新增） | Image | 无 | — | 建议 130×90 | 需要但共用 | G-SYS-01 |
| 存档名文字 | `.../Text (TMP)`（"存档"） | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 存档序号文字 | `.../RecordNum_text`（"1"） | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |
| 存档时间文字 | `.../RecordTime_text`（"2026/10/04"） | Text | — | — | 锚点自适应 | 不需要（纯文本） | — |

**本预制体新增素材条数（去重后）：1**

---

## 3. 全量素材需求汇总

> 「条数」= 该复用组内的独立图片文件数；「被哪些预制体引用」为 25 个预制体中的引用者。

| 复用组 | 素材名 | 像素尺寸 | 九宫格边距建议 (L,B,R,T) | 被哪些预制体引用 | 条数 |
|---|---|---|---|---|---|
| G-PANEL-01 | 面板底板_Main（主功能面板统一底板，实体框） | 748×560 | 36,36,36,36 | MainMenu、Setting、SL、EraProgress、Event、Inventory、Market、OfficeTasks、Retainer、RunCamp、RunSettlement、Village | 1 |
| G-PANEL-02 | 面板底板_Titled（标题栏与主体一体） | 1100×700 | 36,86,36,36 | RunMenu | 1 |
| G-PANEL-03 | 子面板底板_Sub（内嵌卡片/办公卡） | 880×560 | 32,32,32,32 | Pause（480×270 同图缩放）、RunContent、VillageOffice | 1 |
| G-PANEL-04 | 列表载体内区（内凹、无边框） | 1600×900 | 24,24,24,24 | Setting、SL、FamilyTree | 1 |
| G-PANEL-05 | 对话/事件框 + 确认框（含标题带） | 748×420 | 36,86,36,36 | Setting、SL、Event、RunContent | 1 |
| G-PANEL-06 | 标题栏横条（面板标题底衬 / 信息条底衬） | 748×50 | 170,0,170,0 | Dispatch、EraProgress、Event、FamilyTree、Inventory、Market、OfficeTasks、Retainer、RunCamp、RunSettlement、Setting、SL、Village、WorldMap | 1 |
| G-TAB-01 | Tab/主选择按钮 四态（Normal/Selected/Hover/Disabled） | 252×68 | 24,0,24,0 | Setting、OfficeTasks、RunMenu | 4 |
| G-BTN-02 | 中按钮 四态（Normal/Hover/Pressed/Disabled） | 234×70 | 12,11,12,11 | SettingRow_Button、MainMenu、Setting、SL、EraProgress、FamilyTree、Inventory、Market、OfficeTasks、Pause、Retainer、RunCamp、RunContent、RunMenu、RunSettlement、VillageOffice、Village、WorldMap | 4 |
| G-BTN-03 | 小按钮 三态（含图标按钮位） | 120×44 | 16,0,16,0 | SettingRow_Keybind、Dispatch、Setting | 3 |
| G-CTRL-01 | 输入框/下拉框体（InputField 底，含聚焦态） | 322×48 | 16,0,16,0 | Dispatch | 1 |
| G-CTRL-02 | 翻页/下拉几何指示（左右/上下箭头，纯几何） | 28×28 | 0,0,0,0（无九宫格） | SettingRow_Selector、Setting | 2 |
| G-CTRL-03 | 滚动条轨道 + 滑块 | 轨道 16×350 / 滑块 16×92 | 0,8,0,8 | Setting | 2 |
| G-CTRL-05 | 勾选框 两态（未选/已选） | 44×44 | 8,8,8,8 | SettingRow_Toggle、Setting | 2 |
| G-CTRL-06 | 滑条：轨道 + 填充 + 手柄三态 | 轨道/填充 468×18；手柄 26×30 | 轨道 8,0,8,0；手柄 0 | SettingRow_Slider、Setting | 5 |
| G-CTRL-07 | 数字选择器左右钮（Prev/Next 三态） | 44×44 | 8,8,8,8 | SettingRow_Selector、Setting | 3 |
| G-ROW-01 | 通用行容器（设置行/列表行/市场行/战利品行统一） | 1634×134 | 32,0,32,0 | SettingRow×5、Setting、Event、Inventory、Market、OfficeTasks、Retainer、RunSettlement、Village | 1 |
| G-ROW-02 | 行选中/悬停态（叠加层，透明底） | 1634×134 | 32,0,32,0 | Event、Market | 1 |
| G-SLOT-01 | 格子底 | 94×94 | 12,12,12,12 | Dispatch、Inventory、RunMenu | 1 |
| G-SLOT-02 | 选中框（叠加，空心） | 94×94 | 12,12,12,12 | Dispatch、Inventory、RunMenu | 1 |
| G-SLOT-03 | 品质框 五色（同形，五色靠 tint 或五张） | 94×94 | 12,12,12,12 | Dispatch、Inventory | 5 |
| G-CARD-01 | 通用卡片底 | 408×292 | 24,24,24,24 | Dispatch、RunCamp | 1 |
| G-CARD-02 | 节点卡（族谱/时代节点） | 172×144 | 16,16,16,16 | EraProgress、FamilyTree | 1 |
| G-CARD-03 | 头像框（圆形 + 方形） | 圆 94×94 / 方 188×234 | 圆 0（弧形）/ 方 16,16,16,16 | Dispatch、FamilyTree、Retainer | 2 |
| G-SYS-01 | 存档槽 两态（Normal/Selected，含缩略图位） | 554×934 | 24,24,24,24 | SL、RecordCell | 2 |
| G-SYS-04 | 标题印/徽记（主菜单标题装饰） | 748×748 | 0（无九宫格） | MainMenu | 1 |
| G-OVL-05 | 键帽胶囊（键盘/鼠标键帽底） | 182×48 | 24,0,24,0 | SettingRow_Keybind | 1 |
| （无对应组） | 世界地图底图_天下中原 | 4096×3138 | 0（Simple，不使用九宫格） | WorldMap | 1 |

**合计：26 个 G- 复用组，49 张；另有 1 张世界地图底图无复用组归属（待 Lead 决策），若计入则 27 项 / 50 张。**

### 3.1 归并冲突与统一建议（Lead 需定案）

1. **行底板**：5 个 SettingRow 用内置 UISprite（32×32）→ 29 个行实例用 `SettingsRow.png`（1634×134）。**统一 G-ROW-01**，行高 134。
2. **Tab/页签**：左栏 4 个用 `Button_Md_Normal`（234×70）、顶部 5 个用 `Tab_Normal`（252×68）、选中态又叠 `Slot_Selected`（94×94）三套混用 → **统一 G-TAB-01 四态**，取消 Slot_Selected 复用。
3. **按钮**：设置行内用内置 UISprite、实例内用 `Button_Md_*`、WorldMap 底栏用内置 UISprite（tint 2E3642FF）→ **统一 G-BTN-02**；小尺寸场景走 G-BTN-03。
4. **勾选**：`SettingRow_Toggle` 内置 UISprite + 40×40 勾、实例用 `Checkbox_Unchecked` 44×44、库内 `Switch_On/Off` 88×42 零引用 → **统一 G-CTRL-05**；若产品要"开关"外观则改 G-CTRL-04，二选一。
5. **滑块**：预制体与实例**都没有手柄**，库内 `Slider_Handle_*`（26×30）未接入 → **统一 G-CTRL-06 含手柄三态**。
6. **滚动条**：轨道正确（`Scrollbar_Track` 16×350），滑块误用 `Slider_Handle_Normal`（26×30）→ **统一 G-CTRL-03**，滑块改用 16×92 形态。
7. **存档槽**：`RecordCell` 用 `Card_Normal`（408×292）压到 150×200 当存档格，库内 `SaveSlot_Normal/Selected`（554×934）零引用 → **统一 G-SYS-01**，并明确存档格最终尺寸（150×200 还是 554×934 两套）。
8. **对话框**：Setting/SL 的 `ConfirmGroup/Box` 设 900×400 却用 `Panel_Dialog` 700×408（九宫格 border 36,86）→ **统一 G-PANEL-05**，按 748×420 出图后九宫格拉伸。
9. **主面板**：`Panel_Main`（748×560）被拉满 1920×1080，而 `Panel_Main_Titled`（带标题栏）零引用 → 需定 **G-PANEL-01 与 G-PANEL-02 分工**（全屏面板用九宫格大底板，还是保留 748×560 居中框）。
10. **标题悬空**：9 个面板的 Title 直接压在纯色根上，无任何底衬 → **需补 G-PANEL-06**（库内 `Panel_TitleBar` 748×50 零引用）。
11. **零引用闲置素材**（建议纳入首轮需求复用评估）：`Panel_Main_Titled`、`Panel_TitleBar`、`Panel_Sub`、`Panel_ListBody`、`Dropdown_*`、`Switch_*`、`Scrollbar_Thumb`、`Slider_Handle_*`、`SaveSlot_*`、`TitleSeal`、`DialogueBox`、`Keycap_*`、全部 `Bar_*`、`SkillKey_*`、`MapNode_*`、`EventBar_*`、`Toast_*`、`Tooltip`、`Warn_*`、`Ornament_*`。
12. **无对应复用组**：WorldMapPanel 的 `BaseMap`（4096×3138 世界地图底图）不属于任何 G- 组 —— 不得自创前缀，**需 Lead 决策**是否新增地图类组。

---

## 4. 预制体覆盖率自检

| 预制体 | 已盘控件数 | 需新素材数 | 状态 |
|---|---|---|---|
| `Prefabs/UI/SettingRows/SettingRow_Button` | 4 | 2 | ok |
| `Prefabs/UI/SettingRows/SettingRow_Keybind` | 10 | 3 | ok |
| `Prefabs/UI/SettingRows/SettingRow_Selector` | 7 | 3 | ok |
| `Prefabs/UI/SettingRows/SettingRow_Slider` | 6 | 2 | ok |
| `Prefabs/UI/SettingRows/SettingRow_Toggle` | 4 | 2 | ok |
| `Resources/UIPanels/DispatchPanel` | 11 | 8 | 待确认 |
| `Resources/UIPanels/EraProgressPanel` | 6 | 4 | 待确认 |
| `Resources/UIPanels/EventPanel` | 7 | 5 | 待确认 |
| `Resources/UIPanels/FamilyTreePanel` | 8 | 5 | 待确认 |
| `Resources/UIPanels/InventoryPanel` | 9 | 7 | 待确认 |
| `Resources/UIPanels/MainMenuPanel` | 6 | 3 | ok |
| `Resources/UIPanels/MarketPanel` | 8 | 5 | 待确认 |
| `Resources/UIPanels/OfficeTasksPanel` | 7 | 5 | 待确认 |
| `Resources/UIPanels/PausePanel` | 5 | 2 | 待确认 |
| `Resources/UIPanels/RetainerPanel` | 7 | 5 | 待确认 |
| `Resources/UIPanels/RunCampPanel` | 7 | 4 | 待确认 |
| `Resources/UIPanels/RunContentPanel` | 4 | 3 | 待确认 |
| `Resources/UIPanels/RunMenuPanel` | 7 | 5 | 待确认 |
| `Resources/UIPanels/RunSettlementPanel` | 7 | 4 | 待确认 |
| `Resources/UIPanels/SettingPanel` | 26 | 12 | ok |
| `Resources/UIPanels/SLPanel` | 12 | 6 | ok |
| `Resources/UIPanels/VillageOfficePanel` | 5 | 2 | 待确认 |
| `Resources/UIPanels/VillagePanel` | 6 | 4 | 待确认 |
| `Resources/UIPanels/WorldMapPanel` | 8 | 2 | ok |
| `Resources/UIPanels/SubUI_Prefab/RecordCell` | 6 | 1 | ok |

**覆盖：25 / 25 个预制体；ok 11 个、待确认 14 个；需新素材 26 组（49 张）+ 1 项待决策。**

---

## 5. 待确认清单（14 项）

**统一原因**：这些预制体在项目里是**"空壳"**——只有全屏根 Image（+ 可选 Title 文本），**所有内容节点都由 `<Xxx>Panel.cs` / `<Xxx>View` 在运行时用代码创建**；且布局大量使用 anchors + offset 的自适应写法，而非固定像素。因此：① 预制体层面已 100% 盘清（无遗漏节点）；② 运行时控件的**精确像素无法从预制体取得**，需按面板脚本做二次代码级盘点。表中列出的运行时控件尺寸为**代码常量**（有则给，无则写"锚点自适应"）。

| # | 预制体 | 待确认内容 | 已确认的代码尺寸 |
|---|---|---|---|
| 1 | DispatchPanel | 卡片/槽位/预览框的具体像素与内部结构 | 无固定值（三栏锚点布局）；`CardWidget`/`SlotWidget`/`PreviewFrame`/`_search` 结构已确认 |
| 2 | EraProgressPanel | 时代节点排布、面板框内边距 | 面板框 800×600；关闭钮 300×54 |
| 3 | EventPanel | 正文区高度、选项行高 | 面板框 800×600；选项区 offsetMin (120,72) |
| 4 | FamilyTreePanel | 节点连线与滚动区域最终尺寸 | 画布 1300×800；节点 172×144 |
| 5 | InventoryPanel | 物品行高、格子列数 | 升级钮 160×48 |
| 6 | MarketPanel | 行高、列宽 | 面板框 800×600 |
| 7 | OfficeTasksPanel | 分类页签数量与布局 | 无固定值 |
| 8 | PausePanel | — | 卡片 480×270；按钮 380×64 |
| 9 | RetainerPanel | 行高、头像尺寸 | 升级钮 180×48 |
| 10 | RunCampPanel | 选项卡排布与轮盘关系 | 面板框 800×600；轮盘 440×440 |
| 11 | RunContentPanel | 内容卡片尺寸 | 无固定值 |
| 12 | RunMenuPanel | 页签数量、槽位网格 | Box 1100×700；Tab 210×42；槽 156×60；目录格 82×56 |
| 13 | RunSettlementPanel | 行数与翻页 | 面板框 800×600；页码 680×28 |
| 14 | VillageOfficePanel / VillagePanel | 按钮组与行结构 | 办公卡 880×560；按钮 260×56 / 240×54 |

**另有 4 项数据异常需 Lead 注意**：

1. `OfficeTasksPanel`、`PausePanel`、`VillageOfficePanel` 根 Image color = `#FFFFFFFF`（**纯白不透**），会把整个场景盖成白色 —— 疑为占位遗留，实机应表现为半透明遮罩或全屏面板底。
2. `WorldMapPanel` 的 `InfoBar` 与 `BottomBar` anchors 越界（offset −40），宽度 1880 > 1920 画布内宽，存在横向出屏风险。
3. 13 个空壳面板的 `Title` 均为 1872×40 的裸文字，无底衬、无对齐锚点约束，接入美术前需先统一标题布局。
4. `RunContentPanel`（000000B8）与 `RunMenuPanel`（000000A6）根为半透明黑，与其它面板的 `0F1217FA` / `14171CF5` 不一致，建议统一到 G-PANEL-07 遮罩规范（如 `#000000A6`）。
