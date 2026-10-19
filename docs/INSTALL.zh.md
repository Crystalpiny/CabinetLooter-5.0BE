# 安装与验收（SPT 5.0BE）

Cabinet Looter 1.0.2（SPT 5.0BE 适配版）—— 打开文件柜任意一个抽屉，整个柜子的抽屉一屏全显示、自动逐个搜索。
原作者 MyButtHasARash（MIT，[sp-mod mod 页](https://sp-mod.com/mod/3103/cabinet-looter)、[原仓库](https://github.com/JoelHauser/CabinetSearch)），本仓库为其 SPT 5.0BE（IL2CPP 客户端）移植版。

## 安装

- `CabinetLooter.Client.dll` 放到 `<SPT客户端根>\BepInEx\plugins\CabinetLooter\` 目录内（目录自己建）。
- 纯客户端插件，无服务端部分，不需要动 `SPT_Runtime`。
- 装前请备份已有的 `BepInEx\plugins\CabinetLooter\`；换 dll 必须先完全退出游戏，BepInEx 只在启动时加载插件。

## 怎么确认生效

启动游戏后打开 `<SPT客户端根>\BepInEx\LogOutput.log`，搜 `Cabinet Looter`，应出现：

```
Cabinet Looter 1.0.2 loaded (SPT 5.0BE).
```

若出现 `could not apply its patches`，说明某个补丁目标签名又变了（此时游戏保持原版行为，不会坏档），把整段日志发出来即可。

## 进游戏怎么验（开一局带文件柜的地图，如立交桥/储备站）

1. F12 打开配置，把 `[3. Debug]` 的 `Log cabinet details` 打开；
2. 对着文件柜（`card_file`）任意抽屉按 F 搜索/打开，应看到整个柜子的抽屉并排显示在一个窗口里，
   每个抽屉带 `DRAWER n` 标题条（当前打开的那格是 `DRAWER n (OPENED)`）；
3. LogOutput.log 应依次出现 `Opened card_file_box_0N ...` 与 `Showing N drawers from 1 cabinet(s) ...`；
   没有 `Showing` 行 = 面板挂接失败，请带日志反馈；
4. 搜索条（SEARCHING/计时/X）会一直跟着正在搜的抽屉；点某个抽屉标题条可单独开始/停止该抽屉；
5. Ctrl+点击非打开抽屉里的物品 = 进玩家背包（不是塞进打开的抽屉）；
6. 开着柜子在 装备→健康→装备 切页签，抽屉应能原样回来。

## 调参（F12，共 7 个键分三组）

- `[1. General]` 显示整个柜子 / 自动逐个搜索 / 续搜半搜抽屉；
- `[2. Cabinet clusters]` 是否连排柜一起显示 + 间距（米）+ 最多几个柜（默认关；同排贴着的柜子会合并显示）；
- `[3. Debug]` 日志开关（平时保持关）。

## 怎么卸

删除整个 `BepInEx\plugins\CabinetLooter\` 目录，再删 `BepInEx\config\com.mybutthasarash.cabinetlooter.cfg`，重启游戏。

## 已知说明与限制

- 移植版已通过静态校验（5 个补丁目标逐签名核对 + 52 个游戏 API 引用全部可解析），但尚未实机进图测试，
  标题条与网格的对齐（1.0.2 的修复项）请进游戏目验；
- 5.0 的 `SearchButton.SetSearchStatus` 多了 `changeButtonEnabling` 参数，此处传 `false` 以保持 4.1.6 行为（按钮始终可点）；
- 清道夫（玩家 scav）局里空抽屉是 SPT 的掠夺削减机制（static loot 打折后 1 件容器按四舍五入变 0），
  不是本 mod 的问题，PMC 局正常；
- Fika 联机未测试（搜索是每人自己的，取物走主机）；与 UIFixes、AutoCorpseSearch 同用时留意日志。
