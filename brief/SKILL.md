---
name: brief
description: 学习固定排版格式，为任意产品生成面向 X/Twitter 创作者（KOL）的 Launch Campaign Brief 文档（飞书或 Notion）。当用户说"写一个 brief"、"生成发布 brief"、"KOL brief"、"创作者合作简报"、"influencer brief"，或要为某个产品/模型发布快速产出创作者简报时使用。格式原型为《Kimi K3.1 Launch Campaign Brief for X Creators》（https://moonshot.feishu.cn/docx/UnPkdqkTBofnG2xtH3Sc7c7GnSe），但产品不限于 Kimi——为其他产品写 brief 时，需先去 X/Twitter 调研竞品与相关推文并用 astra-screenshot-guard 截图（只截推文卡片、防白框），作为 Best Practice 素材。文档结构固定：顶部注意事项 callout、成本速查表、产品介绍、创意方向（Quote Retweet / Single Post / Threads 三类创作者）、Use Case 素材库。依赖 lark-doc 完成飞书创建与媒体插入；输出 Notion 时用 ntn（Notion CLI），注意 callout 需 API 重建、图片定点插入需 curl 直调带 after 参数、占位符高亮需 API 遍历重写。
---

# Launch Campaign Brief 生成器（格式学习自原型文档）

按固定排版生成一份面向 X（Twitter）创作者的发布 Campaign Brief 飞书文档。**本 skill 学的是排版格式，不是只给 Kimi 用**——任何产品（模型、App、硬件、游戏等）都可以套这个格式。

格式原型：《Kimi K3.1 Launch Campaign Brief for X Creators》
https://moonshot.feishu.cn/docx/UnPkdqkTBofnG2xtH3Sc7c7GnSe

## 适用场景判断

- 给 **Kimi 自家产品**写 brief：产品信息（参数、benchmark、成本）通常用户会直接给，或可从内部文档拿；推文素材以自家 launch 推文 + 已有创作者案例为主。
- 给 **其他产品**写 brief：产品信息需要用户提供或公开调研；**Best Practice 推文截图必须去 X/Twitter 实地调研**——找该产品（或竞品）的高互动推文、创作者合作案例、对比测评帖，截图后插入对应位置。不要凭空编造推文内容。

## 触发与输入

用户调用时应尽量收集以下信息（缺失的用占位符，不要阻塞）：

| 信息 | 说明 | 缺失时 |
|-|-|-|
| 产品品牌名 + 型号 | 如 NovaAtom Geodesic | 品牌名必填；型号用绿色高亮占位 |
| 核心参数 | 参数量、上下文窗口、多模态等 | 绿色 `占位符` |
| Benchmark 成绩 | 2~3 项 | 绿色 `Benchmark占位符` |
| 成本速查表 | 各档 vibe coding 任务的成本 | 绿色 `价格占位符` |
| 对比竞品 | 如 Opus 5.5 / Astra | 沿用原型默认 |
| 价格倍数 | 比竞品便宜多少倍 | 沿用原型（7x / 15x） |
| 官方链接 | Tweet / 视频 / 博客 / 品牌资产 / 产品链接 | 留空或只给品牌资产 + 官网 |
| 图片素材 | 推文截图、benchmark 图、对比图 | 走 Step 0 调研截图，或留绿色 `图片占位符` |
| 视频素材 | 案例视频文件 | 见「视频插入」 |

## Step 0：X/Twitter 调研与截图（非 Kimi 产品必做）

为其他产品写 brief 时，先调研再动笔。目标是为以下位置收集真实推文截图：

- 成本速查表底部的 **Best Practice**（2 列 grid）：成本对比类推文
- A 类创作者 3 个子方向的 **Best Practice**（3 列 grid）：meme/quote 推文、use case 推文、模型对比推文
- C 类 Threads 主帖的品牌图/人物图（如有官方账号素材）

**调研流程（使用 kimi-webbridge 浏览器工具，走用户真实登录态）：**

1. `navigate` 打开 X 搜索：`https://x.com/search?q=<产品名 OR 竞品名>&f=top`（Top 标签找高互动帖）
2. 候选搜索词：产品名、产品名 + vs + 竞品名、产品名 + pricing/cost、竞品官方账号（`from:xxx`）
3. 用 `snapshot` 读推文内容，挑出互动量高、角度契合 narratives 的推文
4. **截图用 astra-screenshot-guard（推荐）或 WebBridge screenshot（兜底）**：
   - **首选 astra-screenshot-guard**：专门截推文卡片，确认视觉内容真正渲染完成才接受截图（防白框、防懒加载未完成的半图），只截推文 article 本身（不含浏览器边框和侧边栏）。用法见下方「推文截图：astra-screenshot-guard」
   - 兜底才用 WebBridge `screenshot`（带 `selector` 锁定推文 article 元素）——它是"等几秒就截"，X 的懒加载经常导致白框或半图，只在没有 guard 工具时用
