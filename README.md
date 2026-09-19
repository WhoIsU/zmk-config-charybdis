# zmk-config-charybdis

Charybdis 分体键盘（PMW3610 轨迹球）的 ZMK 配置。

基于 **zmkfirmware/zmk `main`** + [badjeff/zmk-pmw3610-driver](https://github.com/badjeff/zmk-pmw3610-driver)，只维护本 config 仓库，不 fork ZMK。

## 层序

`0 Base · 1 MOUSE · 2 SYMB · 3 NAV · 4 SYS · 5 SNIPE · 6 SCROLL`

## 轨迹球

| 模式 | 触发方式 | 实现 |
| --- | --- | --- |
| 普通移动 | 滚球即自动进入 MOUSE 层 | `&zip_temp_layer 1 750` |
| 狙击（降速） | 按住右手小指 `'` | 层 5 + `&zip_xy_scaler 1 3` |
| 滚轮 | 按住右手小指 `\` | 层 6 + scaler + `&zip_xy_to_scroll_mapper` |

三者都在 `config/boards/shields/charybdis/charybdis_right.overlay` 的 `trackball_listener` 里。停止滚球后自动鼠标层还会保持 750ms（改 `zip_temp_layer` 的第二个参数）。

传感器朝向对应旧 inorichi 的 `ORIENTATION_90` + `INVERT_X`：`swap-xy` + `invert-x` + `invert-y`。普通移动调 `cpi`（当前 1200）；滚轮调 scroll 里的 `&zip_xy_scaler 1 N`（N 越大越慢，当前 96）；狙击仍是相对普通速度的 `1/3`。

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

其余键都是 `&trans`，会穿透到 Base 层，所以 Shift+点击（左拇指）和 Alt+点击（长按 `/`）照常可用。但 `;` 被占用成了右键，而 Base 层的 Ctrl 只存在于 `&mt LCTRL SEMI`，所以**鼠标层没有 Ctrl，Ctrl+点击用不了**；想要的话把 `MS_HOME_R` 里 `;` 那一位改回 `&trans`。

另外 `J` 是常用字母：滚球后的超时窗口内敲 `j` 会变成左键。误触明显的话就调小上面那个超时。

## 改 keymap 时注意

`.keymap` 会先过 C 预处理器。注释行末尾不要出现 `\`，否则它会被当成续行符，把下一行的 `#define` 整行吞进注释——展开后宏名会原样留在 keymap 里导致编译失败。

## 布局图

自动生成流程已停用，仓库里不再保留生成结果（旧的 svg/yaml 快照容易和 keymap 脱节）。需要出图时把 `config/charybdis.keymap` 喂给 [caksoylar/keymap-drawer](https://github.com/caksoylar/keymap-drawer)，样式配置仍保留在 `keymap_drawer.config.yaml`。
