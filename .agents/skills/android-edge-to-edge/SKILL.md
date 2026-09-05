---
name: android-edge-to-edge
description: "Android 全面屏 edge-to-edge 沉浸式适配专家，专治 HyperOS/MIUI 手势条白条。触发词：状态栏沉浸、导航栏沉浸、全面屏适配、手势条/小白条颜色、isNavigationBarContrastEnforced、enableEdgeToEdge、SystemBarStyle、windowBackground 透明、底栏/NavigationBar 不延伸到导航栏后面、手势区颜色不一致、Compose insets 失效。适用于 Jetpack Compose（Material 3）与原生 View 项目的系统栏适配、MIUI 官方文档原则落地、像素级验证方法。"
---

# Android Edge-to-Edge 沉浸式适配（含 HyperOS/MIUI 专项）

## 核心原则（来源：MIUI 官方文档 dev.mi.com/xiaomihyperos pId=1629）

> 适配原则：“手势提示线”的背景颜色和**页面整体的背景颜色保持一致**。

手势提示线（小白条）与虚拟按键在同一个 window 中。两种官方适配方式：

1. **沉浸式虚拟键**：content view 延伸到虚拟键区域，虚拟键颜色透明。适合地图/图片等全屏内容。
2. **给虚拟键设置合适颜色**：`setNavigationBarColor` 与底部 tab 色一致。content view 不延伸。

Compose + Material 3 场景必须用方式 1（M3 已在 targetSdk 35+ 强制 edge-to-edge）。

## 三层结构模型（排查问题的思维框架）

系统栏区域从下到上有三层，任何一层颜色不对都会出问题：

| 层 | 内容 | 谁控制 |
|---|---|---|
| L1 window 背景 | DecorView 之下的底色 | 主题 `android:windowBackground` |
| L2 系统栏保护层 | MIUI/HyperOS 对比度 scrim | `window.isNavigationBarContrastEnforced` |
| L3 应用内容 | Compose/View 绘制 | 布局是否延伸 + 底栏背景色 |

**诊断口诀：手势区颜色 = 哪一层最后画在那里。** L1 需透明让 L3 透出；L2 必须显式关闭；L3 必须真的延伸（insets 数据正确 + 底栏背景覆盖）。

## 标准实现（Jetpack Compose + M3，已在 HyperOS 3.0 / Android 15 真机验证）

### 1. Activity 层（必须项）

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // 导航栏 surface 由 Compose 自己绘制，窗口层必须完全透明，
        // 否则默认 scrim 会让小白条区域出现额外底色。
        enableEdgeToEdge(
            statusBarStyle = SystemBarStyle.auto(Color.TRANSPARENT, Color.TRANSPARENT),
            navigationBarStyle = SystemBarStyle.auto(Color.TRANSPARENT, Color.TRANSPARENT),
        )
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            // HyperOS/MIUI 会在透明导航栏上强制叠加对比度保护层，必须显式关闭。
            window.isNavigationBarContrastEnforced = false
        }
        setContent { /* ... */ }
    }
}
```

**`isNavigationBarContrastEnforced = false` 是 MIUI 白条问题的决定性修复**，普通 AOSP 设备上此项默认行为无感，极易遗漏。

### 2. 主题层（themes.xml）

```xml
<style name="Theme.App" parent="android:style/Theme.Material.Light.NoActionBar">
    <item name="android:windowBackground">@android:color/transparent</item>
    <item name="android:statusBarColor">@android:color/transparent</item>
    <item name="android:navigationBarColor">@android:color/transparent</item>
    <item name="android:windowLightStatusBar">true</item>
    <item name="android:windowLightNavigationBar" tools:targetApi="o_mr1">true</item>
</style>
```

- `windowBackground` 必须透明（L1 让位），否则手势区透出主题默认底色。
- 深色模式用 `values-night/themes.xml` 提供 `windowLightStatusBar=false` 变体，图标颜色跟随主题。

### 3. Compose 版本要求（关键坑）

**compose-bom 必须 ≥ 2025.04.01（Compose UI 1.7+）。**

Compose UI 1.6.x（bom 2024.04.01）在 targetSdk 35+ 强制 edge-to-edge 下，insets 桥接有缺陷：`WindowInsets.navigationBars` 读到 0，导致：
- `NavigationBar` 的 `windowInsetsPadding` 不生效，底栏不按 inset 垫高
- 手势区透出 Scaffold 的 `surface` 背景，与底栏 `surfaceContainer` 形成色差

症状识别：底栏色正确但只画到 inset 边界，其下与手势条之间出现独立色带。

### 4. Compose 布局层

```kotlin
Scaffold(
    bottomBar = { NavigationBar { /* tabs */ } },
) { padding ->
    // content 用 padding 避让底栏
}
```

- M3 `NavigationBar` 自带 `windowInsetsPadding(NavigationBarDefaults.windowInsets)`，会自动垫高并把手势区画成底栏色——**前提是 insets 数据正确**（见第 3 节）。
- 全屏内容（地图/图片）避让状态栏用 `statusBarsPadding()`，控件贴边用 `navigationBarsPadding()`，**不要**给根布局整体加 padding。
- 不要设置 `Scaffold(contentWindowInsets = WindowInsets(0))` 除非明确知道后果（content 不再自动避让）。

## 像素级验证方法（无 PIL 依赖）

真机截图后用纯 zlib+struct 解 PNG IDAT，逐行对比底栏色与手势区色：

```python
import struct, zlib

