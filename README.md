# zmk-config-charybdis

Charybdis 分体键盘（PMW3610 轨迹球）的 ZMK 配置。

基于 **zmkfirmware/zmk `main`** + [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)，只维护本 config 仓库，不 fork ZMK。

## 层序

`0 Base · 1 MOUSE · 2 SYMB · 3 NAV · 4 SYS · 5 SNIPE · 6 SCROLL · 7 SHORT`

## 轨迹球

| 模式 | 触发方式 | 实现 |
| --- | --- | --- |
| 普通移动 | 滚球即自动进入 MOUSE 层 | `&zip_temp_layer 1 1200` |
| 狙击（降速） | 按住右手小指 `'` | 层 5 + `&zip_xy_scaler 1 3` |
| 滚轮 | 按住右手小指 `\` | 层 6 + scaler + `&zip_xy_to_scroll_mapper` |

三者都在 `config/boards/shields/charybdis/charybdis_right.overlay` 的 `trackball_listener` 里。停止滚球后自动鼠标层还会保持 1200ms（改 `zip_temp_layer` 的第二个参数）。

传感器朝向对应旧 inorichi 的 `ORIENTATION_90` + `INVERT_X`：`swap-xy` + `invert-x` + `invert-y`。普通移动调 `cpi`（当前 1000）；滚轮调 scroll 里的 `&zip_xy_scaler 1 N`（N 越大越慢，当前 96），上下方向用 `zip_scroll_transform (INPUT_TRANSFORM_Y_INVERT)` 翻转；狙击仍是相对普通速度的 `1/3`。

## 鼠标键

MOUSE / SNIPE / SCROLL 三层共用同一套按键，定义在 `config/charybdis.keymap` 顶部的 `MS_HOME_R` / `MS_BOTTOM_R` 宏里，改一处三层同时生效。

| 键位 | 手指 | 功能 |
| --- | --- | --- |
| `J` | 右食指 | 左键 |
| `K` | 右中指 | 中键 |
| `L` | 右无名指 | 右键 |
| `;` | 右小指 | 右键 |
| `,` | 右中指（下排） | 左键 |
| `.` | 右无名指（下排） | 中键 |

球在右拇指下，按键在食指/中指/无名指下，等于把右手变成一只鼠标。

其余键都是 `&trans`，会穿透到 Base 层，所以 Shift+点击（左拇指）和 Alt+点击（先按住 `/` 超过 200ms）照常可用。但 `;` 被占用成了右键，而 Base 层的 Ctrl 只存在于 `&hmr LCTRL SEMI`，所以**鼠标层没有 Ctrl，Ctrl+点击用不了**；想要的话把 `MS_HOME_R` 里 `;` 那一位改回 `&trans`。

另外 `J` 是常用字母，自动鼠标层为此加了两道保护（`charybdis_right.overlay` 里的 `&zip_temp_layer`）：

- `require-prior-idle-ms = <300>`：打完字 300ms 内碰到球不会进鼠标层。
- `excluded-positions`：鼠标层里按下列表外的任何键会立刻退出鼠标层，回到打字。列表里是鼠标键、`'`/`\`、Shift、Alt；以后往鼠标层加键要同步加进这个列表。

## 点按 / 长按判定

| 键 | 点按 | 长按 | 行为 |
| --- | --- | --- | --- |
| `;` | `;` | Ctrl | `&hmr` |
| `/` | `/`；按住 Shift 时直接输出 `?` | Alt（未先按 Shift 时） | `&slash_alt` → `&hmr` |
| 左拇指 Space | 空格 | NAV | `&lt_spc` |
| 左拇指 Shift | Shift（双击 = `caps_word`） | Shift | `&td_shift` |
| 左拇指 MO | 一次性 SHORT 层 | SYMB | `&mo_sl` |

`&hmr` 按位置判定：按住 `;` / `/` 时按下**左手或拇指**的键，立刻算 Ctrl / Alt；按下**右手**的键算点按。需要配右手键时（Ctrl+L、Alt+点击、Ctrl+滚轮），先按住超过 200ms 再按。刚连续打字（150ms 内）按下时直接当点按。

`/` 外层的 `&slash_alt` 会先检查左右 Shift：已经按住 Shift 时，直接发送保留 Shift 的普通 `/` 按键，不进入长按判定，因此连点或按住 `/` 都不会从这颗键误发 Alt。这样可避免原先 `/` 按住超过 200ms 后变成 Alt、问号丢失并可能触发 Windows 的 Alt+Shift 输入语言切换。中文模式下最终显示 `？` 还是 `?` 由输入法标点设置决定。需要 Alt+Shift 组合时，先按住 `/` 超过 200ms，再按 Shift。

`&td_shift` 单点仍发 Shift，输入法的 Shift 切换中英文照常可用，只是会晚约 200ms 发出。双击进入 `caps_word`：下一个单词全大写，遇到空格或标点自动退出。

## 快捷键层 SHORT

单点左拇指 MO 进入，按一颗键后自动回 Base；1.5 秒内不按键自动退出（`&sl` 的 `release-after-ms`），点错时按 Esc 取消。按住 MO 仍是 SYMB（`hold-preferred`：按住期间按了别的键就一定是 SYMB）。

键位按字母联想摆放，详见 `docs/base-layer.md`。

## 刷机

1. USB 接上要刷的那一半。
2. 按住 Esc 进 SYS，按**同一半**上的 BOOT（左手 X 位 / 右手 `.` 位）；两半连不上时双击该半的物理复位键。
3. 把对应的 `.uf2`（文件名带 `left` / `right`）拖进出现的 U 盘。

只改 keymap 时只刷右半（central）；改了 `charybdis.dtsi`、`config/charybdis.conf` 或 `charybdis_left.*` 时两半都刷。配对或两半互连异常时，先给两半刷 `settings_reset`，再刷回正常固件并重新配对。

## 省电

| 状态 | 左半 | 右半 | 唤醒 |
| --- | --- | --- | --- |
| 空闲 idle | 15 秒无操作 | 30 秒（默认） | 任意按键或滚球，立即恢复 |
| 深度睡眠 | 1 小时无操作 | 1 小时无操作 | 按**该半**任意键；芯片重启并重连蓝牙（约 1–3 秒），唤醒键本身不输出 |

- 深睡配置在 `config/charybdis.conf`（两半共用），idle 在 `charybdis_left.conf`；按键唤醒靠 `charybdis.dtsi` 里 `kscan0` 的 `wakeup-source`。
- 滚球唤醒不了深睡的右半。两半都睡着时先按右半的键（右半是 central，左半的按键要经它转发）。
- 插 USB 供电时不会进入深睡。
- 右半 idle 不设太短：进入 idle 后传感器退出 `force-awake`，第一下滚球会发涩。

## 改 keymap 时注意

`.keymap` 会先过 C 预处理器。注释行末尾不要出现 `\`，否则它会被当成续行符，把下一行的 `#define` 整行吞进注释——展开后宏名会原样留在 keymap 里导致编译失败。

## 布局图

自动生成流程已停用，仓库里不再保留生成结果（旧的 svg/yaml 快照容易和 keymap 脱节）。需要出图时把 `config/charybdis.keymap` 喂给 [caksoylar/keymap-drawer](https://github.com/caksoylar/keymap-drawer)，样式配置仍保留在 `keymap_drawer.config.yaml`。
