# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 仓库性质

这是一个静态的、单页双语（中文 / English）API 集成文档站点，面向在线游戏平台（真人娱乐场、老虎机、迷你游戏、捕鱼、体育、扑克）。仓库名为 `topapidoc.github.io`，由 GitHub Pages 直接托管，**没有构建系统、没有包管理器、也没有任何服务端代码**。`<html>` 标签上的 `xmlns:th="http://www.thymeleaf.org"` 属性是历史残留，未被使用，不存在任何服务端渲染。

全部内容都在 **`index.html`**（约 4900 行）中：内联 CSS、内联 JS、所有文案。没有 `src/`、没有资源目录、没有依赖。

## 本地预览

直接用浏览器打开 `index.html`，或在仓库目录下启一个 HTTP server：

```powershell
python -m http.server 8000
# 然后访问 http://localhost:8000
```

没有测试、没有 lint、没有 build 命令。修改后推送 `main` 分支即上线，GitHub Pages 直接对外提供文件。

**密码门控：** 页面加载时会执行 `prompt("请输入访问密码")`，如果输入的不是 `1234567811`，整个 `<body>` 会被替换成 "Access Denied"（见 `index.html` 4769 行附近）。这只是纯客户端校验，可被轻易绕过，**不要把它当成真正的身份验证**。本地预览时记得输入密码，否则会看到空白页面。

## `index.html` 的内部结构

UI 由三个相互配合的部分驱动，**改动前必须理解它们之间的契约**：

1. **左侧目录树** — `#sidebar` 内的 `<div class="tree-item level-{1,2,3}" data-section="<id>">`（从 671 行左右开始）。`has-children` / `expanded` / `collapsed` 这三个 class 控制折叠箭头；**层级关系通过相邻兄弟节点的 `level-N` 数字来表达，而不是 DOM 嵌套**。JS 在判断某个节点的子项时，是沿兄弟节点向后遍历，直到遇到一个 `level` 数字小于等于当前节点的元素为止。

2. **右侧内容面板** — `<div class="content-section" id="<id>">`（从 847 行开始）。同一时刻只有一个面板带有 `.active`；点击目录树的某一项时，会把 `.active` 切换到 `id` 等于该项 `data-section` 的面板上。**目录项的 `data-section` 必须和内容面板的 `id` 完全一致** —— 这是唯一的关联手段，很容易改坏。

3. **双语文本** — 所有需要翻译的元素都同时带有 `data-zh="..."` 和 `data-en="..."`。`updateTextContent()` 会按当前语言把这些元素的 `textContent` 重写为对应属性的值。**重要后果：带有 `data-zh`/`data-en` 的元素不能包含子 HTML 节点**，因为切换语言时会被 `textContent` 整个覆盖。如果某段文本里需要 `<br>`、`<strong>` 等内联标签，要把 `data-zh`/`data-en` 放在内层的 `<span>` 上，而不是外层的容器上。现有代码遵循这个约定，新增内容时也请保持一致。

默认语言为中文（`DOMContentLoaded` 中调用 `switchLang('zh')`），默认显示的面板是 `version-control`。

## 新增一个 API 章节

新增一个接口文档（这是最常见的修改），必须同步改动三个地方：

1. 在 sidebar 中加一行 `<div class="tree-item level-2" data-section="my-new-id">`，放到对应的父级分组下（II. 集成 API、III. 无缝钱包 API 等），让现有的展开/折叠逻辑能正确归类。
2. 在下方对应位置加一个 `<div class="content-section" id="my-new-id">`，写入正文。建议参考相邻章节的结构（标题 → 描述 → 请求/响应表格），保持视觉一致。
3. 目录项和内容面板里每一段可见文本都要同时提供 `data-zh` 和 `data-en`。任意一个缺失，那种语言下该元素就会显示为空白。

同时还要在 **版本控制** 章节的变更日志表（855 行附近）里加一行 —— 这是用户打开页面第一眼看到的内容，从最近的提交记录也能看出这是约定俗成的做法。

## 从历史记录中得到的约定

- **提交信息用中文**，描述具体的内容改动（例如 "修改API文档：游戏指示器：1: 真人娱乐场 ..."）。内容类修改请沿用这种风格。
- **游戏指示器编码**（最近几次提交统一的标准）：`1`=真人娱乐场，`2`=老虎机，`3`=迷你游戏，`4`=捕鱼游戏，`5`=体育，`6`=扑克。如果你改到任何列出这些编码的表格，请保持一致。
