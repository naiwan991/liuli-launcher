# 琉璃 · LIULI Launcher

> Minecraft，透过一块玻璃看见。

琉璃是一个以 **Apple Liquid Glass** 为视觉语言的 Minecraft 启动器。它由 **React + TypeScript** 前端和 **Node.js 本地启动内核**组成，并提供浏览器版与 Electron 透明桌面版。

它不要求迁移游戏目录，可以直接识别 HMCL / 官方启动器留下的 `.minecraft`：

- 已安装版本与隔离实例；
- Java 运行时；
- 存档、截图、资源包、光影和模组；
- 原版、Fabric、Quilt、Forge、NeoForge、OptiFine 实例。

---

## 核心特性

### 液态玻璃与真实桌面后景

桌面版使用完全透明的无边框 Electron 窗口，通过「排除自身的屏幕捕获 + CSS `filter: blur()`」实现真实桌面后景：

- Lively 动态壁纸、桌面图标和其他窗口都会透过玻璃显示；
- 背景模糊支持 **0–64 px 连续调节**；
- 玻璃浓度支持 **0–100%**，并保留最低内容可读性底衬；
- 窗口边缘由 CSS 圆角裁切，不使用会露出方形底边的系统亚克力；
- 支持浅色 / 深色玻璃、四种强调色和动效开关；
- 切出琉璃时自动暂停实时捕获，兼容 OBS 和外部截图；
- 多显示器移动后自动重新选择捕获源。

浏览器版无法创建透明窗口，会读取当前 Windows 壁纸作为背景；没有可读壁纸时回退到内置「极光山谷」。

### 实例管理

- 扫描 `versions/`，识别原版、Fabric、Quilt、Forge、NeoForge、OptiFine；
- 通过 `.hmcl`、`options.txt`、`saves` 判断版本隔离；
- 支持实例搜索、排序、筛选、切换、删除；
- 新建实例可选择 Minecraft 版本、加载器、加载器版本，并自动校验/补齐完整安装；
- 实例名重复会提示修改；
- 支持手动最大内存、游戏设置、模组启停、资源包、光影、世界和目录管理。

### 实例合并

创建或首次启动时，琉璃会把 base / loader 的版本 JSON 合并进实例自己的 JSON：

```text
versions/<实例ID>/
  <实例ID>.json       # 已合并，不再 inheritsFrom
  <实例ID>.jar        # 已复制原版 client jar
  .liuli-instance.json
  mods/
  config/
  saves/
```

原版和 Loader 前置版本会写入 `.liuli-dependency.json` 并从实例列表隐藏，只保留一个实例卡。

`libraries` 和 `assets` 仍按 HMCL 的方式共享，不会为每个实例复制数 GB 资源；当前实例不再依赖 base / loader 版本目录即可启动。

已有未合并实例会在启动器启动时自动迁移，不删除原有 mods、config、saves。

### 完整下载与加载器

- 原版安装会校验 JSON、client jar、libraries、assets，缺什么补什么；
- 支持官方源、BMCLAPI 镜像和自动回退；
- 下载中心支持多任务并行，显示当前文件、文件数、字节数、阶段和百分比；
- Fabric / Quilt 通过官方 meta 安装；
- Forge / NeoForge 调用官方 installer；
- NeoForge 元数据和 installer 在官方 Maven 不可达时自动回退 BMCLAPI；
- OptiFine 从 BMCLAPI 获取版本列表和 installer，运行官方：

```text
java -cp OptiFine.jar optifine.Patcher <原版client.jar> <installer.jar> <输出.jar>
```

然后提取 launchwrapper，生成 OptiFine wrapper。

如果某个 Minecraft 版本没有 OptiFine 发布版或预览版，界面不会伪造版本。

### Java 运行时

- 自动扫描本机 Java；
- 按实例要求选择 Java 8 / 17 / 21 / 25；
- 缺少运行时时通过 Azul Zulu API 下载并解压到游戏目录的 `runtime/zulu-<major>`；
- 检测 Java `bin` 下的 `*.addon64`、ReShade、DLSS 注入组件；
- 默认避开被注入的运行时，优先选择干净的同代或更高版本 Java。

例如 Java 21 被 `renodx-dlss5.addon64` 注入时，琉璃会自动改用干净的 Java 25；如果必须使用 Java 21，可以手动删除/重命名该 addon。

### 进程标签与游戏语言

启动游戏时会给 JVM 挂上：

