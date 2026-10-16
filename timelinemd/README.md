# Timeline Markdown

用一份纯 Markdown 文件渲染 [vis-timeline](https://visjs.github.io/vis-timeline/)
时间轴，而不用手写 JSON。是 [`../timeline`](../timeline) 的姊妹工具（本工具
的 JS/CSS 资源就是从那里拷贝过来的）——它在 `jstools` 里的定位见
[`../README.md`](../README.md)。

## 用法

```
https://junxnone.github.io/jstools/timelinemd/?md=your_markdown_url
```

- `?md=<url>` — 抓取并渲染该 Markdown 文件。如果 URL 不是以 `.md` 结尾会
  自动补上。不传时默认读取 `test.md`（即下面的示例文件）。
- `?str=<markdown>` — 直接渲染一段内联 Markdown 字符串，不去抓文件（适合
  简短演示；长文档不建议用这个）。

- [demo](https://junxnone.github.io/jstools/timelinemd/?md=https://junxnone.github.io/jstools/timelinemd/test)

## Markdown 格式

### Frontmatter（可选）

文件开头用 `---` 包裹的一段，里面是扁平的 `key: value` 行（不是真正的
YAML——不支持列表，引号规则也很简单，就是"冒号后面的内容原样当字符串"）：

```markdown
---
title: 中国火星探测
timeline.orientation.axis: both
timeline.zoomable: true
timeline.start: 2020-01-01
timeline.min: 2020-01-01
---
```

- `title` 用来设置页面的 `<title>`。除 `timeline.*` 外的其他 key 会被忽略。
- `timeline.a.b.c: value` 会写入底层 vis-timeline 的
  [配置对象](https://visjs.github.io/vis-timeline/docs/timeline/#Configuration_Options)
  的 `a.b.c` 路径——没有 key 白名单，所以 vis-timeline 的任何配置项都能这样
  设置（`timeline.margin.item`、`timeline.orientation.item`、
  `timeline.zoomable` ……）。
- 值会做类型转换：`true`/`false` → 布尔值，纯数字 → 数值，其余保持字符串
  （`"both"`、`"800px"`）。
- `timeline.start`、`timeline.end`、`timeline.min`、`timeline.max` 这四个
  是特例，会按日期解析（和下面事件日期用的是同一套解析逻辑，包括 `now`
  关键字），而不是当成普通字符串。

如果你自己不设置 `start`/`end`/`min`/`max`，工具会根据数据自动计算：默认
显示的时间窗口、以及能拖动/缩放到的硬性边界，都会被设为精确的
`[最早事件的开始时间, 最晚事件的结束时间]`——不留空白边距，也不能滑出这个
范围。在 frontmatter 里显式设置这四项中的任意一个即可覆盖这个行为（比如
留一点边距，或者允许用户看到范围之外）。

另外也没有固定的默认 `height`——时间轴会根据内容自动调整高度，所以不会有
内部滚动条。如果想要固定高度并允许内部滚动，设置 `timeline.height` 即可。

### 分组

一级标题（`#`）开启一个新分组——直接对应 vis-timeline 的
[group](https://visjs.github.io/vis-timeline/docs/timeline/#groups)（带标签
的一行）。分组按第一次出现的顺序排列。

```markdown
# 中国

...事件...

# 美国

...事件...
```

如果文件里完全没有 `#` 标题，所有事件会渲染在同一个无标签的行里（不分组）。

### 事件

二级标题（`##`）表示一个事件：

```
## <开始日期>[ ~ <结束日期>]  <名称>  {属性}
```

```markdown
## 2020-07-23 ~ now  天问一号  {background}

## 2021-02-10 ~ 2021-02-15  近火捕获至平面机动  {range}

## 2020-07-30 11:50 ~ 2020-07-30 12:30  Perseverance launch window  {range}

## 2020-08-02  第一次轨道修正
```

- **`<开始日期>` / `<结束日期>`** — 支持 `YYYY`、`YYYY-MM`、`YYYY-MM-DD`、
  `YYYY-MM-DD HH:mm`（空格或 `T` 分隔，秒数可选），或关键字 `now`。日期
  按严格格式解析——不符合这几种写法之一会被当作解析错误（见下面的
  [校验](#校验)），而不会被"猜"出一个日期。
- **`<结束日期>`** 可选，和开始日期之间用 `~` 分隔（两边加不加空格都行，
  但加上更易读）。
- **`<名称>`** — 日期之后、`{属性}` 之前的所有文字。这是直接画在
  **时间轴色块上** 的短标签，所以要尽量简短；详细描述放在标题下方的正文
  里（见 [正文](#正文)）。不写的话会用开始日期字符串当兜底标签。
- **`{属性}`** — 可选，必须出现在标题行末尾。按空格/逗号分隔多个词：
  - `point` / `range` / `box` / `background` 设置 vis-timeline 条目的
    [`type`](https://visjs.github.io/vis-timeline/docs/timeline/#items)。
    默认值：没有结束日期时为 `point`，有结束日期时为 `range`。
  - 其他词会作为 CSS class 加到该条目上（`className`）——比如 `{expected}`
    会复用本工具 `<style>` 里已经定义好的虚线样式
    `.vis-item.expected`。
  - `background` 类型**必须**有结束日期——vis-timeline 对没有结束日期的
    background 条目会直接报错，所以本工具会提前拦截并给出可读的错误提示，
    而不是让页面崩掉。

一个**没有** `#` 分组包裹的 `{background}` 事件是有意设计的功能，不是
bug：vis-timeline 会把不属于任何分组的 background 条目归到内置的
`__background__` 分组，让它贯穿所有分组、覆盖整个图表高度——适合用来标出
一段整体的时代/阶段。而不属于任何分组、且不是 background 类型的事件，会被
自动归入一个合成的 `(未分组)` 行（并在错误提示区给出说明），不会被悄悄
隐藏掉。

### 正文

每个 `##` 标题到下一个 `#`/`##` 标题之间的内容，就是该事件的正文，会按
Markdown 渲染（通过 [showdown](https://github.com/showdownjs/showdown)）：

```markdown
## 2020-07-23  海南文昌发射天问一号

在海南文昌航天发射场由长征五号遥四运载火箭发射。

- 发射质量约 5 吨
- 包含环绕器、着陆器、巡视器
```

这部分内容**不会**显示在时间轴色块上（vis-timeline 的色块/提示框本来就放
不下多段落内容——参见本工具为短时间段加的 min-width/nowrap 相关 CSS
处理）。而是显示在时间轴下方的**详情面板**里：页面加载时默认显示时间上最早
的那个事件，之后点击某个事件或使用导航按钮都会更新这里。没有正文的事件会
显示 `（无正文）`。

正文里的围栏代码块（```` ``` ```` 或 `~~~`）会被正确处理——代码块内部看起来
像 `##` 的行，会被当作代码内容，不会被误判成新事件的开始。

### 导航

顶部的 `< / First / >` 按钮按**时间先后顺序**（按 `start` 日期，而不是文档
中出现的顺序）在事件间移动，同时会跳转视图并更新详情面板。

## 显示细节：短时间段如何保证可读

如果一个事件的持续时间相对整个时间轴跨度很短（比如几年跨度的时间轴里有个
只持续 5 天的阶段），色块可能会窄到只有几个像素，标签文字也会被裁切到只剩
一两个字。本工具做了两处处理：

- `.vis-item.vis-range` / `.vis-item.vis-background` 设置了 `min-width`，
  保证色块本身不会窄成一条看不见的细线；
- 色块内部的裁切（`overflow: hidden`）被关闭，让标签文字可以溢出到色块
  边界之外完整显示，而不是被截断——短标签偶尔与相邻条目视觉上略微重叠，
  也好过标签被裁得看不出写的是什么。

## 示例

完整示例见 [`test.md`](test.md)（两个分组、一个背景带、普通的单点事件、
一个跨多天的时间段、一个带具体时刻的时间段，以及一个 `{expected}` 样式的
事件）。

## 校验

遇到问题会在时间轴上方的红色错误框里汇总列出，而不是默默渲染出一个空白或
错误的图表：

- `<开始日期>`/`<结束日期>` 无法解析（不符合支持的日期格式）
- `<结束日期>` 早于 `<开始日期>`
- 出现未知的 `{type}` 取值
- `{background}` 没有 `<结束日期>`
- 事件标题前面没有任何 `#` 分组（会归入 `(未分组)`，并给出提示——不会被
  悄悄丢弃）

如果**一个事件都没能解析出来**，会显示提示信息，而不是一个空页面。

## 从零写一份

参见 [`skills/timelinemd-writer`](skills/timelinemd-writer)，这是一个
Claude Code skill，能帮你针对某个主题调研并写出一份符合 `timelinemd`
格式的文件。
