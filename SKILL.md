---
name: 最强安卓APP上架
description: 把鸿蒙/H5 工具类 APP 打成可上架的安卓 APK、修 WebView 壳常见坑，并推进国内安卓应用商店（尤其华为 AGC）上传提审时使用。
---
# 最强安卓 APP 上架

把已有鸿蒙/Web 工具类产品同步到安卓商城时按本流程执行。HAP 不能当 APK。优先华为 AGC 安卓通道，再铺小米/OPPO/vivo/荣耀/应用宝。

## 硬约定

1. **包名文件**：桌面/上传目录命名为「`{中文应用名}.apk`」，不要加「安卓」，不要用拼音主文件名。ASCII 副包仅可放 `ascii-backup/`。
2. **交付节奏**：侧载自测由助手用云测/本机 adb/静态加固闭环；不要把「请用户手机试一下」当默认卡点。用户授权全权推进时：**修完即上传（及按授权提审）**，勿逐步请示。
3. **安卓隐私**：华为「隐私托管」面向鸿蒙/元服务；安卓须自备可公网访问的 HTTPS 隐私 URL（可用 Cloudflare Worker 子域名）。
4. **签名**：沿用该工程既有 upload keystore，versionCode 单调递增。
5. **鸿蒙 AGC 工程**：不要抢鸿蒙 bot 的 HAP/AGC 流程；只动安卓应用与安卓包。

## 标准流水线

1. **盘点**：工程是否已有 Android 壳 / 仅 HAP / 已有 release APK。无壳则建最小 WebView 壳或 Capacitor，禁止把 HAP 改后缀。
2. **隐私与材料**：隐私 HTTPS、图标、截图、简介、分类、备案字段（单机也常要备案号或说明）。
3. **出包**：release 签名 APK；`aapt dump badging` 核对 package、versionName、versionCode、targetSdk、`INTERNET` 等权限、application-label 中文名。
4. **上架前必过清单**（见下节 WebView 壳）→ 修 → 重打 → 再上传。
5. **华为 AGC 安卓**：创建/选择安卓应用 → 上传 APK → 填资料 →（授权后）提交审核。驳回后读审核报告/附件 logcat，按根因修版本再传。
6. **多店**：账号实名须用户本人；助手可代填引导。有华为账号后并行注册其它店。

## WebView / 混合壳：上架前必过清单

这些是真实驳回与真机闪退里反复出现的坑，**出包前默认全部检查**。

### 启动即闪退（审核「首次/再次启动失败」）

- Manifest 必须有 `INTERNET`（即便主要加载 `file`/`appassets`）。缺权限时华为机常见 onCreate 内秒崩，且审核 log 可能剥掉 FATAL 栈。
- `targetSdk` 建议 **33 或 34**；对鸿蒙 2/3/4 审核机避免一上来就 35。
- **禁止**在 `MainActivity` 字节码里直接引用 API 30+ 的 `WindowInsetsController` / `WindowInsets.Type` / `setDecorFitsSystemWindows`。一律拆到 `@RequiresApi(30)` 独立类，否则旧 ART 可能 VerifyError，表现为无 WebView 日志的极早 SIGKILL，try/catch 也拦不住。
- `new WebView` + settings 包 try/catch；失败时展示错误页，不要立刻 `finish()` 造成「点一下就闪退」。
- 去掉 `setAllowFileAccessFromFileURLs` / `setAllowUniversalAccessFromFileURLs`；本地资源优先 `WebViewAssetLoader`（`https://appassets.androidplatform.net/...`）。
- 保留对老 WebView 有害 API 的防护（如禁用 `MessageChannel`），但若进程死在 `loadUrl` 之前，先修原生壳。

### 系统栏 / 全屏观感

- 状态栏、导航栏颜色与页面底色一致（或透明 + 页面延伸）。
- 浅色底用深色状态栏图标（`LIGHT_STATUS_BAR` / API30 Appearance）。
- 沉浸式 insets 只放在 API30+ 独立类；向页面注入 safe-area CSS 变量时注意手势条遮挡底栏。

### 底栏 / 小方字图标

- 方框内单字用 flex 水平垂直居中，`line-height: 1`，必要时轻微 `translateY` 做光学居中；不要靠默认 inline 基线。

### 导出下载 / 分享 / 选文件（WebView 必接系统能力）

| 能力 | 错误表现 | 正确做法 |
|------|----------|----------|
| 下载档 | Toast 成功但下载目录没有 | MediaStore 写 `RELATIVE_PATH=Download/`，写后校验；失败改走分享，**禁止假成功**；可用 DownloadManager 通知 |
| 分享 | — | cache + FileProvider + `ACTION_SEND` |
| 选文件 | 按钮无反应 | `WebChromeClient.onShowFileChooser` + `OPEN_DOCUMENT`/`GET_CONTENT`，`onActivityResult` 回传 `filePathCallback`；仅 `<input type=file>` 不够 |
| 复制 | — | 剪贴板即可 |

用 `@JavascriptInterface` 桥接下载/分享；`setAllowContentAccess(true)` 以读 SAF `content://`；勿随意打开通用 file URL 访问。

### 文件命名与互传

- 主交付物：`应用名.apk`（中文）。
- 中文名经部分互传/控制台易 mojibake；构建脚本用 .NET/`[IO.File]::Copy` 写 Unicode 路径，并 `.NET Directory.GetFiles` 校验，不要信乱码控制台输出。

## 华为审核驳回处置

1. 邮件常无细则 → AGC「审核报告」。报告页若因空 screenshot URL 报 JS 错导致列表空白，从版本历史/接口 JSON 或「审核附件」取因。
2. 附件多为 logcat + 录屏。搜 `dispatchingThrewException`、`SIG: 9`、`AndroidRuntime`、包名；无堆栈时反编译对 onCreate。
3. 「启动失败」优先按上面启动清单修，升 versionCode 再传，勿只改 JS。

## 版本与产物

- 每轮修复：`versionName` 语义化升，`versionCode` +1。
- 输出：`Desktop\{应用名}.apk`，以及 `Desktop\鸿蒙上架\安卓上架\{应用名}-release-{version}.apk`（上传用，仍避免无意义的「安卓」后缀作主名）。
- `apksigner verify` + badging 留证。

## 多应用推进

同一开发者多款 APP：逐个（或小并行）走「壳检查 → 必过清单 → 出包 → AGC 上传/提审」。一款稳定后复制壳能力到下一款，避免每款重复踩 WebView 坑。
