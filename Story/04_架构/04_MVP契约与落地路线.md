# 04 MVP 契约与落地路线

> 状态：草稿（待 task-5 落地方案评审闸门通过后转为定稿）
> Owner：架构师
> 最后更新：2026-10-05
> 关联：`Story/04_架构/01_架构现状核查.md`、`Story/04_架构/02_目标结构蓝图.md`、`Story/04_架构/03_路径与命名规范.md`、`Story/README.md`（第 4 章协作流程与硬性闸门）

本文件回答四个问题：**Presenter 长什么样、Model 谁创建谁释放、View 怎么被驱动、EventCenter 放哪一层**；并给出**按风险从低到高**的落地顺序。

---

## 1. 契约总览

### 1.1 分层与依赖方向

```
┌──────────────────────────────────────────────────────────────┐
│ View 层   MVP/View/<System>/          （继承 BasePanel，MonoBehaviour）
│   · 只显示、只采集输入；把玩家意图以强类型事件抛给 Presenter
│   · 实现 I<System>View（接口定义在 Presenter 目录）
└───────────────▲──────────────────────────┬───────────────────┘
                │ 方法调用（刷新显示）        │ 事件/回调（玩家意图）
┌───────────────┴──────────────────────────▼───────────────────┐
│ Presenter 层  MVP/Presenter/<System>/   （纯 C# 类，不继承 MonoBehaviour）
│   · 唯一职责：双向翻译。持有 I<System>View 与 <System>Model 的引用
│   · 不碰 Transform / Button / TextMeshPro / Resources.Load
└───────────────▲──────────────────────────┬───────────────────┘
                │ 强类型 C# event             │ 公开方法调用（业务动作）
┌───────────────┴──────────────────────────▼───────────────────┐
│ Model 层  MVP/Model/<System>/           （实现 IModel，有生命周期）
│   · 状态 + 业务规则；变化时抛强类型 event
└───────────────▲──────────────────────────┬───────────────────┘
                │                            │
┌───────────────┴──────────────────────────▼───────────────────┐
│ 框架层  GameBase/  （ConfigRepository / Recorder / EventManager / UIManager…）
│   · 与平台相关的 IO、资源、事件总线、UI 栈；全部 SingletonBase
└──────────────────────────────────────────────────────────────┘
```

**允许的依赖**（箭头即允许）：
- View → Presenter（只通过 `PresenterBase` 的 `Attach`/`Dispose` 与 `I<System>View` 实现）
- Presenter → Model（构造注入）、Presenter → 框架层服务（构造注入或 `Instance`）
- Model → 框架层服务（`ConfigRepository` / `Recorder` / `EventManager`）
- Model → View：**禁止**；Model → Presenter：**禁止**（只抛 event，谁订阅是别人的事）

### 1.2 与现状的差距（一句话）

现状 = **View → Manager 单例**（6 个文件、43 处 Manager 单例调用，`01_架构现状核查.md` 第 3 章）。
目标 = **View → Presenter → Model → 框架层**。中间那一层现在完全不存在（`MVP/Presenter/` 0 文件）。

---

## 2. Presenter 的职责与接口形态

### 2.1 职责边界（可据此判对错）

| Presenter **必须**做 | Presenter **禁止**做 |
| --- | --- |
| 在 `OnAttach` 订阅 Model 事件、在 `OnDetach` **全部退订** | 持有 `GameObject` / `Transform` / `Button` / `TMP_Text` |
| 把 Model 的数据**映射成** View 能直接用的结构（`*VM`） | 直接 `Resources.Load` / 直接读 Json |
| 把 View 的意图翻译成 Model 的一次方法调用 | 写存档文件、写 `PlayerPrefs` |
| 处理「View 还没准备好」的时序（`Attach` 前不调 View） | 在构造函数里调用 View 的任何方法 |
| 幂等：重复 `Attach` 不重复订阅 | 继承 `MonoBehaviour` |

> 判定口诀：**Presenter 里出现 `UnityEngine.UI` 的 `using` = 分层错误；Presenter 里出现 `Resources.` = 分层错误。**

### 2.2 基类契约（新增文件 `MVP/Presenter/PresenterBase.cs`）

```csharp
namespace Chaos.MVP.Presenter
{
    /// 所有 Presenter 的基类。生命周期只有 Attach / Dispose 两段。
    public abstract class PresenterBase<TView> : System.IDisposable where TView : class
    {
        protected TView View { get; private set; }

        public bool IsDisposed { get; private set; }

        /// 由 View 在 OnEnter 里调用。重复调用是空操作（幂等）。
        public void Attach(TView view)
        {
            if (view == null) throw new System.ArgumentNullException(nameof(view));
            if (View != null) return;              // 幂等：已挂载则忽略
            if (IsDisposed) throw new System.ObjectDisposedException(GetType().Name);

            View = view;
            OnAttach();                            // 子类在这里订阅 Model 事件 + 首次刷新
        }

        public void Dispose()
        {
            if (IsDisposed) return;
            IsDisposed = true;
            OnDetach();                            // 子类在这里退订（必须与 OnAttach 成对）
            View = null;
        }

        /// 子类必须成对实现：OnAttach 订阅什么，OnDetach 就退订什么。
        protected abstract void OnAttach();
        protected abstract void OnDetach();
    }
}
```

