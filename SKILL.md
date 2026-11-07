---
name: yunfeng-gzh
description: Yunfeng的折腾笔记（公众号作者"云峰"）的公众号文章全流程流水线。当用户要求"走一遍公众号流程""修订+排版+题图一条龙""按流程处理这篇文章"时使用。覆盖四步：1) 按其文风轻度修订 Markdown 原稿；2) 生成 3 个候选标题供选择；3) 调用 gzh-design skill 排版为公众号 HTML 并输出预览页；4) 调用 ian-xiaohei-illustrations skill 生成两张题图（2.35:1 头条封面 + 1:1 次条封面）。单独的修订、排版或题图请求不必用本 skill，直接用对应的单项能力即可。
agent_created: true
---

# 公众号写作流水线

## Overview

把一篇Markdown 原稿加工成可直接发布的公众号内容：修订 → 标题 → 排版 → 题图，四步一条龙。用户随时可查看、可跳步（用户只要求其中某几步时，只跑那几步）。

**前置事实（每个会话都适用）**：
- 用户：公众号署名一律用"云峰"。
- 工作区：Obsidian 库 `D:\Ivan\Markdown`，文件名为中文长标题；图片多为双链 `![[xxx.png]]`，实际文件在 `D:\Ivan\Markdown\assets\`。
- 文风档案：`D:\Ivan\Markdown\我的写作风格描述.md`——开工前必须读它。
- 长期偏好细节见 `~/.workbuddy/MEMORY.md`。

## 产物落盘位置（用户的硬约定，2026-09-24 起）

vault 根目录**只留 `.md`**。所有非 Markdown 产物一律落进 `D:\Ivan\Markdown\output\`：

| 产物 | 落盘位置 |
|------|----------|
| 修订稿（原地改） | `D:\Ivan\Markdown\<原文件名>.md`（就在原处） |
| 原稿备份 | `D:\Ivan\Markdown\<原名>_原版.md`（`.md`，留在原处） |
| 干净正文 HTML | `D:\Ivan\Markdown\output\<文章简称>_排版_<主题中文名>(<主题标识>).html` |
| 预览页 HTML | `D:\Ivan\Markdown\output\公众号预览_<文章简称>.html` |
| 题图（**例外**） | `D:\Ivan\Markdown\assets\<文章简称>-illustrations\`——`assets\` 已有 4 个同款目录（15年前老爷机 / C盘组合拳 / omarchy-linux / 优秀演讲三法则），题图留在那里保持既有结构，不要挪进 output |

- `output\` 不存在时先创建（`fs.mkdirSync(p, { recursive: true })`）。
- 搬错名残留、清历史遗留 → 一律移进 `D:\Ivan\Markdown\.trash\`，**不要直接删**（用户可从 Obsidian 回收站还原）。
- 交付前用 Node 独立列一遍 vault 根目录，确认非 md 文件为 0（`assets` / `Excalidraw` / `.obsidian` / `.trash` / `.workbuddy` / `.agents` / `output` 是目录，不算）。

## 流程总览

```
原稿.md ──► ① 文风修订（备份→轻度编辑→改动清单）      → 就地改 .md + `_原版.md`
       ──► ② 3 个候选标题（AskUserQuestion 选定）
       ──► ③ gzh-design 排版（HTML + 校验 + 预览页）    → output\
       ──► ④ guizang 封面对（2.35:1 + 1:1）             → assets\<简称>-illustrations\
```

默认顺序执行，但 ④ 的题图只依赖文章主题不依赖排版结果，若图片生成额度紧张或用户催进度，③④ 可并行。

## Step 1：文风修订（轻度编辑，底线是不重写）

1. **先备份**：原稿复制为 `<原名>_原版.md`（用户习惯后缀）。
2. **读两份文件**：原稿全文 + `D:\Ivan\Markdown\我的写作风格描述.md`。
3. **只做轻度编辑**，改动范围白名单：
   - 错字、的/得/地、标点（中文并列用顿号）
   - 弯引号统一（`""''`），中英文之间加空格
   - 专名一致性（如 ThinkPad、WPS Office、Zorin 大小写）
   - 明显冗余、重复用词、多余空行
   - 逻辑硬伤（如"Wine 在 Windows 下运行"这类事实错误）——改，但在清单里单独标出
   - 列表符号统一为 Markdown 格式
4. **红线**：不重写句子、不调句序、不动图片链接、正文不加 H1（标题归发布平台）、`#` 一级标题降为 `##`。
5. **事实存疑处不擅自改**（拼写存疑的产品名、查不到的数据），列出来让用户确认。
6. **输出改动清单**：分类列（错字语病 / 逻辑修正 / 名称格式统一 / 存疑待确认），逐条编号，让用户能对照。
7. 结尾若明显收得突然，可主动提议补一句回扣开头的收束，经确认再写。

