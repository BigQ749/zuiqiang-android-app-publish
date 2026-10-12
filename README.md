# 最强安卓 APP 上架

把鸿蒙 / H5 工具类 APP 打成可上架的安卓 APK，按 WebView 壳必过清单修好启动闪退、系统栏、下载分享选文件等坑，再推进华为 AGC 等国内安卓店上传提审。

对应本地 Grok Bot skill：[最强安卓APP上架](sand-workflow:app)（仓库内以 `SKILL.md` 为准）。

## 装到 Cursor

把本仓库放到：

`C:\Users\齐赛军\.cursor\skills\最强安卓APP上架`

或克隆：

```bash
gh repo clone BigQ749/zuiqiang-android-app-publish "%USERPROFILE%\.cursor\skills\最强安卓APP上架"
```

## 里面有什么

| 文件 | 干什么 |
|---|---|
| SKILL.md | 接到安卓上架任务后怎么做：硬约定、流水线、WebView 必过清单、驳回处置 |

## 硬约定摘要

- 安装包桌面命名：`{中文应用名}.apk`，不加「安卓」，不用拼音主名
- HAP 不能当 APK；安卓须自备 HTTPS 隐私页（华为隐私托管只管鸿蒙）
- 全权推进时修完即上传，不以「请用户侧载」为默认卡点

口令与 keystore 不在这个仓库里。