设计要点：
- **`Attach` 幂等**：Panel 的 `Show()` 可能被重复触发（`PushPanel` 已有防重逻辑，但 P 层不该依赖它）。
- **`Dispose` 只做退订**，不调 View（View 可能正在销毁，调它会 NRE）。`OnDetach` 里若必须通知 View，用「先置空再判空」的写法。
- **不提供 `Update`/`Tick`**：需要每帧逻辑的系统，由 Model 持有计时器，或让 View 自己在 `Update` 里把时间推给 Presenter。不给基类塞 `Update`，是为了避免「Presenter 变成第二个 MonoBehaviour」。

### 2.3 View 抽象接口（写在 Presenter 目录，由 View 实现）

```csharp
namespace Chaos.MVP.Presenter.Quest
{
    /// View 能做什么：只有「显示」与「把玩家操作抛出来」。
    public interface IQuestView
    {
        void ShowQuestList(System.Collections.Generic.IReadOnlyList<QuestItemVM> items);
        void ShowDetail(QuestDetailVM detail);
        void ShowEmptyState();                 // 无数据时的表现，避免 View 自己判断

        event System.Action<uint> QuestSelected;   // 玩家点了某条任务
        event System.Action CloseRequested;        // 玩家点了关闭
    }
}
```

- 接口**只暴露 View 的能力**，不暴露 View 的实现细节（不给 `GameObject`、不给 `Button`）。
- **事件用强类型**（`Action<uint>`），不用字符串事件名 —— 字符串事件名留给跨系统广播（见第 5 章）。
- `*VM` 是与 View 约定好的展示数据（`Presenter` 目录下，`sealed class` 或 `readonly struct`），**不继承 MonoBehaviour**，可含已本地化好的字符串。

### 2.4 一个完整样板（Quest）

```csharp
// MVP/Presenter/Quest/QuestPresenter.cs
namespace Chaos.MVP.Presenter.Quest
{
    public sealed class QuestPresenter : PresenterBase<IQuestView>
    {
        private readonly Chaos.MVP.Model.Quest.QuestModel _model;

        public QuestPresenter(Chaos.MVP.Model.Quest.QuestModel model)
        {
            _model = model ?? throw new System.ArgumentNullException(nameof(model));
        }

        protected override void OnAttach()
        {
            _model.OnQuestChanged += HandleQuestChanged;   // Model → Presenter
            View.QuestSelected   += HandleQuestSelected;   // View  → Presenter
            View.CloseRequested  += HandleCloseRequested;
            RefreshAll();                                   // 首次进入即渲染，View 不用自己拉数据
        }

        protected override void OnDetach()
        {
            _model.OnQuestChanged -= HandleQuestChanged;   // 与 OnAttach 严格成对
            if (View != null)                              // Dispose 时 View 可能已销毁
            {
                View.QuestSelected  -= HandleQuestSelected;
                View.CloseRequested -= HandleCloseRequested;
            }
        }

        private void HandleQuestChanged() => RefreshAll();

        private void RefreshAll()
        {
            if (View == null) return;
            var items = _model.BuildDisplayList();          // Model 负责映射成 VM
            if (items.Count == 0) View.ShowEmptyState();
            else                  View.ShowQuestList(items);
        }

        private void HandleQuestSelected(uint questId) => _model.Select(questId);
        private void HandleCloseRequested()            => _model.CloseDetail();
    }
}
```

---

## 3. Model 的创建与释放：唯一入口 `ModelHub`

### 3.1 问题

现状 `IModel` **0 实现、0 可执行引用**（`01_架构现状核查.md` 第 4 章）。若只把接口补齐而不规定「谁 new、谁 Dispose」，它会重新退化成一个装饰品 —— 这正是 `Learn/MVP架构与配置数据分层说明.docx` 第 5 章的核心告诫，本团队采纳。

### 3.2 接口改造（把 `IModel` 从 `Recorder.cs` 里搬出来）

```csharp
// MVP/Model/IModel.cs  ← 从 GameBase/ConfigManager/SL/Recorder.cs:12-28 迁出
namespace Chaos.MVP.Model
{
    /// 运行时模型：有生命周期，由 ModelHub 创建与释放。
    /// 注意：Excel 生成的 *Config / *Data 是纯字段 DTO，【禁止】实现本接口。
    public interface IModel : System.IDisposable
    {
        void Init();
    }
}
```

改动点（相对现状）：
- `Dispose()` 从自定义方法改为**继承 `System.IDisposable`**（可直接配 `using`，符合 C# 约定）。
- 删掉被注释掉的 `OnDataChanged`（`:24-27`）—— 模型通知用**强类型 event**，不用 `Action<string,object>`。
- 搬出 `Recorder.cs`，放进 `MVP/Model/`（接口定义在一个存档文件里是现状的错位）。

### 3.3 唯一入口

```csharp
// MVP/Model/ModelHub.cs
namespace Chaos.MVP.Model
{
    /// 运行时模型的唯一创建与释放入口。业务代码【禁止】直接 new 模型。
    public static class ModelHub
    {
        private static readonly System.Collections.Generic.List<IModel> _live
            = new System.Collections.Generic.List<IModel>();

        public static T Create<T>() where T : class, IModel, new()
        {
            var model = new T();
            model.Init();                 // 统一在入口 Init，调用方不需要记得
            _live.Add(model);
            return model;
        }

        public static void Release(IModel model)
        {
            if (model == null) return;
            if (!_live.Remove(model)) return;   // 不在册说明已被释放，幂等返回
            model.Dispose();
        }

        /// 场景卸载 / 退出游戏时统一释放，防止跨场泄漏。
        public static void ReleaseAll()
        {
            for (int i = _live.Count - 1; i >= 0; i--) _live[i].Dispose();
            _live.Clear();
        }

        public static int LiveCount { get { return _live.Count; } }  // 供自测断言
    }
}
```