5. 截图后在 Step 2 的 XML 中直接用 `<img path="@./<work_dir>/tweet-cost-1.png"/>` 替换对应绿色占位符，随创建一起上传

**注意：**

- 截图要完整包含推文作者、正文、互动数据；对比类推文连引用帖一起截
- 调研结果同时反哺内容：把真实推文里的价格、耗时、token 数填进示例文案，替换 `$xx` 占位
- 如果 X 登录态不可用或搜索无结果，不要编造截图——保留绿色占位符并在交付时说明
- 用户已提供截图素材时跳过本步骤

### 推文截图：astra-screenshot-guard

X 是 React SPA，文本先进 DOM、图片靠懒加载、视频 `preload=none`、后台标签页 rAF 被降频——所以"article 出现"不等于"内容可截"。astra-screenshot-guard 做的是：轮询确认图片有自然尺寸 / 视频 poster 已解码、连续多轮 readiness fingerprint 不变、双帧像素差检查、Pillow 检测近白区域，全部通过才接受截图，失败 fail-closed 不产出疑似错图。

```bash
# 准备 urls.txt（每行一个推文链接），然后：
python3 x_screenshot_guard.py \
  --input urls.txt \
  --out-dir ./x_screenshots \
  --session <独立session名> \
  --max-wait 30 --stable-rounds 4

# 或直接传 URL：
python3 x_screenshot_guard.py https://x.com/<user>/status/<id> --out-dir ./x_screenshots
```

- 输出：`001_<status_id>.png` + `manifest.json`（含每条的 readiness、媒体数、截图 QA、失败原因）
- **只把 `status=captured` 且 `mediaQA.ok=true` 的 PNG 交给后续上传**
- 前提：Kimi WebBridge 运行中且扩展已连接（`kimi-webbridge status` 看到 `running: true` + `extension_connected: true`）；建议装 Pillow 启用完整 QA
- 每条任务用独立 `--session` 名；默认结束关闭工具创建的标签页（`--keep-session` 保留）

## 执行流程（严格按序）

### Step 1：初始化草稿

先组装 Presentation Decision JSON（audience = X/Twitter 创作者，reader_task = 按 brief 产出合作内容，presentation_mode = rich），执行：

```bash
lark-cli docs +script --command init-draft --presentation-decision '<JSON>' --format json
```

记录返回的 `data.cwd`、`data.workspace`（记为 `work_dir`）和 `data.draft_path`。

### Step 2：生成 XML

读取 [`references/brief-template.xml`](references/brief-template.xml)，把其中的占位符替换为用户提供的真实信息，写入 `<data.cwd>/<draft_path>`。

**产品名替换规则（两段式，避免粘连）：**

- 模板中产品名固定写成两段：普通粗体品牌名 + 绿色高亮型号，即 `<b>NovaAtom </b><b><span background-color="rgb(98,210,86)">Geodesic</span></b>`
- 替换时先处理带 `Kimi K` / `Kimi ` 前缀的长组合，再处理单独的 `名字占位符`，最后才做 `Kimi` → 品牌名的全局替换；每步替换后 grep 检查是否出现 `KGeodesic` 这类粘连
- 型号未定时保留绿色 `名字占位符`，不要留空

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

XML 中的 `<img>` 和 `<figure><source/></figure>` 只在素材已就绪时使用（Step 0 截图或用户提供）；素材缺失时保留绿色文字占位符，创建后按以下方式补插：

**图片（三选一，按素材形态）：**

```bash
# 剪切板截图（用户说"我刚截图了"时优先）
lark-cli docs +media-insert --doc <document_id> --from-clipboard --align center

# 本地文件（含 Step 0 的调研截图）
lark-cli docs +media-insert --doc <document_id> --file ./tweet-screenshot.png --align center

# 网络图片：先 fetch 拿到目标位置 block_id，再精确插入
lark-cli docs +fetch --doc <document_id> --scope keyword --keyword "图片占位符" --detail with-ids
lark-cli docs +update --doc <document_id> --command block_insert_after \
  --block-id <blkID> --content '<img href="https://example.com/pic.png"/>'
```

注意：`+media-insert` 只能插在**文档末尾**；要插入到表格/grid 内的指定位置，必须用 `+update --command block_insert_after/block_replace` 配合 `<img href>` 或 `<img path>`。

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

