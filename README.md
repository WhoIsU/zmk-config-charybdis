# zmk-config-charybdis

Charybdis 分体键盘（PMW3610 轨迹球）的 ZMK 配置。

## 层序

`0 Base · 1 MOUSE · 2 SYMB · 3 NAV · 4 SYS · 5 SNIPE · 6 SCROLL`

PMW3610 只看当前**最高**激活层，所以自动鼠标层（1）必须低于狙击（5）和滚轮（6）。层序一旦改错，按住 `'` 再滚球就不会降速。

## 轨迹球

| 模式 | 触发方式 | 定义位置 |
| --- | --- | --- |
| 普通移动 | 滚球即自动进入 MOUSE 层 | `automouse-layer = <1>` |
| 狙击（降 CPI） | 按住右手小指 `'` | `snipe-layers = <5>` |
| 滚轮 | 按住右手小指 `\` | `scroll-layers = <6>` |

三者都在 `config/boards/shields/charybdis/charybdis_right.overlay` 里。停止滚球后自动鼠标层还会保持 `CONFIG_PMW3610_AUTOMOUSE_TIMEOUT_MS` 毫秒（见 `charybdis_right.conf`，当前 750）。

这个超时是唯一的调节点——早先 `Kconfig.defconfig` 里有个 `MOUSE_LAYER_ACTIVE_MS=400`，但 inorichi 的驱动根本不读它，已删除。

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
