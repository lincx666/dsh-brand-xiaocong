# dsh-brand-xiaocong

把 DSH 界面上几处「官方标识」换成自己的：侧边栏品牌、大厅标题、模型标签、空白会话标题。

图标来自一张贴纸，抠掉白底、裁到轮廓、缩成 96px，以 base64 内嵌在客户端模块里——不依赖外部文件，复制包就带走了。

## 能力

| 位置 | 实现方式 | 说明 |
|:--|:--|:--|
| 侧边栏图标 + 文字 | 插槽 `sidebar.brand.mark` / `sidebar.brand.name` | `priority: -1` 顶掉官方注册 |
| 大厅图标 | 插槽 `conversation.hero.brand.mark` | 尺寸 ×1.35 |
| 大厅标题文案 | 页面层替换 | 真值在 locale，重复注册会抛错 |
| 大厅标题配色/字号 | 页面层内联样式 | `#4C7F6E`，`1.18em` |
| 「预览版」徽章 | 页面层隐藏 | `display: none`，不删节点 |
| 模型标签 | 插槽 `conversation.input.model.label` | 只改名，不改 id |
| 推理等级 `High` | 注入样式 | `[class*="triggerEffort"]` |
| 空白会话标题 | 页面层替换 | `session.new` 渲染成「新画卷」 |

## 安装

本机（profile 为 `desktop`）：

```powershell
dsh plugin add C:\Users\lincx\Documents\deepseek-harness\default-workspace\dsh-brand-xiaocong
```

它会给 profile 加一条 `link:` 依赖，并在 `dsh.profile.bundles` 里登记。装完要重启 DSH，host 端的 bundle 列表是启动时读的。

## 卸载

```powershell
dsh plugin remove dsh-brand-xiaocong
```

或手动把 `~/.dsh/profiles/desktop/package.json` 里的 `dsh-brand-xiaocong` 从 `dependencies` 与 `dsh.profile.bundles` 各删一行。

## 想换内容时改哪里

全部集中在 `lib/client.js` 顶部：

| 常量 | 作用 |
|:--|:--|
| `HEADLINE_TO` | 大厅标题文字 |
| `HEADLINE_COLOR` / `HEADLINE_SCALE` | 大厅标题颜色与字号 |
| `NEW_SESSION_FROM` / `NEW_SESSION_TO` | 空白会话标题替换 |
| `MODEL_ID_RENAMED` / `MODEL_NAME` | 哪个模型、显示成什么 |
| `HeroMark` 里的 `1.35` | 大厅图标缩放 |
| `ICON`（base64） | 图标本身，来自 `assets/xiaocong.png` |

改完不需要重装，`link:` 指向的就是这个目录；刷新页面或重启 DSH 即可。

## 已知限制

- **页面层替换**依赖 class 名片段（`titleGroup`、`previewBadge`、`triggerEffort`、`data-row-key^="session:"`、`[data-composer-placeholder]`）。DSH 改了这些命名就会失效，表现为「某处文字回退成官方版本」，不会报错。
- 插槽部分稳一些，但插槽名或 `single` 语义变了同样会失效。
- 图标是位图，96px 在 24px 显示清晰；要更锐利得换 SVG。
- 只覆盖中文界面：`HEADLINE_FROM` 里带了英文兜底，其余按中文文案匹配。

## 设计取舍

页面层替换放在 `MutationObserver` 里做，观察根是 `document.documentElement` 而不是 `document.body`——客户端模块加载时 body 还不存在，观察 body 会让整条链静默失效。

文本替换一律只碰文本节点、只碰指定的宿主元素，聊天正文不在扫描范围内。