**规则**：

| 场景 | 谁创建 | 谁释放 | 说明 |
| --- | --- | --- | --- |
| 系统级 Model（要跨面板存活） | `Entry.Start()` 或对应的 `GameBase/<X>Manager.Init()` 调 `ModelHub.Create<T>()` | 同上，在 `OnDestroy`/`ReleaseAll` | 例：设置、背包、任务 |
| 面板级 Model（只服务于一个面板） | 面板首次 `OnEnter` 里 `ModelHub.Create<T>()` | 面板的 `OnExit` 里 `ModelHub.Release` | 例：单次结算界面的临时数据 |
| **DTO**（`*Config`/`*Data`） | 由 `ConfigRepository`/`Recorder` 反序列化产生 | GC | **不进 `ModelHub`、不实现 `IModel`** |

**硬约束**：`View` 永远不创建 Model；`Presenter` 永远不创建 Model（只接收注入的引用）。唯一的例外是「面板级 Model」，此时由**面板**创建（面板是入口，不是显示层逻辑）——且必须成对释放。

### 3.4 `SettingsManager` 怎么办

`SettingsManager` 现在是最接近正确的一例（有明确权威、双份数据、事件），但它继承 `SingletonBase` 而非 `IModel`。**本路线不强行改造它**（改动面 6 文件 / 43 处，风险最高，放 P3），而是：
- 在 P3 之前，为它提供一个**只读外观** `ISettingsModel`（`Applied`/`Pending`/`HasPendingChanges`/`BeginEdit`/`CommitEdit`/`RevertEdit`），由 `SettingsManager` 实现；
- P3 时 Presenter 只依赖 `ISettingsModel`，把这 43 处 Manager 单例调用收编为构造注入。

这样 P3 可以「只替换依赖来源，不改行为」，把回归面压到最小。

---

## 4. View 如何被 Presenter 驱动

### 4.1 时序（面板的 `OnEnter` / `OnExit` 是唯一挂载点）

```
UIManager.PushPanel(PanelType.QuestPanel)
   └─ BasePanel.Show()               UIManager.cs:124 / BasePanel.cs:80-94
        ├─ gameObject.SetActive(true)
        ├─ OnEnter()                  ← 【View 的 OnEnter 里挂载 Presenter】
        │     └─ _presenter = new QuestPresenter(_model);
        │        _presenter.Attach(this);     → OnAttach() 订阅 + 首次刷新 → View.ShowQuestList(...)
        └─ anim.PlayEnter()

UIManager.PopPanel()
   └─ BasePanel.Hide()               UIManager.cs:144 / BasePanel.cs:99-109
        ├─ OnExit()                   ← 【View 的 OnExit 里释放 Presenter】
        │     └─ _presenter.Dispose();  → OnDetach() 全部退订
        │        _presenter = null;
        └─ anim.PlayExit() → 动画结束才 SetActive(false)
```

关键点（都有现状依据）：

1. **`OnExit` 在退场动画开始之前调用，且不在动画完成回调里** —— 这是既有设计（`BasePanel.cs:104-106` 注释说明「动画被打断时完成回调不会触发，放那里会让业务回调被永久吞掉」）。因此**退订必须放在 `OnExit`**，这样即使动画被打断也不会泄漏订阅。
2. **View 不要在 `Awake` 里 `Attach`**：`SettingsPageBase.cs:24-28` 记录了一个真实踩坑 ——「必须【先激活再绑定】，因为各行控件的引用是在 `Awake` 里找的，未激活的物体 `Awake` 不会跑」。P 层沿用同一时机约定：**在 `OnEnter` 里挂载**。
3. **一个面板一个 Presenter，一个 Presenter 一个 View**：不要多个 Presenter 共用一个 View（订阅成对性会被破坏）。
4. **View 不缓存从 Presenter 收到的数据到字段里**（除显示所需），业务状态的唯一家在 Model。

### 4.2 View 的骨架（照抄即可）

```csharp
// MVP/View/Quest/QuestPanel.cs
namespace Chaos.MVP.View.Quest
{
    public sealed class QuestPanel : BasePanel, IQuestView
    {
        [SerializeField] private Transform _listRoot;
        [SerializeField] private GameObject _cellPrefab;

        private Chaos.MVP.Presenter.Quest.QuestPresenter _presenter;
        private Chaos.MVP.Model.Quest.QuestModel _model;    // 由 QuestManager 注入或面板级创建

        public event System.Action<uint> QuestSelected;
        public event System.Action CloseRequested;

        public override void OnEnter()
        {
            if (_model == null)
                _model = Chaos.MVP.Model.ModelHub.Create<Chaos.MVP.Model.Quest.QuestModel>();
            _presenter = new Chaos.MVP.Presenter.Quest.QuestPresenter(_model);
            _presenter.Attach(this);
        }

        public override void OnExit()
        {
            if (_presenter != null) { _presenter.Dispose(); _presenter = null; }
            // 面板级 Model 在这里 Release；系统级 Model 不在这里释放
        }

        // ── IQuestView 实现：只显示，不判断业务 ──
        public void ShowQuestList(System.Collections.Generic.IReadOnlyList<QuestItemVM> items) { /* 生成 cell */ }
        public void ShowDetail(QuestDetailVM detail) { /* 填文本 */ }
        public void ShowEmptyState() { /* 空态 */ }
    }
}
```

---

## 5. EventCenter / EventManager 放在哪一层

### 5.1 结论

