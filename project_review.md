# DailyTask 项目审查报告

## 项目概览

这是一个 Android 自动打卡工具，使用 Kotlin + Java 混编，Gradle 8.12.1 构建，目标 SDK 36（Android 16）。

---

## 🔴 严重问题

### 1. 签名密钥硬编码在 [app/build.gradle](file:///Users/menghuizhang/Github/DailyTask/app/build.gradle) 中

> [!CAUTION]
> [app/build.gradle](file:///Users/menghuizhang/Github/DailyTask/app/build.gradle#L34-L37) 中签名密钥密码（`123456789`）直接明文写死。这是严重的安全隐患——任何能看到代码的人都能拿到签名证书。

**建议**：将密码移至 `local.properties` 或环境变量，并确保该文件在 [.gitignore](file:///Users/menghuizhang/Github/DailyTask/.gitignore) 中。

### 2. JKS 签名文件提交到了 Git 仓库

> [!CAUTION]
> [app/DailyTask.jks](file:///Users/menghuizhang/Github/DailyTask/app/DailyTask.jks) 签名文件存放在代码仓库中。签名密钥不应提交到版本控制系统。

**建议**：将 `*.jks` 加入 [.gitignore](file:///Users/menghuizhang/Github/DailyTask/.gitignore)，后续从安全渠道获取。

---

## 🟡 构建配置问题（Gradle Problems Report 中的 9 个警告）

> [!WARNING]
> Gradle 8.12.1 报告了 **9 个弃用警告**，全部是 Groovy DSL 中的 **空格赋值语法**，将在 Gradle 10.0 中移除。

涉及 [app/build.gradle](file:///Users/menghuizhang/Github/DailyTask/app/build.gradle) 中以下字段：

| 当前写法（空格赋值） | 应改为（= 赋值） |
|:---|:---|
| `namespace 'com.pengxh.daily.app'` | `namespace = 'com.pengxh.daily.app'` |
| `compileSdk 36` | `compileSdk = 36` |
| `minSdk 26` | `minSdk = 26` |
| `targetSdk 36` | `targetSdk = 36` |
| `ndkVersion '21.4.7075529'` | `ndkVersion = '21.4.7075529'` |
| `signingConfig signingConfigs.release` | `signingConfig = signingConfigs.release` |
| `version "3.22.1"` | `version = "3.22.1"` |
| `buildConfig true` | `buildConfig = true` |
| `viewBinding true` | `viewBinding = true` |

---

## 🟡 其他值得注意的问题

### 3. `android.useDeprecatedNdk=true` 已废弃

> [!WARNING]
> [gradle.properties](file:///Users/menghuizhang/Github/DailyTask/gradle.properties#L22) 中仍保留了 `android.useDeprecatedNdk=true` 这一历史配置。建议确认当前 NDK 构建链路不再依赖它后移除，避免误导后续维护者。

### 4. `android.enableJetifier=true` 可能不再需要

> [!IMPORTANT]
> [gradle.properties](file:///Users/menghuizhang/Github/DailyTask/gradle.properties#L19) 中 `enableJetifier` 用于将旧的 Support Library 依赖转换为 AndroidX。从当前直接依赖看已基本是 AndroidX，可在验证传递依赖也不再需要后尝试关闭，以减少构建开销。

### 5. 若后续重新启用飞书支持，需要补充 `<queries>` 声明

[Constant.kt](file:///Users/menghuizhang/Github/DailyTask/app/src/main/java/com/pengxh/daily/app/utils/Constant.kt) 中保留了飞书包名 `com.ss.android.lark`，但 [SettingsActivity.kt](file:///Users/menghuizhang/Github/DailyTask/app/src/main/java/com/pengxh/daily/app/ui/SettingsActivity.kt#L40-L55) 和 [Constant.kt](file:///Users/menghuizhang/Github/DailyTask/app/src/main/java/com/pengxh/daily/app/utils/Constant.kt#L39-L46) 中对应入口当前处于注释状态，因此这不是已触发的线上问题。

如果后续恢复飞书选项，需要同步在 [AndroidManifest.xml](file:///Users/menghuizhang/Github/DailyTask/app/src/main/AndroidManifest.xml#L69-L76) 的 `<queries>` 中增加 `com.ss.android.lark`，否则 Android 11+ 下将无法稳定查询飞书是否安装。

### 6. [EmailManager](file:///Users/menghuizhang/Github/DailyTask/app/src/main/java/com/pengxh/daily/app/utils/EmailManager.kt#18-89) 中日志输出了邮箱配置信息

> [!WARNING]
> [EmailManager.kt:48](file:///Users/menghuizhang/Github/DailyTask/app/src/main/java/com/pengxh/daily/app/utils/EmailManager.kt#L48) 用 `Log.d` 打印了完整的邮箱配置 JSON（包含授权码），Release 版本应移除或使用 `BuildConfig.DEBUG` 保护。

### 7. 顶层 [build.gradle](file:///Users/menghuizhang/Github/DailyTask/build.gradle) 插件声明风格不一致

[build.gradle](file:///Users/menghuizhang/Github/DailyTask/build.gradle) 混用了 `id("...")` 和 `id '...'` 两种写法，建议统一。

### 8. `compileOptions` 使用 Java 8

当前项目设置 `sourceCompatibility = Java 1.8`，考虑到 `targetSdk = 36`，可以升级到 Java 11 或 17 以使用更多现代 Java 特性。

---

## ✅ 没有问题的地方

- AndroidManifest 权限声明较为合理
- Room 数据库使用正确
- 项目结构清晰，分层合理（ui / service / utils / sqlite / retrofit）
- 依赖库版本比较新