## Step 2：生成 3 个候选标题

1. 基于修订后的正文拟 3 个标题，路子限于用户接受的钩子型：数据钩子、配置反差钩子、热点钩子、"老爷机再战五年"式短句直给。
2. 用 AskUserQuestion 一次让用户选定，推荐项放第一位并标"(推荐)"。
3. 选定的标题用于排版 HTML 的开篇标题；同时记住它，Step 3 排版标题区使用。
4. **选定后立即用文字把最终标题完整回显一遍**（加书名号，如《……》），并在后续每次向用户汇报产物时都带上它——用户要求：确定的标题必须在输出里可见，不能只留在选项记录里。

## Step 3：gzh-design 排版

**产物路径（本步全部 HTML 都落 `D:\Ivan\Markdown\output\`，不要写进 vault 根目录）**：

- 干净正文 → `output\<文章简称>_排版_<主题中文名>(<主题标识>).html`
- 预览页 → `output\公众号预览_<文章简称>.html`

1. 调用 `gzh-design` skill，按其规范选主题。默认推荐"石墨极简风"（与用户气质一致），但要用 AskUserQuestion 让用户在 2-3 套主题里选，别擅自定。
2. **图片必须内嵌**：扫描原稿中的 Obsidian 双链图（`![[xxx.png]]`），到 `D:\Ivan\Markdown\assets\` 找到文件，按文档顺序读出、转 base64 填进 HTML 的 `<img src>`（data URL）。**绝不留 `图片URL` 占位**。读不到文件才提醒用户补。
3. **签名区固定文案**（每篇的最后段落，逐字使用）：
   - 第一句：`我是云峰，平时喜欢折腾工具、记录方法，也写点踩坑和翻车。`
   - 第二句：`如果你觉得今天这篇有收获，欢迎 点赞、在看、转发 三连，我们下篇见`（"点赞、在看、转发"加粗，句尾无句号）
4. **校验**：跑 `gzh-design` 的 `validate_gzh_html.py`，要求 0 ERROR / 0 WARNING。
5. **⚠️ 图片块必须显式写 `text-align:center`（2026-11-01 实测坑，不可省）**：所有包 `<img>` 的容器（外层 section、figure、内层 section 三层都要写），否则浏览器把渲染内容复制进剪贴板时会按**计算值**序列化，未声明对齐的块被写成 `text-align:start`——粘贴到公众号编辑器就弹「内容结构检测：文字对齐异常（参考文档#1.6 text-align）」。实测《墨水屏》篇两个无对齐图片块恰好对应两条告警（第 14 段、第 26 段）。铁律：**产物里 text-align 只允许 `justify` 和 `center` 两个值；左对齐靠默认，右对齐用 flex，绝不写 `left`/`right`/`start`/`end`**（WebKit/Blink 会把 left 序列化成 start、right 序列化成 end）。修复时排版源文件和预览页两个文件都要改。
6. **生成预览页（必须用本 skill 的 Node 脚本）**：跑 `scripts/build_preview.js`，它读 `gzh-design/assets/preview-template.html` 包装出带「复制到公众号」按钮的预览页，**并且由 Node 写最终中文文件名**。
   - **不要用 `gzh-design` 的 `wrap_preview.py` 出最终文件**：实测 Python 往含中文的目标路径写文件时，落盘文件名会被环境改写——`公众号预览_C盘组合拳.html` 写出去躺在磁盘上变成了 `C盘占用太大？用这套组合拳进行操作.html`，而脚本自己还回显"✓ 已生成"。同一次运行里 PIL 存图也被改过名。
   - 用法：先 Write 一个 **ASCII 文件名**的 job.json 到 `.workbuddy\`（字段：`template` 模板路径 / `content` 干净正文路径 / `out` 目标预览路径（写进 `output\`）/ `title` / 可选 `stray` 错名残留数组与 `trashDir`），再
     `node <SKILL_ROOT>\scripts\build_preview.js <job.json>`。
   - 脚本内部：写 ASCII 临时文件 → `fs.renameSync` 成正式中文名 → 自校验，结论落在 `.workbuddy\_preview_report.txt`（文件名 / `<title>` / 图片数 / 复制按钮 / 目录内含"预览"的文件列表）。
7. **命名复核（不可省）**：
   - 最终文件：`output\公众号预览_<文章简称>.html`（不含逗号、括号、全角标点），HTML 内部 `<title>` 同名。
   - **必须用 Node 重新 `fs.readdirSync` 列一遍 `output\` 和 vault 根目录，确认那个名字确实在磁盘上、根目录没有非 md 残留**——不能只看脚本回显。发现被改错名的残留文件，移进 `.trash`，不要直接删。
8. 排版完成后告知：结构做了哪些自拟章节（原文小节不够时需自拟并交待）、数据是否做了卡片/表格化处理。

## Step 4：公众号题图（guizang 封面对，2026-10-29 起）

1. **调用 `guizang-social-card-skill`**（本地路径 `C:\Users\Ivan\.workbuddy\skills\guizang-social-card-skill`），走其"公众号封面对 / WeChat cover pair"分支；动手前必读它的 `references/platform-specs.md`、`references/style-system.md`、`references/theme-presets.md`、`references/title-shortener.md`、`references/production-workflow.md`。
2. **固定产出两张**（同一 HTML 内两块 `.poster` + 组合预览区）：
   - **头条封面 `.poster.wide`**：2100×900（21:9），放完整或近完整标题、副题和一个强视觉元素。
   - **次条封面 `.poster.square`**：1080×1080，用 `title-shortener` 的五步法从长标题提炼短标题，**不要从 21:9 直接裁切**（skill 硬规则）。
3. **风格与主题用 AskUserQuestion 让用户选**（Editorial / Swiss 二选一，10 套主题预设内选，不允许自定义 hex）。默认推荐 Editorial「墨水经典 ink-classic」（与排版端"石墨极简"气质一致）；深色/影调题材推荐「午夜墨 midnight-ink」；数据/测评/方法论类可推荐 Swiss + IKB 蓝。别擅自定。
4. **渲染方式**：README 提到的 `render.mjs` 仓库里**没有**——按 `production-workflow.md` 自写一次性 Playwright 截图脚本，按节点 id 截 `#wechat-21x9`、`#wechat-1x1`、`#wechat-pair-preview`，等待 500-900ms 让字体/画布就绪。任务目录用 **ASCII 名**（放 `<工作区>\.workbuddy\gscs-<slug>\`），**勿在 skill 根目录或 vault 里建任务目录**（skill 非协商规则）。首次使用前在 skill 目录 `npm install` + `npx playwright install chromium`。
5. **⚠️ 比例硬修正（不可省，平台判严格 2.35:1=47:20）**：guizang 的 wide 是 21:9（2.3333），直接上传头条封面会被判比例不符。渲染后必须后处理：把 2100×900 **等比放大到宽 2115**（→2115×906），再**垂直居中裁到 2115×900**（精确 2.35:1）。裁前检查页脚/页码没被切掉，被切就在海报 CSS 加底部余量重渲。1:1 的 1080×1080 不用动。落盘前用 Node 读 PNG IHDR 复核宽高。像素处理用 Pillow/Node，只写 ASCII 临时路径（见环境坑铁律）。
6. **落盘**：任务目录里的 PNG 用 **Node `fs.copyFileSync` + `fs.renameSync`** 改成正式中文名，进 `D:\Ivan\Markdown\assets\<文章简称>-illustrations\`（如 `01-<简称>-235x1.png`、`02-<简称>-1x1.png`），沿用目录既有命名结构。
7. **校验节奏按 skill 默认**：先出图给用户看，用户明确要查时才跑 `node validate-social-deck.mjs <task-dir>`，别自动跑（每轮多花几十秒）。
8. 汇报时描述画面构思与主题选择理由，并主动提供"不满意可换版式/主题重渲"的选项。

**小黑保留条款**：`ian-xiaohei-illustrations` 不再用于本流水线的默认题图，但**不废弃**——用户点名要小黑风（正文配图、其他场景插画、或点名"题图用小黑"）时照常走它，原流程（1536×1024 生成→裁精确 1410×600、白点拉伸等）全部不变。用户没点名且要求的是"题图/封面"时，一律走本步的 guizang 封面对。

## 最终交付汇报（固定项，不可省）

收尾汇报里必须包含以下内容，逐项列出，不要只甩文件：

1. **最终标题**：书名号完整写出用户选定的标题（如《……》）——这是用户的明确要求，每次都要回显。
2. 四步各自的产物清单：修订改动要点、排版结构与校验结果、封对面（题图）画面/版式描述与路径。
3. 存疑待确认项（没动、等用户拍板的内容）。
4. 发布操作一句话指引（预览页 → 复制 → 粘贴）。


## 环境坑（Windows，务必遵守）

- **铁律：最终文件名一律交给 Node 写（`fs.writeFileSync` / `fs.renameSync`），Python 只许写 ASCII 临时路径。** 这条比下面所有条目都重要。本机 Python 向含中文的目标路径写文件时，落盘文件名会被环境改写（已复现两次：预览页 `公众号预览_C盘组合拳.html` → `C盘占用太大？用这套组合拳进行操作.html`；题图 `01-c-drive-combo-235x1.png` → `C盘占用太大？用这套组合拳进行操作.png`），而且**Python 不报错、脚本回显成功**——只能靠事后用 Node 列目录复核。已把这条经验做成 `scripts/build_preview.js`。
- **「下载管理器」黑手（2026-10-03 破案）**：有个后台进程会拦截新写入的文件，按"当前文章主名"重新发出，重名加 `-1`（`文章名.html`、`文章名-1.html`）；连 `.workbuddy\` 下的 ASCII 副本也会被搬到 vault **根目录**且路径分隔符被吃掉（`.workbuddy\tmp.html` → 根目录 `.workbuddytmp.html`）。对策：① Node 写完立刻核内容（字节数 + 占位符 + 关键字），不能只看存在性；② 发现产物消失，先扫 output / 根目录 / .trash 找同尺寸文件——好坏两版常同时躺在 `文章名.html` 和 `文章名-1.html` 下，按大小+内容特征辨认后改回正式名，别急着重新生成；③ 确认不要的错名副本进 `.trash`。
- **IMG 占位符在模板中出现多次时**，内嵌必须用 `t.split(placeholder).join(b64)`——`t.replace()` 只换第一处，封面缩略图复用正文图就会留占位残留。写完后校验 `placeholders left: none`。
- **产物落 `output\`，vault 根目录只留 `.md`**（见上方「产物落盘位置」）。搬移/清理一律用 `fs.renameSync` 进 `output\` 或 `.trash\`，**不要用 `rm`**。收尾时用 Node 列一遍根目录，非 md 文件应为 0。
- **搬完文件必须独立复核目录，不能把脚本自己的输出当结论。** 2026-09-24 把根目录非 md 搬进 `output\`：脚本报告"已移入 10 个、根目录只剩 .md"，事后复核发现另有 3 个 `15年前的老爷机，用Zorin OS焕发新生*.zip`（5KB 级文章导出包）在两次操作之间从整个库里消失——不在 `output\`、不在 `.trash\`、不在回收站、不在临时目录。受控实验证明 `.zip` 在本环境可正常 `renameSync`，且脚本每移一个都会 push 进清单（清单只有 10 条），故不是移动脚本所为；消失原因未查明。**教训：脚本报告后仍要重新列目录对账，"清单条数 == 预期条数"才算过，数量对不上就立刻停下报给用户，别脑补。**
- **中文路径传给 Python 命令行会乱码/静默失败**：PowerShell 和 bash shim 都会。对策：把路径硬编码进 UTF-8 编码的 Python 包装脚本（写到 `<工作区>\.workbuddy\` 下），用 `python 包装脚本.py` 执行。要传参就用 **ASCII 名的 JSON job 文件**，别把中文塞进 argv。
- **Write 工具写超长中文文件名（含全角逗号）可能截断后缀**：产物文件名保持简短、无全角标点。实测 `_排版_石墨极简风(graphite-minimal)` 这一整段会被砍掉，只剩 `文章名.html`。对策：产物先落盘、再用 Node 脚本 `fs.renameSync` 改成正式长文件名。
- **超长 HTML 别直接 Write**：内嵌 6 张 base64 图后有 1.5MB+。对策：Write 只写带 `{{IMG1}}…{{IMG6}}` 占位符的模板，再跑一个 Node 脚本从 `assets\` 读图、替换成 data URL、写回。
- **Pillow 的 venv 入口是 `...\envs\default\Scripts\python.exe`**，不是 `...\envs\default\python.exe`（后者会 ModuleNotFoundError: PIL）。
- **PowerShell 工具在本环境不回显 stdout**：需要看输出时，把结果 `Out-File` 到 `.workbuddy\` 下的临时文件再用 Read 读；文件操作优先用 Node 而不是 PowerShell（PowerShell 的 `Remove-Item` 在本环境还常被沙箱拦下）。
- **ImageGen 出图底色偏灰不是纯白**：落盘前用 Pillow 做白点拉伸（取亮度直方图 97 分位作白点线性拉伸，≥244 直接压 255），线条不受影响。裁 2.35:1 后检查上下边缘——被裁掉一半的红色/橙色标注残留要局部抹白。
- **校验脚本不回显时，把输出重定向到文件再 Read，别反复盲跑。**
- **guizang-social-card-skill（2026-10-29 接入）**：装在 `C:\Users\Ivan\.workbuddy\skills\guizang-social-card-skill`（gh 直连会 502，用 `gh-proxy.com` 前缀镜像下载 zip 后解压，目录已去掉 `-main` 后缀）。依赖 playwright：首次渲染前在 skill 目录 `npm install` + `npx playwright install chromium`。README 提到的 `render.mjs` 仓库没带——按 `references/production-workflow.md` 自写截图脚本。任务目录一律 ASCII 名（放工作区 `.workbuddy\gscs-<slug>\`），PNG 渲染出来后由 **Node** 改中文名落 `assets\<简称>-illustrations\`；头条封面必须做 21:9→2.35:1 的放大裁切（2100×900 → 2115×906 → 裁 2115×900），别直接传 21:9。