| 项 | 位置 | 判定 |
| --- | --- | --- |
| `EventCenter` 类 | 现状 `GameBase/EventCenter/EventCenter.cs:42`，**`[Obsolete("该类未进行使用，请使用EventManager", true)]`** | **删除**。`error:true` 意味着任何人调用它都会**编译失败**；它已不是「弃用」，是「禁止使用」。留着只会让新人困惑 |
| `EventManager` + `EventTriggerExt` | `GameBase/EventCenter/EventManager.cs:24`、`:8` | **保留，是框架层（GameBase）的服务**，位置正确，不改 |
| `EventConstName` | `GameBase/EventCenter/EventConstName.cs` | 保留。事件名常量表，新增事件必须在此追加 |
| `CustomEventArgs` | `GameBase/EventCenter/CustomEventArgs.cs` | 保留，事件参数基类 |
| MVP 层内部通信 | Model ↔ Presenter | **不用 EventManager**，用强类型 C# event |
| 跨系统广播 | 任意层 → 任意系统 | **用 `EventManager`**（`this.TriggerEvent(EventConstName.X, args)`） |

### 5.2 为什么 MVP 层内不用 EventManager

`EventManager` 的签名是 `AddListener(string eventName, EventHandler handler)` / `TriggerEvent(string, object sender, EventArgs)`（`EventManager.cs:29,43,49`）——**字符串键 + `object sender` + `EventArgs` 基类**，全部是弱类型：

- 拼错事件名**不报错**，静默不触发；
- 参数要 `as` 强转，转错在运行时才炸；
- 「谁订阅了什么」无法静态检索。

而 MVP 层内的通信是**已知的、一对一的、编译期可查的**，用强类型 `event Action<T>` 成本更低、更安全。

**现状证据**：工程里 `EventManager` 的真实使用只有 3 个生产系统 —— `ScenesLoadManager.cs:45`、`SettingsManager.cs:71,183`、`InputManager.cs:1169,1176`，全部是**跨系统广播**（加载进度、设置已应用、按键按下）。没有一个用于「层内一对一」。契约与既有用法一致。

> 补充（现状小缺陷）：`EventManager.AddListener` 只加不减会累积（`:37-41` 的 remove 用 `-=`，但 `EventConstName` 只有 5 个事件）。`TriggerEvent` 用 `?.Invoke` 是安全的。**唯一已知的实际问题是 `InputManager.cs:229`**：`inputOverridesJson` 为空串时 `JsonUtility.FromJson("")` 抛异常并打 WARN —— 应在解析前 `string.IsNullOrEmpty` 短路。列进 P2。

---

## 6. 跨层数据流（三条典型通路）

### 6.1 只读配置表（策划数值 → 界面）

```
Excel/Chaos_excel/QuestConfig_任务表.xlsx
   │  Tool/Excel/Export Configs（编辑器菜单）
   ▼
Resources/Data/Json/Runtime/QuestConfig.json        （生成物，勿手改）
Scripts/Runtime/MVP/Model/ConfigData/QuestConfig.cs （生成物，勿手改）
   │  运行时：ConfigRepository.Load<QuestConfig>()   ← 修复 C1 后的唯一读取入口
   ▼
DataList<QuestConfig>                                （反序列化结果）
   │  手写包装：MVP/Model/Quest/QuestConfigTable.cs（建 id→行 索引）
   ▼
QuestModel（持有 table，业务规则用它）
   │  强类型 event
   ▼
QuestPresenter → IQuestView.ShowQuestList(List<QuestItemVM>)
```

要点：**配置只被 Model 读一次并缓存**；View 拿不到 `DataList`，只拿映射好的 `*VM`；生成物永远不被手改（`03_路径与命名规范.md` 第 6 章）。

### 6.2 可写存档（玩家进度 → 落盘）

```
QuestModel.CompleteQuest(id)
   │  Model 只改内存状态
   ▼
Recorder.SaveData<QuestProgressData>(list, save:false)   ← 缓存
   │  玩家到存档点 / 退出面板
   ▼
Recorder.ForceSave()  →  Application.persistentDataPath/Records/QuestProgressData.record
```

要点：**存档路径必须是基准④（`persistentDataPath`）**，不是 `Application.dataPath`；**`ForceSave` 必须遍历 `_cache` 而不是磁盘目录**（否则新建键静默丢失，`01_架构现状核查.md` R4）。

### 6.3 面板开关（UI 栈）

```
View: 点击「任务」
   └─ UIManager.Instance.PushPanel(PanelType.QuestPanel)    ← 唯一开面板方式
        └─ SpawnPanel: Resources.Load<GameObject>("UIPanels/QuestPanel")
             └─ BasePanel.Show() → OnEnter() → Presenter.Attach()
```
要点：面板的打开是**框架层职责**（`UIManager`），Presenter **不负责开面板**（否则 Presenter 依赖 UI 栈，变成第二套 UIManager）。需要跳转时，由 View 调 `UIManager`，或由 Model 抛 event、由外层协调者订阅。

---

## 7. 分阶段落地路线（按风险从低到高）

> 每阶段独立可验收、可单独回滚。**P0 与 P1 不涉及 MVP 契约，因而可以先做、且收益最大。**

### P0 —— 消除确定性缺陷（不动架构）

