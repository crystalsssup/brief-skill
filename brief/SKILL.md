---
name: brief
description: 生成 Kimi 产品发布 Campaign Brief 飞书文档（面向 X/Twitter 创作者的合作简报）。当用户说"写一个 brief"、"生成发布 brief"、"Kimi K 系列 launch brief"、"创作者合作简报"或要基于某次模型发布快速产出 influencer brief 文档时使用。文档格式固定：顶部注意事项 callout、成本速查表、产品介绍、创意方向（Quote Retweet / Single Post / Threads 三类创作者）、Use Case 素材库。依赖 lark-doc 完成创建与媒体插入。
---

# Kimi Launch Campaign Brief 生成器

按固定排版生成一份面向 X（Twitter）创作者的发布 Campaign Brief 飞书文档。原型为《Kimi K3.1 Launch Campaign Brief for X Creators》。

## 触发与输入

用户调用时应尽量收集以下信息（缺失的用占位符，不要阻塞）：

| 信息 | 说明 | 缺失时 |
|-|-|-|
| 产品名 | 如 Kimi K3.1 | 用 `Kimi K<名字占位符>` 绿色高亮占位 |
| 核心参数 | 参数量、上下文窗口、多模态等 | 绿色 `占位符` |
| Benchmark 成绩 | 2~3 项 | 绿色 `Benchmark占位符` |
| 成本速查表 | 各档 vibe coding 任务的成本 | 绿色 `价格占位符` |
| 对比竞品 | 如 Opus 5.5 / Astra | 沿用原型默认 |
| 价格倍数 | 比竞品便宜多少倍 | 沿用原型（7x / 15x） |
| 官方链接 | Tweet / 视频 / 博客 / 品牌资产 / 产品链接 | 留空或只给 Brand Asset + Try Kimi |
| 图片素材 | 推文截图、benchmark 图、对比图 | 绿色 `图片占位符`，创建后由用户替换 |
| 视频素材 | 案例视频文件 | 见「视频插入」 |

## 执行流程（严格按序）

### Step 1：初始化草稿

先组装 Presentation Decision JSON（audience = X/Twitter 创作者，reader_task = 按 brief 产出合作内容，presentation_mode = rich），执行：

```bash
lark-cli docs +script --command init-draft --presentation-decision '<JSON>' --format json
```

记录返回的 `data.cwd` 和 `data.draft_path`。

### Step 2：生成 XML

读取 [`references/brief-template.xml`](references/brief-template.xml)，把其中的占位符替换为用户提供的真实信息，写入 `<data.cwd>/<draft_path>`。

**占位符规则（必须保留的排版语义）：**

- 未确定的产品名 / 参数 / 价格 / benchmark：保留 `<b><span background-color="rgb(98,210,86)">xxx占位符</span></b>` 绿色高亮，方便后续人工替换。这是本格式的核心约定，不要改成普通文本。
- 强提醒（先发草稿审核、不要打 paid partnership 标、必须写成本等）：红字 `text-color="rgb(216,57,49)"`，常叠加黄色高亮 `background-color="rgba(255,246,122,0.8)"`。
- 小标题强调条：橙色高亮 `background-color="rgba(254,212,164,0.8)"`。
- 创意方向小标题：蓝字 `text-color="rgb(36,91,219)"`。
- 成本对比示例中的实测数据：粉底 `background-color="rgb(251,191,188)"`。

**排版结构（与原型一致，顺序不可乱）：**

