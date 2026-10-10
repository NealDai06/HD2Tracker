# HD2Tracker —— 绝地潜兵 2 玩家名册抓取插件（v21.2）

装进游戏，把**「这一局队里有谁」**写成一行行 JSON 到本机日志文件：

```
%LOCALAPPDATA%\CowboyBingus\Helldivers2\Logs\playerLog.txt
```

有人加入/离开对局时**自动**记录，不用按键（`[` 也能手动抓一次全队）。
每行一条记录 = 玩家标识（peer id，游戏内的账号级 ID）+ 玩家名 + 事件类型 + 时间戳。

它本身不做判断，只是"把看到的事实写下来"。配套的桌面端
[**HD2 Blacklist**](https://github.com/NealDai06/HD2Blacklist)（Windows）会尾随这个文件，
在黑名单里的玩家进队时弹无焦点提示 —— **两个要一起用**：插件负责采集，判断与提示都在桌面端。
不装桌面端也行，这个文件是纯文本，自己写脚本消费同样可以。

| 做 | 不做 |
|---|---|
| 读游戏自己的**名册**（`game.dll`）拿玩家名 | ❌ 不发任何网络请求 |
| 读游戏 Lua API 拿玩家标识（peer id） | ❌ 不改游戏包、不改 `data\` |
| 只往本机日志文件**追加**文本 | ❌ 不注入别的进程、不挂钩子、不截图 |
| 事件驱动：有人进/出队就写一条 | ❌ 不采集 SteamID（默认关闭）、不区分平台（见「已知限制」） |

> ⚠️ 任何第三方自动化工具都可能违反游戏的服务条款（ToS）。是否使用请自行评估与承担。

**本仓库只放可安装的产物与说明**：`Addon\`（部署进游戏的资源文件）、`manifest.json`（模组元数据）、
`README.md`、`LICENSE`。开发工程（源码、构建脚本、设计文档）不在公开仓库里。

> 开发期这个插件叫 `hd2list`，所以你在加载器日志里会看到 `mods/hd2list/watch` ——
> 那是**注册路径**（改了就等于变成另一个 mod），与显示名 `HD2Tracker` 无关，不影响使用。

---

## 依赖与前置

| 需要 | 说明 |
|---|---|
| **Bingus Shared Loader v15+** | HD2 的 addon 加载器（第三方，玩家自装）。**本插件不分发它** |
| **Arsenal** 或 **HD2 Mod Manager** | 用来导入、开关模组，以及 Purge & Deploy |
| 游戏版本 | 名字读取对 `game.dll` 两段代码签名做自校验；游戏更新后签名失配 → 自动停用名字读取（peer id 仍然可用） |

## 安装

1. **关掉游戏**（加载器只在启动时读一次模组目录）
2. 取插件包，两种都行：
   - 从 [Releases](../../releases) 下载 `HD2Tracker-v21.2.zip`（推荐，直接给管理器导入）；
   - 或者直接用本仓库里的 `Addon\` + `manifest.json` —— 这两个就是那个 zip 里的内容。
3. 用 Arsenal / HD2MM **导入**
4. 如果装过旧的 `kick_steamid_tracker`：**把它禁用** —— 本插件是它的替代品，
   两者往同一个日志目录里写，同时启用只会互相干扰
5. 确认 **Bingus Shared Loader 仍在启用**，然后 **Purge & Deploy**
6. 启动游戏，验三处：
   - `%LOCALAPPDATA%\CowboyBingus\Helldivers2\Logs\` 下出现 `playerLog.txt` 与 `playerLog.log`
   - `playerLog.log` 里有 `HD2Tracker hd2trackerv21.2 启动（STAGE=3）` 与自检结果
   - 加载器日志（`BingusSharedLoader.log`）最后几行是 `mods/HD2Tracker/watch: loaded`
     —— 若是 `load failed: …` 就会带 Lua 行号，能直接定位问题

进对局后 `playerLog.txt` 里就会出现 `file_start` / `match_start` / `join` / `squad` 等记录。

## 卸载 / 回滚

管理器里**禁用本插件**（要换回旧 tracker 就顺手启用它）→ **Purge & Deploy**。
`playerLog.txt` 是独立文件，删不删都不影响游戏本身。

---

## 输出

| 项目 | 值 |
|---|---|
| 路径 | `%LOCALAPPDATA%\CowboyBingus\Helldivers2\Logs\playerLog.txt` |
| 格式 | 一行一条 JSON（UTF-8 / LF） |
| 写入 | **只追加**（每次打开后 flush）→ 消费方按字节偏移尾随最省资源 |
| 诊断 | 同目录 `playerLog.log`（启动横幅 / 自检 / 扫描统计 / 异常，给人看） |

一个玩家进队、名字补上、再离开，是这样四行（id 与名字都是占位值）：

```json
{"v":2,"mod":"hd2trackerv21.2","ev":"join","capture_id":"1111222233334444","ts":"2026-10-07T14:26:21Z",
 "ts_local":"2026-10-07 22:26:21","game_pid":12345,"game_exe":"D:\\…\\helldivers2.exe","trigger":"join",
 "peer_id_hex":"AABBCCDD00112233","is_self":false,"name":null,"name_src":"unresolved",
 "active_peers":2,"peers":null,"squad":null,"self_name":null,"note":null}

{"v":2,"mod":"hd2trackerv21.2","ev":"squad","capture_id":"1111222233334445","ts":"2026-10-07T14:26:22Z",
 "ts_local":"2026-10-07 22:26:22","game_pid":12345,"game_exe":"D:\\…\\helldivers2.exe","trigger":"roster",
 "peer_id_hex":null,"is_self":false,"name":null,"name_src":"unresolved","active_peers":2,"peers":null,
 "squad":"1122334455667788=PlayerA;AABBCCDD00112233=MyName","self_name":null,"note":"名册：2 人"}

{"v":2,"mod":"hd2trackerv21.2","ev":"update","capture_id":"1111222233334446","ts":"2026-10-07T14:26:24Z",
 "ts_local":"2026-10-07 22:26:24","game_pid":12345,"game_exe":"D:\\…\\helldivers2.exe","trigger":"roster",
 "peer_id_hex":"AABBCCDD00112233","is_self":false,"name":"PlayerA","name_src":"roster",
 "active_peers":2,"peers":null,"squad":null,"self_name":null,"note":"名字来自名册（补）"}

{"v":2,"mod":"hd2trackerv21.2","ev":"leave","capture_id":"1111222233334447","ts":"2026-10-07T14:31:02Z",
 "ts_local":"2026-10-07 22:31:02","game_pid":12345,"game_exe":"D:\\…\\helldivers2.exe","trigger":"left",
 "peer_id_hex":"AABBCCDD00112233","is_self":false,"name":"PlayerA","name_src":"roster",
 "active_peers":1,"peers":null,"squad":null,"self_name":null,"note":null}
```

### 字段（每行 18 个，全部有明确含义）

| 字段 | 类型 | 什么时候有值 | 含义 |
|---|---|---|---|
| `v` | 数字 | 恒有 | JSON 结构版本，恒为 `2` |
| `mod` | 字符串 | 恒有 | 写这行的插件版本，如 `hd2trackerv21.2` |
| `ev` | 字符串 | 恒有 | 事件类型（见下） |
| `capture_id` | 16 位十进制字符串 | 恒有 | **本行唯一 ID**（进程内自增）→ 消费方拿它幂等去重 |
| `ts` / `ts_local` | 字符串 | 恒有 | UTC / 本地时间 |
| `game_pid` | 数字 | 恒有 | 游戏进程 PID。同一个值 = 同一次游戏运行（可当"场次"分组键） |
| `game_exe` | 字符串 | 恒有 | 游戏 exe 完整路径 |
| `trigger` | 字符串 | 恒有 | 这条记录由什么触发（见下） |
| `peer_id_hex` | 16 位大写十六进制字符串 | 与玩家相关的事件 | **玩家标识**：PlayFab `title_player_account`，与游戏 Lua `Network.peer_id()` 同格式。**跨局稳定 ⇒ 拿它当唯一键**；`null` = 本行不对应某个玩家 |
| `is_self` | true/false | 恒有 | 该 peer 是不是你自己 |
| `name` | 字符串 | 名册填好之后 | **玩家名**（游戏里显示的那个，UTF-8，中/日/韩/俄文都能带） |
| `name_src` | 字符串 | 恒有 | `roster` = 来自游戏名册；`unresolved` = 还没拿到 |
| `active_peers` | 数字 | 多数事件 | 队里当前人数（含你自己；独自一人是 1） |
| `peers` | 字符串 | 仅 `match_start` / `match_end` | 队友 id 列表（逗号分隔） |
| `squad` | 字符串 | 仅 `squad` 事件 | 整队快照：`PEERID=名字;PEERID=名字` |
| `self_name` | 字符串 | 仅 `file_start` | 你自己的名字（来自游戏 Lua `Steam.user_name()`） |
| `note` | 字符串 | 部分事件 | 给人看的补充说明 |

空值在 JSON 里就是 `null`（不是空字符串）。

### 事件（`ev`）

| `ev` | 什么时候写 | 关键字段 |
|---|---|---|
| `file_start` | 插件每次加载（= 游戏每次启动） | `self_name` |
| `match_start` / `match_end` | 进入 / 离开一局 | `peers`、`active_peers` |
| `join` | 队里出现一个新玩家（或有人回来了） | `peer_id_hex`、`name`、`is_self` |
| `leave` | 某玩家**连续 2 秒**都不在队里（防抖） | `peer_id_hex`、最后一次已知的 `name` |
| `update` | 同一个人的信息补齐/变化（典型：名字晚几秒才读到） | `peer_id_hex`、新的 `name` |
| `squad` | 名册内容有变化 | `squad` 整队快照 |

`trigger` 取值：`startup`（加载）、`session`（开局/收尾）、`join`、`rejoin`（之前离开又回来）、
`left`、`match_end`、`roster`（名册变化）。

**消费时的三条要点**：

1. `peer_id_hex` = **游戏内的账号级 ID**（PlayFab `title_player_account`，与 `Network.peer_id()` 同值），
   跨局稳定 ⇒ 拿它当唯一键；`name` 只负责显示，别拿名字当键。
2. `join` / `leave` 是**事件**（只有当事人会产生）；`squad` 是**整队快照**（是状态，不要用它触发告警）；
   `update` 是同一个人的信息补齐 → 按 `peer_id_hex` 合并。
3. `join` 先到的时候 `name` 可能还是 `null`，几秒后 `update` 会把名字补上 —— 先显示 id，等补上再刷新。

### 名字是怎么拿到的

- `peer_id_hex`：游戏 Lua API（`Network.peer_id()` / `GameSession.peers()`）直接给；
- `name`：读**游戏自己的名册**（`game.dll` 全局 → 4 槽位 × `0xC0` 字节，`+0`=peer id、`+8`=名字）。
  读取前先校验 `game.dll` 里两段代码签名，**签名不匹配就永久停用名字读取并在日志里写明** ——
  游戏更新后绝不乱读；
- 两处格式一致，所以按 `peer_id_hex` 就能把「名字」和「标识」拼起来 ——
  **不需要 OCR、不需要联网、不需要全内存扫描**。

## 已知限制

| 现象 | 说明 |
|---|---|
| `name` 是 `null` | 名册里 peer id 先出现、名字晚几秒才填好 → 稍后会有 `update` 补上；签名失配时则一直是 `null` |
| 没有 `steam64` | 内存扫描默认关闭。需要 Steam64 得改源码重新打包（源码不随本仓库发布） |
| 没有平台（PC/PS/Xbox） | 需求已搁置 |
| 主机玩家 | 没有 SteamID，但 peer id 与名字照常记录 |
| 快速抖动（<2 秒） | `leave` 有 2 秒防抖，一进一出太快不会写记录 |
| 游戏更新后名字失效 | 名册签名对不上就自动停用名字读取（日志里会写明），peer id 不受影响 |

> **「游戏内 ID」不需要另找**：`peer_id_hex` 就是它。
> 「对局 ID」同理已移除：要按场次分组就用 `game_pid`（同一次游戏运行里恒定）。

## 隐私

- 插件**只写本机文件、不联网**；不含截图、不含聊天内容。
- 日志里包含**其他玩家**的 peer id 与玩家名。**把日志发给别人之前请自行脱敏**
  （本仓库文档与示例里的 id / 名字全部是虚构的占位值）。

## 版本历史

| 版本 | 主要变化 |
|---|---|
| **v21.2** | 去掉内存扫描（不再抓 Steam64）；记录字段收敛为 peer id + 玩家名；事件收敛为上面那 7 种；平台与游戏内 ID 需求搁置（游戏内 ID 就是 peer id） |
| v19.0 | **名字直接从游戏名册读**，不再需要 OCR / 联网；读取前先做代码签名自校验 |
| v17.8 | 修"进局后游戏崩溃"：只调用已验证签名的游戏接口，未知契约的一律不碰 |
| v17.5 | 修"插件根本没加载"（一个 Lua 语法错误）；打包流程加上语法粗检 |
| v17.3 | 修"命中 0"（peer id 是文本，不是原始字节）与"闪退 + 材质闪烁"（限制内存扫描的时间片与全扫次数） |
| v17.2 | 修"一条记录都没有"（JSON 转义器出错）与"扫描慢 100 倍" |
| v17.1 | 修"卡在开始界面"（生成 ID 的循环可能不推进） |
| v17.0 | 首个版本：事件驱动抓取 + 结构化 JSON 输出 + 分级开关 `STAGE` |

## 许可与致谢

本插件的**本体框架不是从零写的**：

| 上游 | 关系 | 许可 |
|---|---|---|
| [360de250/Helldivers2-SteamID-Tracker](https://github.com/360de250/Helldivers2-SteamID-Tracker)（mod 名 `kick`） | **本体框架由它扩张而来**：抓取 / 事件 / 输出三段骨架，以及"写本地日志 + manifest/patch 打包"的做法 | **MIT**，Copyright (c) 2026 360de250 |
| [CowboyBingus/BetterLobbyManagement](https://github.com/CowboyBingus/BetterLobbyManagement)（v1.2） | **受它启发**，并借用它标定的 `game.dll` 名册布局与两段代码签名做自校验 | **0BSD**，Copyright (C) 2026 CowboyBingus |
| Bingus Shared Loader | 加载器（第三方，玩家自装）。**不分发它** | — |

**致谢 360de250 与 CowboyBingus 两位作者。** 没有前者那份 `kick` 的骨架，这个插件不会存在；
没有后者把 `game.dll` 的布局标定出来并在同一个游戏版本上实测过，"直接读游戏名册拿玩家名"这条路也走不通。

本插件以 **MIT** 许可发布（见 [`LICENSE`](LICENSE)）。上游 `kick` 也是 MIT，
该许可要求**保留版权声明与许可正文**，因此把两份上游正文照录于下。

<details>
<summary>上游许可正文：Helldivers2-SteamID-Tracker（MIT）</summary>

```
MIT License

Copyright (c) 2026 360de250

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

</details>

<details>
<summary>上游许可正文：BetterLobbyManagement（0BSD）</summary>

```
Zero-Clause BSD
=============

Copyright (C) 2026 by CowboyBingus

Permission to use, copy, modify, and/or distribute this software for any
purpose with or without fee is hereby granted.

THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH
REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY
AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,
INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM
LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR
OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
PERFORMANCE OF THIS SOFTWARE.
```

</details>

> 0BSD **不要求**署名，我们仍然致谢。另：上游 `kick` 的 README 写明"可随意解包更改优化
> 进一步做出更优秀 mod"，我们正是在这句话的许可下把它扩成了现在这个事件驱动版本。