| # | 动作 | 对应问题 | 成本 | 风险 |
| --- | --- | --- | --- | --- |
| P0-1 | `GlobalPath.data_JsonPathToRead` → `"Data/Json/Runtime/"` | C1 | 小 | 低 |
| P0-2 | `ConfigManager.cs` 的 `using UnityEditor;` 移进 `#if UNITY_EDITOR`（**更优做法：整文件迁到 `Scripts/Editor/ExcelTool/`**） | C2 | 小（移动） | 低 |
| P0-3 | 删除 `EventCenter/EventCenter.cs` | C7/D2 | 小 | 低 |
| P0-4 | 删除 `GameBase/UIManager/OtherUIManager/`（`TangLaoShi.*`，0 引用） | D3/D4 | 小 | 低 |
| P0-5 | 删除 `GameTest/*` 在 `TestScene` 的 `GameManager` 上的 5 个测试组件（或移入独立 `GameTest` 场景） | C8 | 小 | 低 |
| P0-6 | 删除 `MVP/Model/DialogueRuntimeData.cs`（0 引用），或补转换器后启用 | D5 | 小 | 低 |

**执行顺序**：P0-1 → P0-2 → 其余（删除类先做，减少编译面）。

**验收方式**：
1. Unity 0 编译错误（`unity_get_compilation_errors`，`severity=error` 计数为 0）。
2. Play `TestScene`：Console 无 `NullReferenceException`。
3. **Unity 实测**断言：`Resources.Load<TextAsset>("Data/Json/Runtime/TestTableConfig") != null`，且 `ConfigLoader.LoadData<TestTableConfig>().datas.Count == 4`。
4. 出一次 **Development Build**（Windows）成功 —— 这是 `using UnityEditor` 的唯一真实验证方式。
5. 导出菜单 `ExcelTool/*` 仍可正常导出（若 P0-2 采用了「迁移到 Editor/」，菜单路径改为 `Tool/Excel/*`）。

---

### P1 —— 数据通路定形（配置与存档各只剩一条）

| # | 动作 | 对应问题 | 成本 | 风险 |
| --- | --- | --- | --- | --- |
| P1-1 | 新建 `GameBase/ConfigManager/ConfigRepository.cs`（唯一只读配置入口：正确路径、异步+缓存、失败可诊断、`TryLoad`），删除 `ConfigLoader.cs` | C1/C3、D7 | 中 | 低 |
| P1-2 | `Recorder`：路径改 `persistentDataPath`；`ForceSave` 遍历 `_cache`；构造函数不再做 IO（改显式 `Init()`）；异常不再包成 `new Exception(ToString())` | C3/R2/R4/R5/R6 | 中 | 中 |
| P1-3 | 导出器 Data 模式输出改 `Resources/Data/Default/`，与 `Data/Records/` 分离 | C5 | 小 | 低 |
| P1-4 | 生成器补 `<auto-generated>` 文件头；删掉失效的 `//用于{X}Model类…` 注释；补重名查重（重名报错跳过）；`GetEnd` 告警用 `endIndex` | C4、6.5 | 小 | 低 |
| **P1-4b** | **空页签/空表处理修复**（Lead 已裁决为确凿缺陷 C11，本阶段只记录不改）：守卫改为 `Dimension == null \|\| Dimension.End.Row < 7` → `continue`；`GetFiles("*")` 结果**排序**；日志**逐表**报告「已导出 / 已跳过」 | **C11** | **小** | **低** |
| P1-5 | `GlobalPath` 按 `ui_`/`res_`/`data_`/`save_` 四组重排并标注基准（`03_路径与命名规范.md` 2.2） | C1 根因 | 小 | 低 |
| P1-6 | 根 `.gitignore` 加 `Assets/Data/Records/`；`git rm --cached` 现有 `.record` | C5 | 小 | 低 |
| P1-7 | `InputManager.LoadOverridesJson` 空串短路（消除 Play 期 WARN） | 5.2 | 小 | 低 |
| P1-8 | 抽出唯一的 `FormatJson`（现 `ConfigManager.cs:424-501` 与 `Recorder.cs:232-309` 双份） | D11 | 小 | 低 |
| P1-9 | 修 Json 组装的数据完整性：`Convert` 对 `string` 不转义（`:355`）+ 用 `json.Split(',')` 重组（`:379`）→ 单元格含英文逗号/引号/换行时**整表 Json 损坏且日志显示成功**。改为逐值转义并直接拼对象（或走 `JsonUtility.ToJson`） | 6.5、策划 L3/L4 | 中 | 中（会改变产物排版，需重导一次并比对内容等价） |

#### P1-4b 专项：空页签 / 空表缺陷（Lead 已裁决并实测，正式记录与排期）

Lead 已于 2026-10-05 复核裁决：本阶段**以代码行为为准、不改导出器**，但把「空页签/空表导致其后所有表不导出」列为**确凿缺陷**，要求在 T1 正式记录、在落地方案里给出修复排期与成本/风险。对应本文件 **P1-4b**。

Lead 另用 **EPPlus 4.5.3 在 Unity 内实测**，裁定该缺陷有**两条不同路径**（见下表「触发条件」），并指出原守卫 `Dimension.End.Row == 0` **不可达** —— 真实失败是 `Dimension == null` 引发的 NRE。本节已按实测结论更新修复方案，并同步进 `01_架构现状核查.md` 的 C11 与 §14.1。

