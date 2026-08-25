---
name: zhiqi-xhs-article-card-studio
description: 智企小红书文章成卡工坊。把用户提供的文章、文案、字幕、截图或笔记先拆成封面、金句、正文、要点和结尾，再交给独立的 Article 模板渲染为小红书 3:4 PNG 图文卡片。内容拆解与图片渲染均在本 Skill 内完成，不调用其他智企 Skill。
---

# 智企小红书文章成卡工坊

这是一个独立 Skill，专门处理“文章/文案 → 内容拆解 → Article 模板 → 小红书图片”。不要读取、修改或调用 `zhiqi-xhs-content-card-studio` 或其他智企 Skill。

## 固定边界

- **内容拆解层**：使用本目录 `vendor/rednote-director-skill/` 中的上游拆解规则，负责识别核心观点、传播钩子、页面节奏和逐页文案。
- **图片渲染层**：使用本目录 `vendor/article-renderer/` 中的 Article 渲染器，负责 HTML/CSS 模板和 Playwright 截图。
- 内容拆解层只输出页面计划，不生成最终 PNG。
- 图片渲染层只按页面计划排版，不擅自改变事实或补写原文。
- 两层都只使用本 Skill 内的文件，不能跨 Skill 读取资源。

## 内容来源规则

用户提供的文章、文案、字幕、截图或笔记是第一来源。先提炼，再成卡；不要把通用话术冒充原文。

当素材不足时：

1. 先说明缺少哪些信息。
2. 询问是否允许补充公开资料或 AI 创意补写。
3. 允许后，分别标记 `原始素材`、`公开资料补充`、`AI 补写`。
4. 只有用户允许后，才可以搜索或创意扩写。

## 工作流程

### 1. 接收素材

接受 Markdown、纯文本、字幕、截图说明、文章链接或用户提供的图片。只需要询问会影响成卡的缺失信息：目标页数、是否保留全文、Article 皮肤（E 雅刊、A 暖杏奶咖、B 雾松青、C 黛墨描金、D 夜读鎏金或女性向主题）和是否允许补充素材。

### 2. 输出内容拆解计划

先在内部形成以下结构，再开始排版：

```text
主题：
核心观点：
目标读者：
传播钩子：
建议页数：
P1 / cover / 标题 / 导语 / 素材来源
P2 / quote-or-body / 标题 / 正文 / 素材来源
P3 / body-or-list / 标题 / 正文 / 素材来源
...
Pn / ending / 结尾 / 行动提示 / 素材来源
```

默认 5–9 页；用户明确要求全文转换时，按文章结构完整拆页，不为了凑页删掉事实。每页只承担一个主要信息任务。

页面类型只在内容需要时使用：封面、金句、正文、清单/步骤、图文混排、结尾。不要把每篇文章强行套成同一套卡片。

### 3. 交给 Article 渲染器

从 `vendor/article-renderer/assets/template.html` 开始，在任务目录生成 `index.html`，按内容拆解计划填入 `.card`、`.dcard`、`.ecard` 或 `.femcard` 页面。不要从空白 HTML 重新设计，也不要修改 `vendor/article-renderer/assets/` 内的模板。

任务目录建议：

```text
local-tests/<slug>/
├── index.html
├── assets/
└── output/
```

渲染命令：

```bash
node vendor/article-renderer/scripts/render.cjs local-tests/<slug>
```

依赖未安装时，在 `vendor/article-renderer/` 执行 `npm install`，再运行 Playwright 渲染。最终输出必须是 1080×1440 的 PNG。

### 4. 交付前检查

- 标题、正文、页码没有溢出。
- 页面顺序与内容拆解计划一致。
- 图片没有裁掉主体或关键文字。
- 原始素材、公开资料和 AI 补写标记清楚。
- 给出每张 PNG 的绝对路径，并说明使用的 Article 皮肤。

## 不做的事

- 不调用现有的 `智企小红书图文工坊`。
- 不使用 Guizang Editorial / Swiss 模板。
- 不主动抓取书籍全文、付费资料或未授权内容。
- 不把 AI 补写内容伪装成原文、引语或事实。
- 不把 Article 渲染器改造成选题工具或自动发布工具。
