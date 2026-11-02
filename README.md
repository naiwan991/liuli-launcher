# 琉璃 · LIULI Launcher

> Minecraft，透过一块玻璃看见。

琉璃是一个以 **Apple Liquid Glass** 为视觉语言的 Minecraft 启动器。它用本地 Node 启动内核读取 HMCL / 官方启动器留下的游戏目录，可以直接识别你已经安装的版本、隔离实例、Java 与存档，不要求迁移任何文件。

它同时提供两种形态：**浏览器版**（普通网页）与**桌面透明版**（Electron 无边框透明窗口）。桌面版会实时捕获窗口后方的真实桌面（包括 Lively、桌面图标和其他窗口），并把琉璃自身从捕获中排除，再通过 CSS 模糊实现浏览器版同款的连续模糊滑杆；窗口边缘完全由 CSS 圆角裁切，不再使用会露出方形底边的系统亚克力。

功能骨架参考了 HMCL（分析版本：`HMCL-3.17.0.363.jar`）的实例模型、版本隔离、Java 匹配、依赖解析与启动参数构建；前端和本地内核均为本项目原创的 TypeScript / JavaScript 实现，没有复制 HMCL 的 Java 源代码，也没有分发 HMCL JAR。

---

## 亮点

- **真实桌面后景 + 连续模糊**
  桌面版用屏幕捕获读取窗口后面真正发生的事情：Lively、桌面图标、其他窗口都会出现在玻璃后方。捕获前把琉璃窗口标记为“不参与捕获”，避免递归画面；捕获结果再交给 CSS `filter: blur()`，所以 **背景模糊滑杆真的会从 0 到 64 px 连续改变**。屏幕捕获仍读取整个显示器，但使用 `object-view-box` 只把窗口覆盖的那一块送进窗口渲染，不会把整张桌面缩放进玻璃。捕获上限读取显示器刷新率；窗口移动时通过窗口 bounds 事件 + 每帧几何同步、GPU `translate3d` 平移背景，避免拖动窗口时背景滞后。
- **桌面端边缘**
  不使用 Windows 亚克力、原生圆角或原生阴影，窗口只是一个完全透明的矩形容器；CSS 圆角玻璃面板直接铺满窗口。因此不会再出现方形亚克力底、矩形窗口边、外部留白或裁切台阶。
- **无鼠标光圈、无光标残影**
  移除了所有跟随指针的径向高光层，鼠标移动不会在玻璃上生成光圈或亮斑。桌面版同时隐藏系统光标并使用轻量自绘箭头，避免系统光标被一起捕获到后景画面里。
- **完整的下载工作台**
  下载中心现在包含「新游戏 → 游戏 / 整合包」和「游戏内容 → 模组 / 资源包 / 光影 / 世界」分类。模组、资源包和光影可从 Modrinth 直接安装到当前实例；整合包下载为 `.mrpack`；版本安装可以在官方源、BMCLAPI 镜像与自动回退之间切换。CurseForge 官方 API 需要 Key，配置后即可加入内容来源切换。
- **液态玻璃设计系统**
  多层半透明材质、**0–100% 玻璃浓度**、0–64 px 连续背景模糊、明暗双主题、四种强调色、弹簧式过渡和克制的动效。
- **直接理解你的 HMCL 目录**
  自动扫描 `.minecraft/versions`，识别 Fabric / Forge / NeoForge / Quilt / 原版；通过实例目录里的 `.hmcl`、`options.txt`、`saves` 判断版本隔离，从而正确使用每个实例自己的游戏目录和模组。
- **真正的本地启动内核**
  解析版本 JSON、OS 规则、libraries、natives、assets index、Java 版本，校验依赖并构建 JVM 命令；支持日志、失败提示、启动预检和启动状态。
- **Mojang 版本下载与安装**
  从官方 `launchermeta` 读取 900+ 个版本，下载核心、依赖库、资源索引和资源对象，自动复用已有缓存，校验 SHA-1，支持实时进度、取消与续传。