| 维度 | 内容 |
| --- | --- |
| 缺陷 | `ConfigManager.cs:88-89`（Config 模式）、`:124-125`（Data 模式）在 `foreach (var file in files)` 里写 `return`，应为 `continue` |
| 放大因素 | 处理顺序由 `FileUtil.cs:27` 的 `directory.GetFiles("*")` 枚举顺序决定（文件系统相关）。叠加 `return` 后：**哪些表没导出是不确定的**，且两次导出都只打印「已导出所有」，看起来都成功 |
| 触发条件<br>（**Lead 已用 EPPlus 4.5.3 在 Unity 内实测裁定**） | ① **页签完全空白** → `workSheet.Dimension == null` → `:88`/`:124` 取 `.End` 时抛 **NRE**，异常冲出 `foreach`，被 `:99`（Config）/ `:135`（Data）的 catch 记 error，**其后所有表都不导出**。<br>② **只有表头**（第 1–6 行有内容、无数据行，`End.Row = 6`）→ 通过 `:88` 守卫 → `GetValues` 返回空 → `str == ""` → `:205-209` 打日志后 `return`，**只跳过本表**，不影响后续表，也**走不到**第 4 行。<br>③ **补充结论**：原守卫 `if (Dimension.End.Row == 0) return;` **不可达**（空页签是 `null`，`.End.Row` 永不为 `0`）；即便可达也应改为 `continue` |
| 修复方案 | ① **守卫重写**：`if (workSheet.Dimension == null \|\| workSheet.Dimension.End.Row < 7) { /* 逐表日志"已跳过：空表 / 无数据行" */ continue; }` —— 一并覆盖「完全空白」（原 NRE 路径）与「只有表头」（原 `:205` 提前 return 路径），并替换掉**不可达**的 `End.Row == 0` 判断。<br>② `LoadFiles` 结果按文件名 `OrderBy` 排序，使处理顺序确定可复现。<br>③ 日志改为**逐表**输出「已导出 / 已跳过（原因）」，不再只打印一句「已导出所有」。<br>④ `:99`/`:135` 的 catch 保留，但应把「已处理到第几张表」一并写进 error，便于定位中断点 |
| 修复排期 | **P1**（与 P1-4 同批，属导出器改造，不单独立阶段）。未修复前，`Story/03_配置表设计/01_配置表填写规范.md` 的 L1 限制继续生效：**源表目录不得出现空表/空白页签** |
| 成本 | **小**（4 处小改动，均在 `ConfigManager.cs` 与 `FileUtil.cs`，无 API 变化，不影响生成物格式） |
| 风险 | **低**。改动只影响「遇到空页签/无数据行时是否继续处理后续表」：修复前完全空白的页签会**中断整批**，修复后变成**跳过该表并继续**；表头-only 的表在修复前后都是被跳过（修复前走 `:205` 提前 return，修复后走守卫 `continue`），**已正常导出的表内容不受影响**。残余风险是「原先被中断而从未导出的表，修复后开始导出」，可能暴露此前没发现的表级问题（如第 4 行空列）——属**暴露既有问题**，非引入新问题 |
| 验证方式 | ① 源表目录放三张表：**完全空白页签**的表、**只有表头**的表、正常表，且让前两张按文件名排在正常表**之前**。修复前：正常表不产出（被 NRE / return 中断）；修复后：正常表产出，前两张被跳过且各有日志。<br>② 连续跑两次导出，产物 SHA256 一致（验证排序带来的顺序确定）。<br>③ Console 能逐表看到「已导出 / 已跳过（原因）」。<br>④ 空白页签表不再产生 NRE 堆栈 |
| 责任边界 | 架构师负责本项排期与验收标准（本表）；**执行由程序**在 P1 阶段完成；本阶段（T1）不动手 |

> 备注（对 T5 的输入）：策划文档把该缺陷编号为 **L1**，与本文件 C11 / P1-4b 是同一件事。Lead 已通知 designer 在修正 L12/A9 时交叉引用本文件的 C11 / P1-4b；T5 评审时只需**确认编号已合并**，避免程序同时看到两套编号。

**验收方式**：
1. `ConfigRepository.Load<TestTableConfig>()` 返回 4 条（用真实表断言），且**不存在第二个读 Json 的类**（grep 确认）。
2. 存档自测：`SaveData(save:false)` 一个**磁盘上原本不存在的新键** → `ForceSave()` → 重新 `Recorder.Instance` 后能读到该键（**这条直接验证 R4**）。
3. 删除 `persistentDataPath/Records/` 目录后启动不崩（验证 R5：目录不存在不再抛异常）。
4. 跑一次 `Tool/Excel/Export Configs|Models` 后 `git status` 干净（生成物不该出现在工作区改动里）。
5. 生成的 `.cs` 第一行是 `<auto-generated>`。
6. 制造两张 `A_B.xlsx` / `A_C.xlsx`，导出时应**报错**而不是静默覆盖（验证重名查重）。

---

### P2 —— MVP 骨架落地 + 第一个样板系统

| # | 动作 | 成本 | 风险 |
| --- | --- | --- | --- |
| P2-1 | `MVP/Model/IModel.cs`（`IDisposable` + `Init`），并**从 `Recorder.cs` 删掉旧定义** | 小 | 低 |
| P2-2 | `MVP/Model/ModelHub.cs`（唯一创建/释放入口 + `ReleaseAll`） | 小 | 低 |
| P2-3 | `MVP/Presenter/PresenterBase.cs` | 小 | 低 |
| P2-4 | 样板系统选 **SLPanel（存读档）**：它 UI 已完整（`MVP/View/SLPanel.cs` + `SLPanel_Record/RecordCell.cs`），且正好打通 `Recorder` | 中 | 中 |
| P2-5 | 新增 `MVP/Presenter/SL/ISaveLoadView.cs` + `SaveLoadPresenter.cs`；`MVP/Model/SL/SaveLoadModel.cs`（持有存档槽列表） | 中 | 中 |
| P2-6 | `Entry.cs` 增加 `ModelHub.ReleaseAll()` 的退出钩子（`OnApplicationQuit`/`OnDestroy`） | 小 | 低 |
| P2-7 | 在 `Story/05_程序/` 写「MVP 落地记录」，把契约固化为可抄的范例 | 小 | 低 |

