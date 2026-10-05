# ClashX Meta（自用补丁版）

在官方 [ClashX Meta](https://github.com/MetaCubeX/ClashX.Meta) 的每个正式版上打补丁、自动编译，发布到本仓库的 Releases。

## 补丁内容

| 补丁 | 作用 |
|---|---|
| `patches/0001-group-test-url.patch`（必需） | 菜单栏 |
| `patches/optional/0002-dashboard-group-test-url.patch`（可选） | 内置面板（Dashboard） |

按组决定用哪个测速链接、显示哪个链接测出的延迟：

- **配置里写了 `url` 的组**：组的「测速」和「重新测速」都用这个 url。菜单和面板只显示这个 url 测出的延迟，没测过就显示空白，不会拿别的链接测出的结果来充数。
- **没写 `url` 的组**：和官方一样用全局测速链接。显示时优先用全局链接的结果，没有就退回官方的做法，显示最后一次测速结果，不管它是用哪个链接测的。
- 菜单顶部的「测速」：除了用全局链接测所有节点，还会用组自己的 url 测一遍写了 `url` 的 select 组。url-test、fallback 这类自动组不测，因为测它们会清掉手动固定的节点。

可选补丁打不上时，构建照常进行，只是不带这部分改动。Actions 里会出现警告，Release 说明里也会写明跳过了哪个补丁。

## 首次设置

1. 在 GitHub 上新建一个**公开**仓库（公开仓库的 macOS 构建免费），把本目录推上去。
2. 打开仓库的 Actions 页面，启用 workflow，然后对 **Build patched ClashX Meta** 手动点 Run workflow。
   - 编译不过的话，大概率是 Xcode 版本问题。可以在 `xcode` 里填一个版本号（如 `26.4`），或在 `runner` 里换一台机器再试。
3. 构建完成后，到 Releases 下载 `ClashX.Meta.zip`，然后安装（旧版本会移到 `~/Downloads/ClashX Meta.old.app`，方便回退）：

```bash
osascript -e 'quit app "ClashX Meta"'
```

```bash
cd ~/Downloads && rm -rf "ClashX Meta.app" "ClashX Meta.old.app" && ditto -x -k ClashX.Meta.zip . && mv "/Applications/ClashX Meta.app" "ClashX Meta.old.app" && mv "ClashX Meta.app" /Applications/ && xattr -cr "/Applications/ClashX Meta.app" && open "/Applications/ClashX Meta.app"
```

bundle ID 和官方相同，配置、提权助手都照常使用。

## 之后的同步

- workflow 每天检查一次官方最新的正式版。还没编译过的版本会自动打补丁、编译，并发布为 `vX.Y.Z-patched`。
- 必需补丁打不上时，构建失败，GitHub 会发邮件通知。这时需要按下面的方法更新补丁。
- 本版本里的 Sparkle 更新源改成了本仓库的空 `appcast.xml`，所以应用不会提示更新到官方版。官方版会覆盖掉补丁版。
- GitHub 的规定：公开仓库 60 天没有提交时，定时任务会被停用，到时会发邮件提醒，点一下就能继续。

## 修改或更新补丁

```bash
git clone https://github.com/MetaCubeX/ClashX.Meta.git /tmp/ClashX.Meta
```

```bash
cd /tmp/ClashX.Meta && git switch -c patched v1.4.46 && git am --3way ~/code/clashx-meta-patched/patches/*.patch ~/code/clashx-meta-patched/patches/optional/*.patch
```

把 `v1.4.46` 换成要适配的版本。解决冲突、修改并提交后，重新导出补丁：

```bash
cd /tmp/ClashX.Meta && git format-patch -1 HEAD~1 --stdout > ~/code/clashx-meta-patched/patches/0001-group-test-url.patch && git format-patch -1 HEAD --stdout > ~/code/clashx-meta-patched/patches/optional/0002-dashboard-group-test-url.patch
```