1. `<title>`：🌖 + 产品名 + ` Launch Campaign Brief for X Creators`
2. 浅蓝 callout（emoji 🔆，`background-color="rgb(240,244,255)"` `border-color="rgb(186,206,253)"`）：称呼 + 感谢 + ⭐️ Please note 四条注意事项（`&gt;` 引导）
3. 成本速查表：两行表头（浅红底 `rgb(254,241,241)` 的 h4 标题行 + 黄色提醒行）+ 成本档位行 + 红字价格倍数 ul + Best Practice 行（grid 双列推文截图）
4. `<h1>` 🌍 Introducing + 产品名：一段加粗简介 + 浅蓝 callout（emoji 📎）放 Official Links 列表
5. `<h3>` 🌟 Model Highlight：嵌套 ul（Open Source SOTA / Performance / Coding / Knowledge Work / Price）
6. 三列 grid 放 benchmark 占位 + 居中价格图占位
7. Feature/Showcase 二列表格（表头浅蓝底 `rgb(225,234,255)`）
8. `<h1>` 🎥 Creative Directions（红字标注 please do NOT just copy）
9. 浅蓝 callout（emoji 💡）：Narratives 有序列表（5 条，标题蓝字，细节用 u 下划线强调）
10. `<h2>` A/B/C 三类创作者（标题加 u 下划线，之间用 `<hr/>` 分隔）：
    - A. Quote Retweet：3 个 h4 子方向（自动编号 `seq="auto"`），每个含 Message/Form ul + "e.g. (please do NOT just copy)" + 示例表格（表头浅蓝底，Best Practice 列用 grid 多列放推文截图）
    - B. Single Post：5 个 h4 子方向，Must Include 用黄色高亮，示例表格 Main Post 列内嵌 `<figure><source/></figure>` 视频
    - C. Threads：1 个 h4，示例表格含 Main Post + 多个 Sub thread 行
11. `<h1>` 📖 Use Case Library：按类别（Threejs/Shader、AAA Games、Computer Use 等）用 `<h3>` 分组，每组用 grid 多列排视频

### Step 3：校验

```bash
lark-cli docs +script --command parse --content "@./<draft_path>" --format json
```

按 `data.diagnostics[]` 修复，直到 `data.assessment.status` 通过。

### Step 4：创建文档

```bash
lark-cli docs +create --doc-format xml --content "@<draft_path>" --as user
```

返回 `document_id` 和 `url`。检查 `warnings`。

### Step 5：插入图片与视频

XML 中的 `<img>` 和 `<figure><source/></figure>` 只在用户已提供素材时使用；素材缺失时保留绿色文字占位符，创建后按以下方式补插：

**图片（三选一，按用户素材形态）：**

```bash
# 剪切板截图（用户说"我刚截图了"时优先）
lark-cli docs +media-insert --doc <document_id> --from-clipboard --align center

# 本地文件
lark-cli docs +media-insert --doc <document_id> --file ./tweet-screenshot.png --align center

# 网络图片：先 fetch 拿到目标位置 block_id，再精确插入
lark-cli docs +fetch --doc <document_id> --scope keyword --keyword "图片占位符" --detail with-ids
lark-cli docs +update --doc <document_id> --command block_insert_after \
  --block-id <blkID> --content '<img href="https://example.com/pic.png"/>'
```

注意：`+media-insert` 只能插在**文档末尾**；要插入到表格/grid 内的指定位置，必须用 `+update --command block_insert_after/block_replace` 配合 `<img href>` 或先上传再用 `src="token"`。

**视频：** 视频无法通过 `href` 上传。本地视频文件用：

```bash
lark-cli docs +media-insert --doc <document_id> --file ./case.mp4 --type file
```

或先在 XML 中写 `<figure view-type="Preview"><source path="@./video.mp4" name="case.mp4"/></figure>` 随创建一起上传（推荐，可精确定位到表格/grid 内）。

**替换占位符：** 插入真实素材后，用 `str_replace` 清掉对应绿色占位文字：

```bash
lark-cli docs +update --doc <document_id> --command str_replace --pattern "图片占位符" --content ""
```

### Step 6：交付

返回文档 URL，并列出：哪些占位符仍需人工替换、哪些素材位还空着、插入失败的资源及原因。

## 关键规则

- **不要**把绿色 `占位符` 高亮当成待办删掉——它是该文档的协作约定，留给运营同事填写。
- **不要**改变 callout / 表格 / grid 的颜色值，颜色即语义（浅蓝=信息区、浅红=成本表头、浅蓝底=示例表头、黄=警告、红=禁令/强调、绿=待填、橙=小标题、蓝=创意标题、粉底=实测数据）。
- 示例推文截图的 `alt` 文本在创建时不必手写，飞书会自动生成；用 `<img href/path>` 即可。
- 文档为英文面向海外创作者，正文用英文；占位符保留中文（与原型一致）。
- 三类创作者结构 A/B/C 可按用户需要裁剪（比如只要 Single Post），但保留的段落内部结构不变。
- Use Case Library 没有素材时可以整节省略，或只留 h3 标题 + 占位符。
