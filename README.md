# Cabinet Looter — SPT 5.0BE port

Open any drawer of a filing cabinet and loot the whole cabinet from one window: every drawer gets
its own grid in the same panel, searched one after another at normal speed.

打开文件柜任意一个抽屉，整个柜子的抽屉一屏全显示、自动逐个搜索 —— 不用再开一个搜一个关一个。

This repository is a **port of [Cabinet Looter](https://sp-mod.com/mod/3103/cabinet-looter) 1.0.2 to
SPT 5.0BE** (the IL2CPP client). It is client-only, one BepInEx plugin, no server part.

## Credits / 致谢

**Cabinet Looter is the work of [MyButtHasARash](https://sp-mod.com/user/51018/mybutthasarash)
([@JoelHauser](https://github.com/JoelHauser)).** The original — the idea, the cabinet detection
read out of the map scenes, the drawer-by-drawer search chain, the panel compositing, the careful
heading-alignment work and the F12 settings — was built for SPT 4.1.x and published at
[sp-mod.com/mod/3103/cabinet-looter](https://sp-mod.com/mod/3103/cabinet-looter), source at
[JoelHauser/CabinetSearch](https://github.com/JoelHauser/CabinetSearch), MIT License.

This port does one thing only: it moves that work onto SPT 5.0BE's IL2CPP client — new build
system (net10.0 + Il2CppInterop), renamed and moved game APIs, and interop array semantics. None
of the gameplay design here is mine. If the mod saves you time in raid, the thanks belong upstream:
endorse the [original mod page](https://sp-mod.com/mod/3103/cabinet-looter). 🙏

（中文）本仓库只是把原作者 **MyButtHasARash** 的 Cabinet Looter 移植到 SPT 5.0BE 的 IL2CPP 客户端。
创意、柜体识别、逐抽屉搜索链、面板布局、标题对齐等全部玩法设计均出自原作者之手，移植部分仅涉及
编译体系与 IL2CPP 适配。如果这个 mod 帮到了你，请到 [原 mod 页面](https://sp-mod.com/mod/3103/cabinet-looter)
为原作者点赞致谢。

## Features / 功能

- **The whole cabinet in one window.** Every drawer gets its own grid, with a heading showing its
  state: not searched, searching, partly searched, empty, or how many items it holds.
- **Normal searching.** Drawers are searched at the usual speed with your Search and Attention
  skills. Items still turn up one by one.
- **The loot is untouched.** Each drawer keeps the loot the raid gave it. Nothing is rerolled,
  copied or moved into your inventory for you.
- **Progress is kept.** Closing the window interrupts searching, as in vanilla; opening any drawer
  of that cabinet again picks up where it left off.
- **The search bar follows along.** SEARCHING and the timer stay up until the last drawer is done;
  the X stops the whole cabinet. Click a drawer's heading to start or stop that drawer alone.
- **Cabinet clusters (optional, off by default).** Also show the cabinets standing right beside the
  one you opened, one row per cabinet.

## Install / 安装

Unzip over your SPT 5.0BE folder so `CabinetLooter.Client.dll` lands in
`BepInEx\plugins\CabinetLooter\`. Client only, nothing server-side. Restart the game and check
`BepInEx\LogOutput.log` for `Cabinet Looter 1.0.2 loaded (SPT 5.0BE).`

把 `CabinetLooter.Client.dll` 放进 `<SPT客户端根>\BepInEx\plugins\CabinetLooter\`，重启游戏即可；
详细验收步骤见 [docs/INSTALL.zh.md](docs/INSTALL.zh.md)。

## Settings / 设置

Press F12 in game (BepInEx Configuration Manager).

| Setting | Default | |
| --- | --- | --- |
| Show the whole cabinet | on | Off restores vanilla. |
| Search every drawer automatically | on | Off: click a heading to search a drawer. |
| Resume half-searched drawers | on | Vanilla makes you press Search again. |
| Include cabinets standing next to it | off | Cluster mode. |
| Largest gap between cabinets (metres) | 0.3 | Measured side to side. |
| Most cabinets in a cluster | 4 | The nearest ones are kept. |
| Log cabinet details | off | For bug reports: writes to `BepInEx\LogOutput.log`. |

## What the port changes / 移植改动

| 4.1.6 (Mono) original | 5.0BE (IL2CPP) port |
| --- | --- |
| net472 + BepInEx 5, `BaseUnityPlugin` | net10.0 + BepInEx 6 `BasePlugin`, per-frame drive via an injected `PluginUpdater` MonoBehaviour |
| `EFT.Interactive.InteractionContextHelper.OnContainerOpen` | `EFT.InteractionContextHelper.OnContainerOpen` (`System.Action` → `Il2CppSystem.Action`) |
| `ItemUiContext.Configure` (12 args) | 13 args (new `RepairController`), arrays as `Il2CppReferenceArray` |
| Harmony `FieldRefAccess` on `_rightPanelItem` / `_containedGridsView` / `_inventoryController` | interop properties, assigned directly |
| `SimpleStashPanel.Show` (8 args) | 7 args (one `bool` gone); `ContainedGridsView.Show` gained `magnify` |
| `TimerText.Show(float)`-style timer | `TimerText.Show(Il2CppSystem.DateTime)` |
| managed result buffers for `Physics.*NonAlloc`, `GetWorldCorners`, `GetPixels32` | `Il2CppStructArray` / `Il2CppReferenceArray` buffers (a managed array is copied on the way in and comes back empty) |
| `HeadingClick` as a plain MonoBehaviour | registered with `ClassInjector.RegisterTypeInIl2Cpp` |

Full notes: [docs/PORTING-NOTES.zh.md](docs/PORTING-NOTES.zh.md).

## Build from source / 编译

Needs a **.NET 10 SDK** (the game's bundled runtime is .NET 10 and `Il2CppInterop.Runtime`
targets net10.0) and a started-once SPT 5.0BE install (BepInEx generates the interop assemblies
in `BepInEx\interop` on first launch).

```powershell
cd src
dotnet build -c Release -p:DeployToSPT=true   # also copies the dll into BepInEx\plugins\CabinetLooter
```

Set `-p:ClientRoot=<path>` if the game root is not four levels up from `src`.

## Known notes / 已知说明

- Empty drawers in a **scav** raid are SPT, not this mod: when SPT cuts a scav raid's loot by half
  or more, every one-item container on the map (every drawer) spawns empty.
- Fika is untested with this port: searching is per player and item moves go through the host, so
  it should behave like two players at one container always have.
- 清道夫（玩家 scav）局里空抽屉是 SPT 的掠夺削减机制，不是 mod 的问题，PMC 局正常。

## License / 协议

[MIT](LICENSE). Original Cabinet Looter © MyButtHasARash (JoelHauser); SPT 5.0BE port © Crystalpiny.
The port is released under the same MIT License as the original, per the original's license.