返回文档 URL，并列出：哪些占位符仍需人工替换、哪些素材位还空着、Step 0 调研覆盖了哪些位置、插入失败的资源及原因。

## 输出到 Notion（替代飞书时）

skill 原生走 lark-cli 创建飞书文档。当用户要求写到 **Notion** 时，用 `ntn`（Notion 官方 CLI），但有几个坑必须绕开，否则排版会散。

### Notion 注意事项 callout 的处理

**问题**：`ntn pages create/edit` 走 markdown import，会把 callout 拆成一串普通 `quote` 块（每行一个，还夹空行 quote），且无法通过 API 把 quote 块改成 callout 类型（`PATCH` 报 `Block type mismatch: Changing a block's type is not supported`）。

**正确做法（按可靠性排序）：**

1. **整页按顺序重建（最可靠）**：先用 `ntn api` 把所有内容块按正确顺序 append——callout 作为第一个块用 API 插入（`type: "callout"`, `color: "blue_background"`, `icon: emoji`），再逐个 append 后续内容。操作前必须完整备份（`GET /v1/blocks/{id}/children` 存 JSON）。
2. **quote + blue_background 模拟（次选）**：markdown import 后用 `PATCH /v1/blocks/{quote_id}` 给 quote 块设 `color: "blue_background"`，视觉上是蓝底引用块，最接近 callout。用户在 UI 里全选 → Turn into → Callout 可 10 秒转正。

**绝对不要**：试图用 `PATCH` 把现有 quote 块直接改成 callout——API 明确拒绝改块类型。

### Notion 图片上传与定点插入

**上传（free plan 限制）**：workspace 是 free plan 时不支持 multipart，必须 `mode=single_part`（≤5MB）：

```bash
# 1. 创建 upload 对象
ntn api v1/file_uploads -X POST \
  -d '{"mode":"single_part","filename":"tweet.png","content_type":"image/png"}'
# 拿到 id

# 2. 上传文件内容
ntn api v1/file_uploads/<id>/send -X POST --file ./tweet.png
```

**定点插入（`after` 参数的坑）**：`ntn api -d` 会把 `after` 字段过滤掉，导致 append 永远到页面末尾。**必须绕过 ntn，用 curl 直调 API（复用 ntn 的 auth token）**：

```bash
TOKEN=$(ntn auth token | tail -1 | tr -d '[:space:]')
curl -s -X PATCH "https://api.notion.com/v1/blocks/<page_id>/children" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Notion-Version: 2022-06-28" \
  -H "Content-Type: application/json" \
  -d '{
    "children": [
      {"object":"block","type":"image","image":{
        "type":"file_upload","file_upload":{"id":"<upload_id>"},
        "caption":[{"type":"text","text":{"content":"说明文字"}}]}}
    ],
    "after": "<前置块_id>"
  }'
```

- 连续插入多张时，每插一张后页面索引会变，**下一张的 `after` 要用插入后的最新块 id**（重新 `GET children` 确认）
- 推文截图统一走 astra-screenshot-guard（见 Step 0），只截推文卡片，不用 WebBridge 整页截图

### Notion 占位符高亮

markdown 的 `==高亮==` 在 Notion import 时不生效。用 API 遍历所有文本块和表格单元格，把 `==xxx==` 替换为 `annotations: {"color": "green_background", "bold": true}` 的 rich_text 段：

- 普通块：`PATCH /v1/blocks/{id}` 重写该块的 `rich_text`
- 表格单元格：`PATCH /v1/blocks/{row_id}` 重写 `table_row.cells`（每个 cell 是 rich_text 数组）
- 禁令/强调用 `color: "red"`，警告用 `color: "yellow_background"`，实测数据用 `color: "red_background"`（粉底）

## 关键规则

- **不要**把绿色 `占位符` 高亮当成待办删掉——它是该文档的协作约定，留给运营同事填写。
- **不要**改变 callout / 表格 / grid 的颜色值，颜色即语义（浅蓝=信息区、浅红=成本表头、浅蓝底=示例表头、黄=警告、红=禁令/强调、绿=待填、橙=小标题、蓝=创意标题、粉底=实测数据）。
- **不要**编造推文截图或推文内容——非 Kimi 产品的 Best Practice 必须来自 Step 0 的真实 X 调研；调研不可行就留占位符。
- 示例推文截图的 `alt` 文本在创建时不必手写，飞书会自动生成；用 `<img href/path>` 即可。
- 文档为英文面向海外创作者，正文用英文；占位符保留中文（与原型一致）。
- 三类创作者结构 A/B/C 可按用户需要裁剪（比如只要 Single Post），但保留的段落内部结构不变。
- Use Case Library 没有素材时可以整节省略，或只留 h3 标题 + 占位符。