def read_png(path):
    with open(path, 'rb') as f: data = f.read()
    pos = 8; idat = b''
    while pos < len(data):
        ln = struct.unpack('>I', data[pos:pos+4])[0]
        ct = data[pos+4:pos+8]; ch = data[pos+8:pos+8+ln]
        if ct == b'IHDR': w, h = struct.unpack('>II', ch[:8])
        elif ct == b'IDAT': idat += ch
        elif ct == b'IEND': break
        pos += 12 + ln
    raw = zlib.decompress(idat)
    out = bytearray(); prev = bytearray(w * 4); ptr = 0
    for y in range(h):
        ft = raw[ptr]; ptr += 1
        line = bytearray(raw[ptr:ptr + w * 4]); ptr += w * 4
        if ft == 1:
            for i in range(4, len(line)): line[i] = (line[i] + line[i-4]) & 0xFF
        elif ft == 2:
            for i in range(len(line)): line[i] = (line[i] + prev[i]) & 0xFF
        elif ft == 3:
            for i in range(len(line)):
                left = line[i-4] if i >= 4 else 0
                line[i] = (line[i] + ((left + prev[i]) >> 1)) & 0xFF
        elif ft == 4:
            for i in range(len(line)):
                a = line[i-4] if i >= 4 else 0; b = prev[i]
                c = prev[i-4] if i >= 4 else 0
                p = a + b - c
                pa, pb, pc = abs(p-a), abs(p-b), abs(p-c)
                pr = a if (pa <= pb and pa <= pc) else (b if pb <= pc else c)
                line[i] = (line[i] + pr) & 0xFF
        prev = line; out += line
    return w, h, out

w, h, px = read_png('screen.png')
# 对比底栏区与手势区（手势 inset 一般 12-24dp，乘 density）
for y in [2300, 2370, 2395]:
    off = (y * w + w // 2) * 4
    print(y, tuple(px[off:off+3]))
```

**通过标准：底栏上沿到屏幕最底一行，颜色完全一致。**

辅助取证命令：

```bash
# 窗口是否真的铺满（frame 应为 [0,0][w,h]）
adb shell dumpsys window <pkg> | grep -E "mFrame=|frame="

# 手势条 inset 高度（TaskSnapshot 的 mContentInsets）
adb shell dumpsys window | grep mContentInsets

# MIUI 是否强制对比度层（无直接开关可查，靠像素验证）
adb exec-out screencap -p > /tmp/check.png
```

## 常见症状 → 根因速查

| 症状 | 根因 | 修复 |
|---|---|---|
| 手势区白条/色带，其余正常 | MIUI 对比度保护层 | `isNavigationBarContrastEnforced = false` |
| 底栏不垫高，内容被手势条遮挡 | Compose 1.6 insets 桥接缺陷 | compose-bom ≥ 2025.04.01 |
| 手势区显示主题默认底色 | `windowBackground` 不透明 | 主题加 transparent windowBackground |
| 状态栏区域黑条/白条 | 未调用 `enableEdgeToEdge` 或 statusBarColor 非透明 | 显式 `SystemBarStyle.auto(TRANSPARENT, TRANSPARENT)` |
| 深色模式状态栏图标看不清 | 缺 values-night 的 lightStatusBar 变体 | 深色主题 `windowLightStatusBar=false` |
| 内容延伸后正文被状态栏遮挡 | 布局未避让 | 内容区 `statusBarsPadding()`，地图/图片类不避让 |

## 排查工作流

1. 截图 → 像素分析定位是哪一层的颜色（对照 L1/L2/L3）
2. L1 色 → 检查 windowBackground；L2 色 → 关 contrast enforced；L3 缺失 → 检查 insets 桥接与 Compose 版本
3. 每次只改一层，rebuild + 重装 + 同一采样点复测
4. 对比参考应用（同设备安装一个已正确适配的应用截图对比）能快速锁定差异层

## 参考实现来源

- MIUI 官方：《全面屏手势提示线适配说明》 https://dev.mi.com/xiaomihyperos/documentation/detail?pId=1629
- BiliRoaming 系模块的 `enableEdgeToEdge` + `isNavigationBarContrastEnforced` 写法
- Android 官方 edge-to-edge 指南（targetSdk 35+ 强制行为）
