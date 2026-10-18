# ZError macOS 自动构建

> [Miaozeqiu/ZError](https://github.com/Miaozeqiu/ZError) 的 macOS 版本，**每周自动跟随上游更新并构建**。

## 📥 下载

到 [**Releases**](https://github.com/uloow/ZError/releases) 下载最新的 `ZError_x.y.z_arm64.dmg`。

> 未签名构建。首次运行请**右键点击应用 →「打开」**，直接双击会被 Gatekeeper 拦下。

系统要求：Apple Silicon (M1/M2/M3/M4)，macOS 10.13+

## ⚙️ 它是怎么工作的

`macos-build` 分支**不含源码**，只有构建套件：

```
.github/workflows/build-macos.yml   # 构建流水线
patches/*.patch                     # 6 个 macOS 修复补丁
```

每次运行（每周一 11:00 北京时间，或手动触发）：

1. 查上游 `dev-improvements` 分支最新 commit
2. 和上次构建的 commit 比对，**没变就直接跳过**，不浪费构建资源
3. 有更新 → 检出上游源码 → 按顺序打 `patches/*.patch`
4. `npm ci` → `tauri build` → `hdiutil` 封 DMG
5. 版本号直接取 `src-tauri/tauri.conf.json` 里的值（**跟随上游，不自造版本号**）
6. 发 Release，tag 为 `v{版本号}-macos`；上游同一版本有多次提交时**滚动覆盖同一个 release**，不会堆一堆重复 tag

**补丁打不上时不会出坏包**：流水线会失败并自动开一个 issue 提醒，需要人工把补丁 rebase 到新上游代码上。

## 🔧 内置的 6 个补丁

| 文件 | 解决的问题 |
| --- | --- |
| `01-image-fetch-no-proxy` | 图片题正确率极低——系统代理把超星图片 CDN 拦成 403/567，图片根本没下载成功 |
| `02-updater-no-proxy` | 每次启动弹「更新失败」——`webapi.zaizhexue.top` 走代理返回 HTTP 567，更新器解析不到 release JSON |
| `03-update-error-no-popup` | 自动更新检查失败不再弹窗打断答题，只在手动检查时提示 |
| `04-missing-exports` | 上游 HEAD 有断裂引用，`vue-tsc` 编译不过，补两个缺失导出 |
| `05-ocs-config-v2` | OCS 提示「题库配置暂不支持图片、公式」——「复制配置」模板升级为 POST + 图片字段声明 |
| `06-direct-fetch` | 模型请求偶发 `Timeout waiting for model response`——9 处请求改直连，绕开系统代理 |

补丁是标准的 `git apply` 格式，可以单独摘出来用：

```bash
git clone -b dev-improvements https://github.com/Miaozeqiu/ZError.git
cd ZError
git apply /path/to/patches/*.patch
```

## 🖐 手动触发

Actions → **Build macOS** → Run workflow，勾上 `force` 可以忽略"上游无更新"强制重建。

## ⚠️ 已知限制

- **未签名、未公证**，首次打开需右键 →「打开」
- 配置（`config.json` / `model_config.json`）在上游设计里写在 `.app` 包内 `Contents/MacOS/`，**重装会丢配置和 API Key**，升级前请备份
- 只出 arm64 包（runner 是 Apple Silicon）；Intel Mac 不适用
- 上游 `update.json` 的下载地址指向他们自己的 CDN，应用内自动更新对本构建不适用，请手动下载新版本

## 🔗 相关

- 上游：[Miaozeqiu/ZError](https://github.com/Miaozeqiu/ZError)
- 手工版（含完整源码快照）：[uloow/ZError-macOS](https://github.com/uloow/ZError-macOS)

---

*本仓库仅为 macOS 用户提供构建产物，代码版权归上游作者所有。*