```text
LIULI_INSTANCE_ID
LIULI_INSTANCE_TAG
-Dliuli.instance.id
-Dliuli.instance.tag
```

标签和 PID 会持久化。游戏退出时释放；启动器重启后用 PID 校验并清理失效标签。实例列表据此显示运行状态。

启动前读取 Windows 系统区域或 `LC_ALL` / `LANG`，转换为 Minecraft 语言码并写入实例 `options.txt`。例如 `zh-CN` → `zh_cn`。

### 内容下载

- 浏览 Modrinth 模组、资源包、光影、世界和整合包；
- 按 Minecraft 版本和加载器过滤；
- 点击项目进入详情页，可选择具体文件版本安装；
- 没有匹配实例时可自动创建；
- Modrinth `.mrpack` 支持一键安装并应用 `overrides` / `client-overrides`；
- CurseForge 需要配置 API Key，未配置时只展示 Modrinth。

### 会话日志

每次启动写入：

```text
<数据目录>/logs/session-*.log
```

记录启动、关闭、HTTP/API 请求、导航、下载/安装阶段、实例创建/删除、启动参数、语言、Java 选择、错误与异常。

- 最多保留最近 10 次会话日志；
- 安装 / 启动仍有独立的 `install-*.log` / `launch-*.log`；
- 设置页「打开日志」可直接打开日志目录。

---

## 快速开始

### 环境要求

- Windows 10 / 11；
- Node.js **≥ 22.12**；(桌面端不用)
- 一个可用的 Minecraft 游戏目录；
- Java 运行时（可由琉璃自动准备）。

### 一键启动

双击：

```text
启动琉璃.bat
```

首次运行会安装依赖、构建界面并准备 Electron 运行时（约 250 MB）。

### 手动启动

```powershell
cd "D:\新建文件夹 (2)\liuli-launcher"
npm install
copy .env.example .env
npm run build
npm run desktop
```

如果项目路径包含中文、括号或空格，桌面脚本会把 Electron 运行时复制到 ASCII 缓存目录后再启动，首次可能多等几秒。

浏览器 / 开发模式：

```powershell
npm run dev       # Vite http://127.0.0.1:5178，本地 API http://127.0.0.1:43110
npm start         # 构建后的浏览器版
```

---

## 配置

`.env` 示例：

```dotenv
LIULI_GAME_DIR=E:\hmcl\.minecraft
LIULI_PORT=43110

# 可选：启用 CurseForge 内容来源
# CURSEFORGE_API_KEY=
```

`.env` 已被 Git 忽略。

运行时数据默认位于项目下 `.liuli-data/`：

```text
.liuli-data/
├─ config.json               # 游戏设置、离线账户、启动历史
├─ cache/                    # 版本清单与 Java 缓存
├─ logs/                     # session / install / launch 日志
├─ natives/                  # 每次启动解压的 native 库
└─ instance-process-tags.json
```

---

## 目录结构

```text
liuli-launcher/
├─ electron/
│  ├─ main.mjs               # 透明窗口、内置服务、屏幕捕获
│  └─ preload.cjs            # 窗口控制 IPC
├─ server/
│  ├─ index.mjs              # HTTP 路由与安全限制
│  ├─ discovery.mjs          # 实例 / Java / 世界 / 系统扫描
│  ├─ launcher.mjs           # 启动参数、classpath、native、进程标签
│  ├─ install.mjs            # 原版下载安装与完整性修复
│  ├─ loader-install.mjs     # Fabric / Quilt / Forge / NeoForge / OptiFine
│  ├─ instance-create.mjs    # 新建实例任务
│  ├─ instance-flatten.mjs   # 合并 base / loader、隐藏前置
│  ├─ instance-delete.mjs    # 删除实例与依赖清理
│  ├─ instance-tags.mjs      # 进程标签
│  ├─ game-language.mjs      # 系统语言 → Minecraft 语言
│  ├─ session-log.mjs        # 会话日志与轮转
│  ├─ content.mjs            # Modrinth / CurseForge 内容
│  ├─ modpack-install.mjs    # .mrpack 安装
│  └─ config.mjs             # 本地配置与离线账户
├─ src/
│  ├─ components/            # 玻璃组件、下载中心、内容浏览器
│  ├─ pages/                 # 主页 / 实例 / 下载 / 账户 / 设置 / 关于
│  ├─ overlays/              # 命令面板 / 实例详情 / 启动 / 新建实例
│  ├─ lib/                   # API、格式化、偏好、内容分类
│  └─ styles/                # tokens / glass / components / pages
├─ scripts/
│  └─ desktop.mjs            # 准备 Electron 运行时并启动桌面版
├─ packaging/
│  ├─ build-installer.ps1    # WiX MSI 构建
│  └─ sign.ps1               # exe / MSI 签名
├─ public/media/             # Logo、壁纸、噪声纹理
├─ 启动琉璃.bat
└─ package.json
```