**为什么第一个样板选 SLPanel 而不是设置面板**：设置面板的视图层最复杂（页/子页/行/重绑定），而 SLPanel 是「列表 + 读档 + 存档」三件事，能完整覆盖 `ModelHub` / `PresenterBase` / `Recorder` / `ConfigRepository` 四者，且不触碰最高风险的设置链路。

**验收方式**：
1. `SLPanel` 的 View 里**不再出现** `Recorder.Instance`（改为经 Presenter/Model）。
2. `ModelHub.LiveCount` 在面板反复开关 20 次后回到开关前的值（**验证无泄漏**：订阅成对）。
3. 面板 `OnExit` → Presenter `Dispose` → 再次 `OnEnter` 能正常显示（验证幂等 + 重挂载）。
4. 用 `ModelHub.Create<SaveLoadModel>()` 创建，业务代码里 grep 不到 `new SaveLoadModel`。
5. 生成存档后重启编辑器仍能读到（验证 P1 的持久化在 MVVM 链路上真正打通）。

---

### P3 —— 存量 View 收编（风险最高，最后做）

| # | 动作 | 成本 | 风险 |
| --- | --- | --- | --- |
| P3-1 | 为 `SettingsManager` 提供 `ISettingsModel` 只读外观（`Applied`/`Pending`/`HasPendingChanges`/`BeginEdit`/`CommitEdit`/`RevertEdit`/`ResetPendingSection`） | 中 | 低 |
| P3-2 | 新增 `MVP/Presenter/Settings/SettingsPresenter.cs` + `ISettingsView.cs` | 中 | 中 |
| P3-3 | 把 `SettingsPageBase` 的 7 处 `SettingsManager.Instance` 改为注入的 `ISettingsModel`（`SettingsPageBase.cs:168,207,210,253,256,273,276`） | 中 | **高** |
| P3-4 | 把 `SettingPanel.cs` 的 9 处调用改为经 Presenter（`:84,93,103,309,328,353,368,396,407`） | 中 | **高** |
| P3-5 | `KeybindBindingsPage.cs`（8 处）与 `GraphicsSettingsPage.cs`（1 处）同步 | 中 | **高** |
| P3-6 | `MVP/Model/` 与 `MVP/View/` 按 `<System>/` 分子目录；新代码启用 `Chaos.MVP.*` 命名空间 | 小 | 中 |

**风险控制**：P3 是**行为保持型重构**（只换依赖来源、不改语义），必须逐项回归：设置改值 / 应用 / 返回丢弃（`RevertEdit`）/ 恢复默认 / 按键重绑定与回滚 / 语言切换 / 音量试听。**建议 P3 由程序在 P2 样板稳定后单独排期，不要与 P0/P1 混做。**

---

### P4（可选，暂缓）—— 明确不建议现在做的

| 项 | 为什么暂缓 |
| --- | --- |
| 给 `Recorder` 加多存档位（槽位 + 类型） | 现无任何业务消费；等 SLPanel 样板确定槽位语义后再设计（`Recorder.cs:97,122,125` 现在一律以 `typeof(T).Name` 为键） |
| `ResManager` 拆成 `Load`/`Instantiate` | 属 API 清理，影响多处调用；列 P2 之后 |
| 全量把类型迁进命名空间 | prefab 序列化引用面大、收益低；只在**新增**代码用命名空间（`03_路径与命名规范.md` 4.2） |
| 引入 DI 容器 / 第三方 MVP 框架 | 工程规模不需要，构造函数注入足够 |
| 把 `SettingsManager` 改成 `IModel` | 它是「服务」不是「模型」，改成 `IModel` 只为凑架构，收益为负；P3 用 `ISettingsModel` 外观即可 |

---

## 8. 逐项「可行性 / 优化方向 / 成本 / 风险」总表

| 项 | 可行性 | 优化方向 | 成本 | 风险 |
| --- | --- | --- | --- | --- |
| 修 `data_JsonPathToRead` | 高（改 1 行，Unity 实测可验） | 顺带按四组基准重排 `GlobalPath`，从根上防复现 | 小 | **低** |
| 导出器迁 `Scripts/Editor/` | 高（纯搬移） | 顺带把菜单收进 `Tool/Excel/` 命名空间，并把菜单名与 README 对齐 | 小 | 低（需重新确认菜单路径与文档一致） |
| 删除 `EventCenter` / `OtherUIManager` / `DialogueRuntimeData` | 高（0 可执行引用，已 grep 验证） | 删除前再跑一次全量 grep 确认，并让 Unity 编译一次兜底 | 小 | 低 |
| 修 `Recorder`（路径 + ForceSave + 构造 IO） | 高 | 引入 `IRecordStore` 抽象，便于单测与未来接云存档 | 中 | 中（要重存一次档，需迁移已存在的 `Data/Records/*.record`） |
| 建 `ConfigRepository` | 高 | 增加「加载失败可诊断」（返回错误码而非 null）、支持预加载清单、避免首帧卡顿 | 中 | 低 |
| 生成器补文件头 / 查重 / 空表 continue | 高 | 把「重名报错」做成导出前置校验，一次性列出所有冲突表 | 小 | 低（改了导出行为，需重导一次验证产物差异） |
| `Resources/Data/Default/` 分离生成物与存档 | 高 | 同时把 `Data/Records/` 加 ignore，彻底消除工作区脏化 | 小 | 低 |
| `IModel` 改造 + `ModelHub` | 高（当前 0 实现，无迁移包袱） | 增加「重复 Release 幂等」与泄漏自检日志（`ModelHub.LiveCount`） | 小 | 低 |
| `PresenterBase<TView>` | 高 | 增加「未 Attach 就调 View」的防御性断言（编辑器期） | 小 | 低 |
| 第一个样板系统（SLPanel） | 高（UI 已就绪） | 沉淀为 `Story/05_程序/` 的可抄范例 | 中 | 中（要动现有面板） |
| 收编设置链路（P3） | 中 | 用 `ISettingsModel` 只换依赖来源、不改行为；逐项回归清单 | **大** | **高** |
| 中文注释/文档与代码同步 | 高 | 每阶段结束更新 `01_架构现状核查.md` 的对应条目（标「已修」而非删除） | 小 | 低 |