- **本地优先 / 无遥测**
  服务默认只绑定 `127.0.0.1`，并拒绝非本机 Host / Origin 的 API 请求。账户、设置与日志只保存在项目下的 `.liuli-data`；不会上传游戏目录、账户信息或桌面壁纸。

## 快速开始

环境要求：

- Node.js **≥ 22.12**（当前开发环境为 Node 24）
- Windows 10/11（桌面透明版需要 Windows；浏览器版也可在其他系统运行）
- 一个可用的 Minecraft 游戏目录（默认自动寻找 `%APPDATA%\.minecraft`）
- Java 运行时（琉璃会自动扫描；不同游戏版本需要 Java 8 / 17 / 21 / 25）

最直接的启动方式：双击 **`启动琉璃.bat`**。首次运行会自动安装依赖、构建界面，并打开透明桌面窗口；首次启动桌面版还会准备一份 Electron 运行时（约 250 MB）。

手动启动桌面透明版：

```powershell
cd liuli-launcher
npm install
copy .env.example .env
# 编辑 .env，把 LIULI_GAME_DIR 指向你的 .minecraft
npm run build
npm run desktop
```

浏览器 / 开发模式：

```powershell
npm run dev       # 页面 http://127.0.0.1:5178，本地服务 http://127.0.0.1:43110
npm start         # 构建后的纯浏览器版，打开 http://127.0.0.1:43110
```

`npm run desktop` 会优先复用已经运行的本地服务；如果服务没有启动，会在 Electron 主进程内启动它。桌面版窗口没有系统标题栏，可用顶部栏拖动，右上角提供截图、最小化、最大化和关闭。

### 本机 `.env` 示例

```dotenv
LIULI_GAME_DIR=D:\Games\.minecraft
LIULI_PORT=43110

# 可选，开启 CurseForge 内容来源
# CURSEFORGE_API_KEY=
```

`.env` 已被 Git 忽略，不会提交到仓库。示例见 `.env.example`。

## 功能地图

| 模块 | 已实现 |
| --- | --- |
| 页面 | 主页 / 游戏实例 / 下载中心 / 账户 / 设置 / 关于 |
| 实例 | 扫描、排序、筛选、搜索、当前实例切换、启动诊断；实例详情包含手动最大内存、版本隔离、游戏设置、模组启停/删除、资源包、光影、世界、文件目录管理 |
| 启动 | Java 自动匹配、classpath、资源索引、原版/模组实例、log4j、新旧 native 解压、实时日志 |
| 下载 | Mojang 版本安装、官方 / BMCLAPI / 自动切换；Modrinth 模组、资源包、光影安装；整合包 `.mrpack` 下载 |
| 内容分类 | 游戏 / 整合包 / 模组 / 资源包 / 光影 / 世界；按实例版本与加载器筛选，中文分类菜单 |
| 内容来源 | 全部 / CurseForge / Modrinth；设置 `CURSEFORGE_API_KEY` 后 CurseForge 可参与检索 |
| 账户 | 离线档案、UUID 生成、切换、删除、复制身份 UUID |
| 外观 | 浅/深玻璃、四种强调色、0–100% 玻璃浓度、0–64 px 连续背景模糊、动效开关、桌面留白 |
| 壁纸 | 桌面版实时捕获真实桌面后景（含 Lively）；浏览器版跟随 Windows 壁纸、三套内置壁纸、自定义上传 |
| 桌面壳 | Electron 透明窗口、真实后景屏幕捕获、自身排除、无边框拖拽、窗口控制、窗口截图、单实例本地服务 |
| 系统 | Java 扫描、内存 / 磁盘状态、游戏目录、本地数据路径、全屏 |
| 交互 | 命令面板（`Ctrl/Cmd+K`）、通知面板、启动窗口、安装窗口、Toast |

## 透明的两种实现

### 桌面透明版（推荐）

普通浏览器不能创建透明窗口，而 Windows 的亚克力/云母材质又不能调节模糊半径。琉璃因此采用 **排除自身的实时屏幕捕获 + CSS 模糊**：