---

## 打包与签名

```powershell
npm run package:win
```

生成：

```text
release/Liuli-Launcher-1.0.0-x64.msi
```

安装范围是当前用户，默认安装到：

```text
%LOCALAPPDATA%\Programs\Liuli Launcher
```

MSI 和 `Liuli Launcher.exe` 使用 `CN=DeepSeek` 自签名代码证书，并附带 RFC3161 时间戳。证书公钥：

```text
release/DeepSeek-CodeSigning.cer
```

这是自签名证书，不是公共 CA 证书：

- 本机导入 `CurrentUser\Root` / `TrustedPublisher` 后验证为 Valid；
- 其他机器需要手动导入 `.cer`；
- 公共 SmartScreen 仍可能提示未知发布者。

---

## 已知限制

- 仅提供离线账户；不实现 Microsoft OAuth / authlib-injector 在线认证，正版服务器需要在线验证时无法用离线账户进入。
- 实时桌面背景依赖屏幕捕获，无法在普通浏览器中实现；网页版只能读取静态壁纸。
- 实时背景开启时，琉璃窗口会从外部截图 / 录屏中排除；需要分享窗口时使用顶部相机按钮，或关闭实时背景。
- 多显示器捕获以窗口所在显示器为准；屏幕捕获上限默认 1920×1080，4K / 高刷屏会增加 GPU/CPU 占用。
- 实例合并只合并版本 JSON 和 client jar；`libraries/`、`assets/` 仍共享，不会为每个实例复制完整资源。
- base / loader 文件夹仍保留在磁盘上供其他实例复用，只是从实例列表隐藏。
- CurseForge 需要 API Key；未配置时只显示 Modrinth。
- Forge / NeoForge / OptiFine 依赖官方或 BMCLAPI 网络；网络受限时可能失败或需要重试。
- OptiFine 只在 BMCLAPI 有对应版本时可用；某些 Minecraft 版本没有发布版或预览版。
- 如果 Java 目录被 ReShade / DLSS 注入，启动器会尽量切换到干净 Java；仍可能因 addon 与具体模组不兼容而崩溃。
- 极老版本（如 1.0）的虚拟资源目录启动兼容性未完全覆盖。
- 会话日志最多保留 10 份；日志为本地明文，可能包含本机路径和项目名。
- 自签名 MSI 在未导入证书的机器上仍会显示未知发布者。

---

## HMCL 参考与说明

琉璃的实例模型、版本隔离、Java 匹配、依赖解析和启动参数构建参考了 HMCL 的公开设计；前端和本地内核均为本项目原创实现，不包含 HMCL 的 Java 源码或 JAR。

OptiFine 安装流程参考 HMCL 的组件方式：

- OptiFine 版本列表；
- installer 下载；
- `optifine.Patcher`；
- OptiFine library；
- launchwrapper；
- wrapper JSON 与 `--tweakClass optifine.OptiFineTweaker`。

琉璃与 HMCL 官方无隶属关系。

---

## 开发命令

```powershell
npm run dev            # Vite + API 热开发
npm run check          # TypeScript 检查
npm run build          # 生产构建
npm start              # 浏览器版
npm run desktop        # 透明桌面版
npm run desktop:build  # 构建后启动透明桌面版
npm run capture        # Playwright 截图（需要本机 Edge）
npm run package:win    # 构建并签名 MSI
```

---

## 许可与致谢

本项目代码使用 **MIT License**。

- [HMCL](https://github.com/HMCL-dev/HMCL) — 实例模型与安装流程参考；
- Apple Liquid Glass — 视觉语言灵感；
- React / Vite / Node.js / Electron / Lucide / adm-zip — 工具链；
- Mojang launchermeta / resources CDN — 官方版本数据；
- Modrinth API — 社区模组、资源包、光影与整合包数据；
- BMCLAPI — 下载镜像与 OptiFine / NeoForge 元数据回退。

Minecraft 是 Mojang Studios 的商标。本项目与 Mojang、Microsoft、HMCL 官方均无隶属关系。
