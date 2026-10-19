# 移植说明：Cabinet Looter 1.0.2 → SPT 5.0BE

原 mod：Cabinet Looter 1.0.2（MyButtHasARash，MIT，[JoelHauser/CabinetSearch](https://github.com/JoelHauser/CabinetSearch)），
为 SPT 4.1.6（Mono 客户端，net472，BepInEx 5）构建。
本仓库为 SPT 5.0BE（IL2CPP 客户端，BepInEx 6）移植版，插件名 `CabinetLooter.Client.dll`，GUID 不变。

## 静态校验结果（2026-10-19，对 5.0BE interop）

5 个 Harmony 补丁目标全部在 5.0BE 存在：

| 4.1.6 目标 | 5.0BE 状态 |
| --- | --- |
| `EFT.Interactive.InteractionContextHelper.OnContainerOpen(GamePlayerOwner, System.Action, LootableContainer, float)` | 移到 `EFT.InteractionContextHelper`；callback 变 `Il2CppSystem.Action` |
| `ItemUiContext.Configure(...12 参)` | 13 参：末尾新增 `RepairController`；`CompoundItem[]` → `Il2CppReferenceArray<CompoundItem>` |
| `ItemUiContext._rightPanelItem`（Harmony FieldRef） | 变 interop 属性（get/set），直接赋值 |
| `SimpleStashPanel.Show(...8 参)` | 7 参：少一个 `bool` |
| `SimpleStashPanel.Close()` | 不变（virtual） |
| `EFT.UI.SearchableView.UpdateSearchState()` | 移到 `EFT.UI.DragAndDrop.SearchableView` |

移植 DLL 对 Assembly-CSharp 的 52 个成员引用全部可解析（0 缺失）。

## 相对 4.1.6 原版源码的改动

**构建体系**

- net472/Mono + `EscapeFromTarkov_Data\Managed` → net10.0 + `BepInEx\interop`（IL2CPP interop 程序集）；
  引用 BepInEx.Core / BepInEx.Unity.IL2CPP / Il2CppInterop.Runtime / Sirenix.Serialization（UI 基类链需要）等。
- 需 .NET 10 SDK（游戏自带运行时是 10.0.12，Il2CppInterop.Runtime 按 net10 编译）；SDK 9 会报 CS1705。

**BepInEx 6 插件形态**

- `BaseUnityPlugin` + `Awake/Update` → `BasePlugin`（BepInEx.Unity.IL2CPP）+ `Load()`；
  `BasePlugin` 不是 MonoBehaviour，逐帧驱动 `CabinetSession.TickCurrent()` 改由注入的 `PluginUpdater`（MonoBehaviour，Update）承担。
- `HeadingClick : MonoBehaviour, IPointerClickHandler` 增加 `(IntPtr)` 构造并在 `Load()` 里
  `ClassInjector.RegisterTypeInIl2Cpp`（IL2CPP 自定义组件必经步骤）。

**签名/类型适配**

- `SearchButton.SetSearchStatus(true)` → `SetSearchStatus(true, false)`（5.0 多了 `changeButtonEnabling`，false 保持原行为）。
- 计时器改传 `Il2CppSystem.DateTime`（`SearchContentOperation.StartTime`、`DateTimeExtensions.UtcNow` 同型）。
- `ContainedGridsView.Show(...)` 末尾新增 `bool magnify`，传 `SearchableItemView._magnify`（与原版面板一致）。
- `DefaultItemContext` 从 `EFT.InventoryLogic.Operations` 移到 `EFT.InventoryLogic`（ctor 不变）。
- `Item.GetFirstLevelItems()` → `ItemExtensions.GetFirstLevelItems(ContainerCollection)` 静态调用（扩展方法特性丢失）；
  `IPlayerSearchController.IsSearched/ContainsUnknownItems` 由 `ISearchController` 基接口提供
  （`StopSearching` 参数 string→MongoID，有隐式转换）。
- 三处 `AccessTools.FieldRefAccess` 全部改为 interop 属性直取（`_rightPanelItem`、`_containedGridsView`、`_inventoryController`）。
- `SearchOperations`（`IBindableListWithEvent<T>`，Il2Cpp 可枚举）上的 LINQ 改为手写 foreach
  （`FindOperation` / `FirstLevelItems`）。

**IL2CPP 内存语义陷阱（已在源码注释标注）**

- `GetWorldCorners` / `GetPixels32` / `Physics.OverlapBoxNonAlloc` / `RaycastNonAlloc` 的结果缓冲一律用
  `Il2CppStructArray<T>` / `Il2CppReferenceArray<T>` 静态缓冲：托管数组传进去会被拷贝，结果读不回来。
- `new GameObject(name, typeof(RectTransform))` → `new GameObject(name)` + `AddComponent<RectTransform>()`。
- Il2Cpp 对象相等判断用 `Equals(a, b)`（包装器按原生指针比较），不依赖托管包装器引用同一。
- `BindableEvent<T>.Subscribe` 要 `Il2CppSystem.Action<T>`，由托管 `System.Action<T>` 隐式转换（op_Implicit）。

## 待实机确认

1. 标题条与网格左边对齐（1.0.2 修复项，涉及 LayoutRebuilder 强制布局 + 精灵透明边距测量）；
2. `SetSearchStatus(true, false)` 的第二个参数取值是否与 5.0 原版顶栏手感一致；
3. 集群模式（默认关）的 Physics.OverlapBox 射线遮挡判断。