1. 创建一个完全透明的无边框 `BrowserWindow`，不使用任何原生背景材质；
2. 在请求屏幕捕获前调用 `setContentProtection(true)`，让琉璃窗口从屏幕捕获结果中排除；
3. 通过 `desktopCapturer` + `getUserMedia` 捕获当前显示器，得到窗口后方的真实桌面；
4. 把捕获画面按屏幕坐标对齐放在玻璃面板下方，再用 CSS `filter: blur()` 提供连续 0–64 px 模糊；
5. 窗口本身保持透明，CSS 圆角、边框和透明四角直接构成窗口外观，因此没有方形亚克力底；
6. 关闭“实时桌面背景”后停止捕获并恢复静态壁纸，方便截图和录屏。

这会让玻璃后面出现的确实是 **窗口后方的桌面与 Lively**，不是视频副本；捕获延迟通常只有一帧左右。代价是：实时背景开启时，琉璃窗口会从其他应用的屏幕截图/录屏中排除。需要分享琉璃画面时，用顶部栏的相机按钮保存窗口截图；或者在设置里暂时关闭“实时桌面背景”。

### 浏览器版：桌面壁纸桥接

如果直接运行 `npm start` 或在 `npm run dev` 中打开网页，浏览器仍然是不透明窗口。此时琉璃会读取 Windows 当前桌面壁纸文件（`TranscodedWallpaper`）作为底层画面，再用玻璃层透出它。它能看到壁纸，但无法显示 Lively 的动态画面，也无法透出桌面图标与其他窗口。

如果系统没有可读取的壁纸，浏览器版会自动回退到内置的「极光山谷」。

## 目录结构

```text
liuli-launcher/
├─ electron/                 # 透明桌面壳
│  ├─ main.mjs               # BrowserWindow、内置服务、窗口控制
│  └─ preload.cjs            # 安全的窗口控制 / 状态 IPC 桥
├─ server/                   # 本地 Node 服务（无框架）
│  ├─ index.mjs              # HTTP 路由、安全限制、静态文件
│  ├─ discovery.mjs          # 实例/世界/截图/Java/系统/壁纸扫描
│  ├─ launcher.mjs           # 版本 JSON、规则、native、启动任务
│  ├─ install.mjs            # Mojang 版本下载安装、官方/镜像回退
│  ├─ content.mjs            # Modrinth/CurseForge 内容检索与安装
│  ├─ remote.mjs             # 版本清单缓存
│  └─ config.mjs             # 本地配置与离线账户
├─ src/
│  ├─ components/            # 玻璃组件、内容浏览器、像素头像、体素插画
│  ├─ pages/                 # 六个主页面
│  ├─ overlays/              # 命令面板 / 实例详情 / 启动 / 安装 / 添加账户
│  ├─ lib/                   # API、类型格式化、偏好、演示数据、内容分类
│  └─ styles/                # tokens / glass / components / pages / overlays
├─ public/media/             # Logo、噪声纹理、三套 SVG 壁纸
├─ scripts/                  # 桌面启动器、运行时准备、壁纸生成、截图
├─ .env.example
└─ package.json
```

运行时数据（默认 `.liuli-data/`）：

```text
.liuli-data/
├─ config.json               # 游戏设置、离线账户、启动历史
├─ cache/                    # Mojang 版本清单与版本详情缓存
├─ logs/                     # 启动日志、安装日志
└─ natives/                  # 每次启动解压的 native 库
```

## HMCL 参考与差异

参考的部分：

- 多版本实例模型与“版本隔离”语义；
- 通过 `.hmcl` / 游戏数据目录识别独立版本；
- Java 运行时发现与版本要求匹配；
- 从版本 JSON 构建 JVM/游戏参数、classpath 与 native 目录；
- 官方版本清单与资源 CDN 的下载路径。

为适配网页形态而修改 / 未实现的部分：

