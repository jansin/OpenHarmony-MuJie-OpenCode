# OHOS-OpenCode

OpenCode 的 OpenHarmony 版本（工程代号 `OhosOpenCode`）。

| 项目 | 值 |
| --- | --- |
| bundleName | `com.jason.opencode` |
| versionName / versionCode | `1.0.0` / `1000000` |
| 应用名称 | `OhosOpenCode`（`AppScope/.../string.json` 的 `app_name`） |
| 开发语言 | ArkTS / ArkUI（含 C++ NAPI 原生模块） |
| 目标平台 | OpenHarmony（`runtimeOS: "OpenHarmony"`），Stage 模型，`deviceTypes: ["default"]`，SDK 12 |
| 仓库地址 | https://github.com/jansin/OpenHarmony-MuJie-OpenCode |
| 开源协议 | Apache License 2.0 |

## 一、项目简介与当前状态

本仓库是 **OpenCode** 在 **OpenHarmony** 平台上的移植/客户端工程（包名 `com.jason.opencode`）。OpenCode 上游是面向开发者的 AI 编码工具，但仓库内未见其源码或构建产物（见[第七章](#七第三方依赖与开源许可)）。

> **截至本文撰写时，本仓库内的代码是 DevEco Studio 默认工程模板，尚未包含 OpenCode 的任何实际业务实现。**

判定依据（均为仓库内可复核的事实）：

- `entry/src/main/ets/` 下**只有 1 个页面** `pages/Index.ets`，`main_pages.json` 也只注册了 `pages/Index`。
- 该页面是模板自带的 `Hello World`，点击后改为 `Welcome`，并调用一次 NAPI 函数打印日志。
- **不存在 `entry/src/main/resources/rawfile` 目录**，即没有随仓库分发的任何 Web 前端（HTML/JS/CSS）构建产物。
- 全工程**无 `Web`/`WebviewController` 组件，也无任何网络代码**（`@ohos.net.http`、`@ohos.net.webSocket`、`rcp`）。
- `module.json5` **未声明任何 `requestPermissions`**（零权限）。
- `entry/src/main/cpp/napi_init.cpp` 仅一个 `add(a, b)` 示例函数；`entry/oh-package.json5` 的 `dependencies` 只引用本地 NAPI 库 `libentry.so`。

故**实现方式尚未定型**：非 WebView 包壳，非原生重写，也无"本地服务 + Web 前端"组合。

## 二、功能特性（路径基准 `entry/src/`）

| 特性 | 位置 |
| --- | --- |
| Stage 模型入口 `UIAbility`，颜色模式跟随系统 | `main/ets/entryability/EntryAbility.ets` |
| 唯一页面：显示 `Hello World`，点击变 `Welcome` 并调用 NAPI `add(2, 3)` 打日志 | `main/ets/pages/Index.ets` |
| C++ NAPI 模块 `libentry.so`（`add(a, b)`） | `main/cpp/napi_init.cpp`、`main/cpp/types/libentry/` |
| 备份/恢复扩展 `BackupExtensionAbility` | `main/ets/entrybackupability/EntryBackupAbility.ets` |
| 深浅色启动窗口背景色 | `main/resources/{base,dark}/element/color.json` |
| 原生模块 Mock、单测与 ohosTest 骨架 | `mock/`、`test/`、`ohosTest/` |

> **OpenCode 相关功能特性（待补充）**：仓库内尚无 OpenCode 业务代码，本地服务启动、会话管理、代码补全/对话、Web 前端加载等能力均无法确认。

## 三、技术栈

| 技术 | 说明 |
| --- | --- |
| ArkTS / ArkUI | `@Entry`/`@Component`/`@State` 与 `Row`、`Column`、`Text` 组件 |
| Stage 模型 Ability | `UIAbility` 生命周期回调；`windowStage.loadContent('pages/Index', ...)` 加载页面 |
| HarmonyOS Kit | `@kit.AbilityKit`、`@kit.ArkUI`、`@kit.PerformanceAnalysisKit`（`hilog`）、`@kit.CoreFileKit` |
| 构建与依赖 | hvigor `modelVersion 6.1.0`（`appTasks`／`hapTasks`）、`nativeCompiler: "BiSheng"`、ABI `arm64-v8a`/`x86_64`、C++ 侧 `CMakeLists.txt` 链接 `libace_napi.z.so` |

> **Web 组件 / 本地服务 / 其他三方库**：仓库内均未使用。

## 四、目录结构（关键部分）

```
OpenHarmony-MuJie-OpenCode/
├── AppScope/                       # 应用级配置与图标资源
├── entry/                          # 唯一的 HAP 模块
│   ├── build-profile.json5         # stageMode、CMake、ABI、release 混淆
│   ├── oh-package.json5            # 依赖本地 NAPI 库 libentry.so
│   └── src/
│       ├── main/module.json5       # EntryAbility / EntryBackupAbility / pages 入口
│       ├── main/cpp/               # napi_init.cpp、CMakeLists.txt、types/libentry/
│       ├── main/ets/               # entryability/、entrybackupability/、pages/Index.ets
│       ├── main/resources/         # base/{element,media,profile}、dark/element/
│       └── mock/、ohosTest/、test/ # Mock、设备侧测试、本地单测
├── hvigor/hvigor-config.json5、build-profile.json5   # 构建行为与产品/签名配置
├── code-linter.json5、hvigorfile.ts、oh-package.json5
├── LICENSE                         # Apache License 2.0 全文
└── README.md
```

> 注意：仓库内**没有** `hvigorw` / `hvigorw.bat` 包装脚本，也**没有** `entry/src/main/resources/rawfile` 目录。

## 五、环境要求、构建与运行

| 项 | 要求 |
| --- | --- |
| DevEco Studio | 支持 hvigor `modelVersion 6.1.0`、`compileSdkVersion 12`（最低版本号**待补充**） |
| SDK | OpenHarmony SDK **API 12**（compile / compatible / target 均为 12） |
| 工具链 / Node.js | CMake ≥ 3.5.0、C++ 编译器 `BiSheng`；ABI `arm64-v8a`（真机）、`x86_64`（模拟器）；Node.js 与 ohpm 由 DevEco Studio 自带（版本号**待补充**），源 `https://ohpm.openharmony.cn/ohpm/` |

**获取与同步**：`git clone git@github.com:jansin/OpenHarmony-MuJie-OpenCode.git`，用 DevEco Studio **Open** 工程根目录并等待 Sync；也可右键 `oh-package.json5` → **Sync Now**，或执行 `ohpm install --all`。

**签名配置（重要）**：

- `build-profile.json5` 的 `app.signingConfigs[0]` 是 **debug 签名配置**，`certpath`/`profile`/`storeFile`/`keyAlias` 指向原开发者本机路径 `C:/Users/Jason/.ohos/config/openharmony/`，他机不可用。
- 请用 DevEco Studio 的 **File → Project Structure → Signing Configs → Automatically generate signature** 重新生成，或手动填入自己的证书材料。
- **请勿**把个人签名文件、密码提交到版本库。

**命令行构建**：工程**无 `hvigorw` / `hvigorw.bat` 包装脚本**，需用 DevEco Studio 自带的 hvigor（`tools/hvigor/bin/hvigorw`，Windows 为 `hvigorw.bat`），并确保 `node` 在 PATH 中：

```bash
hvigorw clean --no-daemon
hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
```
产物在 `entry/build/default/outputs/default/`。

**安装到设备**：

```bash
hdc list targets
hdc install -r entry/build/default/outputs/default/entry-default-signed.hap
hdc shell aa start -a EntryAbility -b com.jason.opencode
```

## 六、使用说明与配置说明

**页面**：`main_pages.json` 只注册 `pages/Index`（`entry/src/main/ets/pages/Index.ets`）。

**当前可操作的功能**：

1. 启动应用 → 首页居中显示 **Hello World**。
2. 点击该文本 → 变为 **Welcome**，同时调用 `libentry.so` 的 `add(2, 3)`，结果写入 hilog。
3. 验证：`hdc shell hilog | findstr testTag`，应看到 `Test NAPI 2 + 3 = 5`。

> **OpenCode 使用流程（待补充）**：仓库内尚无 OpenCode 功能实现，无法描述其操作方式；实现落地后本节需重写。

**权限声明**：

- `entry/src/main/module.json5` **未声明任何 `requestPermissions`**，当前不申请任何权限。
- 若后续引入 OpenCode 能力（网络、文件、本地服务进程等），需在此补充权限并更新本节——**具体权限项待实现确定后再补充，切勿提前臆断。**

**需要填写的配置项**：

| 位置 | 配置项 | 说明 |
| --- | --- | --- |
| `build-profile.json5` | `signingConfigs[0]` 的 `certpath`/`profile`/`storeFile`/`keyAlias`/密码；`products[0]` 的 compile / compatible / target SDK | 签名为原开发者本机 debug 材料，**必须替换为你自己的**；SDK 当前均为 `12` |
| `entry/build-profile.json5` | `abiFilters: ["arm64-v8a", "x86_64"]` | 真机通常只需 `arm64-v8a` |
| 各 `string.json`（`AppScope`、`entry/src/main/resources/base/element`） | `app_name`；`module_desc`/`EntryAbility_desc`/`EntryAbility_label` | `app_name` 当前为 `OhosOpenCode`；后三者仍是模板默认值，**待补充为正式文案** |
| `oh-package.json5`、`entry/oh-package.json5` | `description`/`author`/`license` | 占位或空值，**待补充**（`license` 建议 `Apache-2.0`） |

## 七、第三方依赖与开源许可

**ohpm 依赖**（依据 `oh-package.json5`、`oh-package-lock.json5`）：

| 包名 | 版本 | 类型 | 用途 |
| --- | --- | --- | --- |
| `@ohos/hypium` | `1.0.25` | devDependencies | 测试框架（单元测试 / UI 测试） |
| `@ohos/hamock` | `1.0.0` | devDependencies | 测试 Mock 框架 |

二者均取自 OpenHarmony 官方 ohpm 仓库（`https://ohpm.openharmony.cn/ohpm/`）；具体许可证条款以各自发布页面声明为准（**待核实 / 待补充**）。

**本地原生库**：`libentry.so` 不是第三方依赖，而是本仓库自带的 NAPI 模块（源码在 `entry/src/main/cpp/`），由 `entry/oh-package.json5` 以 `file:` 引用。

**上游 OpenCode 项目**：

- 本项目是 OpenCode 在 OpenHarmony 平台的移植/客户端工程。
- **（未在仓库内发现上游源码或产物）**：仓库不含 OpenCode 上游代码、Web 前端构建产物（无 `rawfile` 目录），也没有指向上游仓库或依赖其代码的引用。
- 因此**上游的名称、版本、来源与许可证均未在仓库内发现**，本文档不对上游采用何种许可证作出判断。
- 引入上游代码或其产物前，请先确认其许可证条款，并在本节如实登记名称、版本、来源与许可证。
- **仓库内未发现其他 vendored 第三方代码或资源**。

## 八、开源协议

本项目采用 **Apache License 2.0** 开源，全文见仓库根目录 [LICENSE](LICENSE)，亦可访问 http://www.apache.org/licenses/LICENSE-2.0 查阅。

- **允许**：商用、修改、分发，以及贡献者授予的专利许可。
- **须遵守**：保留原始的版权声明、许可声明与来源说明；对修改过的文件作出显著变更说明。
- **不提供担保**：软件按"现状"提供，不附带任何明示或默示担保。

## 九、贡献指南与联系方式

1. Fork 并创建特性分支，分支名建议用 `feature/xxx`、`fix/xxx` 等前缀。
2. 提交前确保代码通过 `code-linter.json5` 的静态检查；**不提交** `build/`、`oh_modules/`、`.hvigor/`、`.idea/`、`local.properties` 等已忽略内容。
3. **不提交个人签名证书、`.p12`、`.p7b`、密码等敏感信息**；单个 PR 尽量只做一件事。
4. 提交信息建议用 `feat:` / `fix:` / `docs:` / `refactor:` 等约定式前缀。

联系方式：作者 GitHub 主页 https://github.com/jansin ｜ 仓库 https://github.com/jansin/OpenHarmony-MuJie-OpenCode ｜ 问题反馈：仓库 Issues。

感谢 **OpenHarmony** 开源社区与 **OpenCode** 上游项目为本项目提供的基础与灵感。