---

## 9. 给 task-5「落地方案评审」的预判（策划写文档时必须注意什么）

1. **表名规则是最容易返工的一项**：`_` 前英文段决定类名与 json 名，且**不带查重** —— 不同表名不能有相同前缀（`A_B`/`A_C` 会互相覆盖）。策划写新表前必须先查 `Excel/Chaos_excel/` 里已有的前缀。
2. **第 7 行就是数据行，没有示例行**：**三处已统一修正** —— README 已由 Lead 于 2026-10-05 修正，`02_系统策划/` 与 `03_配置表设计/` 已由 T8 同步（见 `Story/README.md` §3；原始冲突与证据见 `01` 第 6 章）。策划若仍按「第 7 行是示例」填表，第 7 行会变成一条真实数据。
3. **第 5 行必须填，且只用 `client` / `both` / `server`**：留空会导致「类里有字段、json 里没有 key」，运行时该字段恒为默认值且不报错。`server` 列会被类与 json 同时剔除（这是唯一真正生效的端标记）。
4. **类型白名单（终版 12 个，task-7 之后）**：`int` `uint` `long` `ulong` `byte` `sbyte` `short` `ushort` `float` `double` `bool` `string`。
   - **`bool` 已支持**（T7 新增）：取值仅 `true`/`false`、`1`/`0`、`是`/`否`（不区分大小写，先 `Trim()`），导出为**裸值** `true`/`false`；其它取值（含**空单元格**）**导出期直接抛异常**，不会静默变成 `false`。
   - **`int32`/`int64`/`long long` 已删除**：现在填了会在**导出期直接抛「不支持此类型」**（不再是「导出成功但生成非法 C# 导致编译不过」）。
   - **`decimal` / `char` 实测后否决**，不要使用：`decimal` 被 `JsonUtility` **静默丢弃**（往返恒 0），`char` 在 JSON 里存成**数字**（`'A'`→`65`）。
   - **权威 = `ConfigManager.Convert` 的 `case` 列表**（不是本文档、不是 Excel 说明文档、不是表头模板）；终版说明见 `Story/README.md` §7，架构师侧的清单与实测证据见 `03_路径与命名规范.md` §9。
5. **需要引用 Sprite / AudioClip / GameObject 的字段不能进 Excel**：Excel 只产出数值与字符串。这类数据要么走 ScriptableObject 资产（如 `DialogueData`），要么在表里存路径字符串再由程序加载。
6. **「是否读取」第 2 行没有实现**：不要依赖它做开关，填了不生效。若确实需要「只导某些表」，应作为导出器改造需求提给架构师（当前未排期）。
7. **第 4 行英文属性名不可为空**：导出器唯一强校验（`ConfigManager.cs:265-268` 抛异常）。名字同时决定 C# 字段名与 json key，**一经发布不可再改**（改了 json key 会让旧存档/旧配置读不到），属于「需要加版本迁移」的变更。
8. **只读第 1 个页签**：多页签的表，第 2 个起一律不导出（`ConfigManager.cs:86`）。
9. **表尾不能留空行、中途不能有空列**（`Dimension` 会带上空行空列）：空列会让 `GetProperties` 抛「第 4 行第 N 列为空」。
10. **「Config vs Data」二分目前是空的**：两者产出内容逐字节相同（实测 SHA256 一致）。策划在 `03_配置表设计/` 里若真要用这个二分，必须说明**运行时差异**（谁能改、是否随存档走），否则建议合并为一张表，避免维护双份。

---

## 10. 契约验收标准（task-5 闸门的判定依据）

- [ ] `MVP/Presenter/` 下存在 `PresenterBase<TView>`，且样板 Presenter 的 `OnAttach`/`OnDetach` 订阅严格成对。
- [ ] `View` 层不出现 `Recorder.Instance` / `ConfigLoader.Instance` / `Resources.Load`（样板范围内）。
- [ ] `Model` 一律经 `ModelHub.Create<T>()` 创建；业务代码 grep 不到 `new <Model>类`。
- [ ] `IModel` 有 ≥1 个真实实现者、≥1 个真实调用点；Excel 生成的 DTO **不实现** `IModel`。
- [ ] MVP 层内通信用强类型 event；跨系统广播才用 `EventManager`；`EventCenter` 已删除。
- [ ] 配置读取入口唯一（`ConfigRepository`），存档入口唯一（`Recorder`），且存档写 `persistentDataPath`。
- [ ] P0 全部完成且「Development Build 成功」这一条有实测记录。
- [ ] `01_架构现状核查.md` 中被修掉的条目已标注「已修（阶段 P?）」，未修条目与暂缓理由保持一致。