- 使用浏览器 / Electron + 本地 HTTP 服务替代 JavaFX；
- 仅提供离线档案，不实现 Microsoft OAuth / authlib-injector 在线认证；
- 原版安装与 Modrinth 内容安装已接入；Fabric / Forge / NeoForge 加载器本身仍建议用 HMCL 安装，整合包下载为 `.mrpack` 后交由 HMCL 导入；
- CurseForge 官方 API 需要 Key，未配置时只展示 Modrinth 结果；不会内置或借用他人的 Key；
- 不做 HMCL 云同步、崩溃分析、在线账户等完整 HMCL 功能。

琉璃与 HMCL 官方无隶属关系，Minecraft 是 Mojang Studios 的商标。

## 已知限制

- 启动依赖本机已有 Java 和完整依赖；缺失依赖时会给出提示，请先用 HMCL 补全。
- 网页版不能创建透明窗口；真正的 Lively 动态透出请使用 `npm run desktop` / `启动琉璃.bat`。
- 实时桌面背景使用屏幕捕获，并读取当前显示器的刷新率作为捕获上限（例如 60Hz / 144Hz / 165Hz）；窗口拖动时背景按同一刷新率跟随窗口。默认最多捕获 1920×1080；在 4K / 高刷屏上会有相应 GPU/CPU 占用。
- 如果显示器刷新率没有被 Electron 正确报告，会回退到 60Hz；可以在设置页的“检测到 N Hz”提示中确认当前值。
- 绘制桌面光标时使用自绘箭头，避免系统光标被屏幕捕获进背景；如果不需要自绘光标，可以关闭实时桌面背景。
- **实时背景开启时，系统截图和录屏不会包含琉璃窗口本身**（这是排除自身、避免递归捕获的必要条件）。顶部栏相机按钮可以保存包含实时背景的窗口截图；设置中关闭“实时桌面背景”则会回退到静态壁纸，方便外部截图。
- 屏幕捕获失败或用户关闭实时背景时，会显示 Windows 静态壁纸并保留模糊滑杆，不会出现黑屏。
- 在多显示器环境中默认捕获琉璃窗口所在显示器；移动窗口后捕获源不会自动切换，重新开关实时背景即可。
- 实例的最大内存与版本隔离保存在实例目录的 `.liuli-instance.json` 中；版本隔离只改变后续使用的游戏目录，不会自动迁移已有的世界、模组或资源包。
- 屏幕捕获 API 只能读取整块显示器，因此“只渲染窗口区域”发生在窗口内的渲染阶段（`object-view-box` 裁剪），而不是捕获源本身；这样不会把整张桌面缩放进窗口显示。
- 若项目路径包含中文或括号，桌面启动脚本会把 Electron 运行时复制到 ASCII 缓存目录再运行，首次启动会慢几秒。
- 仅提供离线档案；正版服务器需要在线验证时无法用离线档案进入。
- 极老版本（如 1.0）的虚拟资源目录尚未完全处理，下载可完成，启动兼容性未覆盖。
- 安装/启动会写入你选择的游戏目录；请确认服务以有权限的用户身份运行。
- CurseForge 内容来源需要 `.env` 中的 `CURSEFORGE_API_KEY`；未配置时「全部」只返回 Modrinth 结果。

## 开发命令

```powershell
npm run dev            # Vite + API 热开发
npm run check          # TypeScript 检查
npm run build          # 生产构建
npm start              # 浏览器版：服务 dist + API
npm run desktop        # 透明桌面版（自动准备 Electron 运行时）
npm run desktop:build  # 构建后启动透明桌面版
npm run capture        # Playwright 截图（需要本机 Edge）
```

## 许可与致谢

本项目代码使用 **MIT License**。

- [HMCL · Hello Minecraft! Launcher](https://github.com/HMCL-dev/HMCL) — 功能地图、版本模型与启动思路参考；GPL-3.0。
- [Apple Liquid Glass](https://www.apple.com/) — 视觉语言灵感。
- React / Vite / Node.js / Electron / Lucide / adm-zip — 开源工具链。
- Mojang launchermeta / resources CDN — 官方版本数据。
- Modrinth API — 社区模组、资源包、光影与整合包数据；CurseForge 为其独立服务。

没有 HMCL、Minecraft 的代码、二进制或官方素材被打包进本仓库。
