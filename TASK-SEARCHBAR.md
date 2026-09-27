# Red Echo｜顶部搜索栏样式调整（执行规格）

## 项目背景
- /workspace 是 STATIC_SITE 小红书风格 iPhone 原型（无构建，纯静态：index.html / styles.css / app.js / data.js / images/）。
- 预览服务器：node .openclaw/scripts/serve.js 监听 3030（响应时把 CSS/JS/data 内联进 HTML）。改完文件后：`node --check /workspace/app.js && cd /workspace && redcowork-preview reload`。
- 验证地址（沙箱内网关）：`http://172.17.0.1:18789/redcowork/preview/yazhsjc0tx/?t=<每次唯一参数>`
- 当前 App 是**深色模式**（.screen 有 dark 类），默认主场景「摄影审美」（8 张结果），次场景「室内绿植养护」「云南雨季自由行」。
- QA 钩子：`window.__recho = {openSheet, closeSheet, showBubble, hideBubble, openDetail, closeDetail, BUB, sheetState, ctrl}`，`window.doSearch(query)`。
- 相关 DOM id：pageSearch（搜索中间页）、pageResults（结果页）、searchInput（中间页输入框）、btnSearch（中间页搜索按钮）、resBack（结果页返回箭头）、resBar（结果页搜索框）、resQuery（结果页关键词）、resClear（结果页 ✕ 清除钮）、resGo（结果页「搜索」按钮，可能不存在，按现状查）。
- 现状：结果页搜索栏是「返回箭头 + 胶囊框（✕+关键词）+ 框外搜索文字按钮」；搜索中间页有自己的输入框布局（看 index.html L97-L140 一带确认现状）。

## 用户需求（原话要点，参考图是唯一视觉基准）
以用户截图为唯一参考，统一修改搜索中间页和搜索结果页的顶部搜索栏：
1. 左侧独立返回箭头（位置尺寸间距参照截图）。
2. 搜索框：横向长圆角矩形、**浅灰背景**（深色模式下应为深灰 #1e1e22 之类，以截图在深色模式下的实际观感为准）、宽度自适应。
3. 搜索文字左对齐显示当前关键词（如「室内绿植养护」），**文字前不放放大镜图标**。
4. 清除按钮：搜索框内右侧圆形 × 图标，点击清空关键词允许重新输入。
5. 搜索按钮「搜索」两字：**位于同一个圆角搜索容器内部**，与输入区之间用**细竖线**分隔，与清除按钮保留间距。
6. 高度、圆角、文字大小、内边距参照截图，各元素垂直居中。

交互要求：
- 点击搜索框可直接编辑关键词（真机是点击后聚焦输入）。
- 点击 × 清空输入。
- 点击「搜索」或键盘搜索键执行搜索。
- 点返回箭头回上一级。

**禁止**：加放大镜图标；把「搜索」按钮放搜索框外面；改搜索数据、Bubble、瀑布流或其他页面。

## 参考图
用 image 工具看这张图（深色模式小红书真机截图）：
https://ep-redcowork-s1.xhscdn.com/ep-redcowork/1040g4o0325jdkva5ko01qaqj926jdg01g72preg.png?sign=917db7902bf9aa211239ecba69831259&t=6abff498
先精确分析它的布局（返回箭头/容器结构/✕位置/竖线分隔/深色配色），再动手。

## 实现要点（以现状代码为准，别臆测）
1. **结果页（#pageResults 的 .sbar）**：重构成 `[返回箭头按钮][圆角容器：左=关键词区+右侧✕ + 细竖线 + 「搜索」按钮]`。容器一个整体圆角胶囊，内部用 1px 竖线（深色模式 rgba(255,255,255,.15)）分隔输入区和搜索按钮。✕ 在关键词右侧、竖线左侧。
2. **搜索中间页（#pageSearch）**：同样结构。注意中间页可能已有输入框（#searchInput）——把它挪进统一容器结构，保持 id 不变（app.js 绑定用 id），或同步改绑定（改绑定要把 app.js 版本号+1）。
3. 点击搜索框/关键词区 → 聚焦输入（中间页）；结果页点击 → backToSearch(true)（回到中间页可编辑，现有逻辑）。✕ → 清空输入（两页都要有）。「搜索」→ 执行当前关键词（doSearch）。
4. 深色模式适配：容器深灰底（比页面基底 #121214 亮一档，如 #1e1e22）、文字浅色、竖线半透明白。浅色模式样式也别坏（浅灰底 #f5f5f7）。
5. 改动涉及 styles.css（搜索栏区块）+ index.html（两个页面的 sbar 结构）+ 可能 app.js（绑定微调）。每改一个文件版本号 +1（index.html 里 ?v=N）。

## 验收（必须逐项实测）
1. 浏览器打开结果页截图：结构 = 返回箭头 + 浅灰胶囊容器（关键词左对齐 + 圆✕ + 细竖线 + 「搜索」），无放大镜。
2. 中间页截图：同样结构，输入框可编辑。
3. 功能四项：
   - 点击搜索框（结果页→回中间页可编辑；或直接可编辑，按你的实现）。
   - 点 ✕ → 输入清空。
   - 输入新词（如「室内绿植养护」）→ 点「搜索」→ 结果页出现 8 卡绿植结果 + Bubble 球（4 篇）。
   - 点返回箭头 → 回上一级（中间页或默认页，按层级逻辑）。
4. 回归：默认场景「摄影审美」打开正常（8 卡+球）、深色模式正常、window.onerror 无报错。
5. 提供最终运行截图（结果页 + 中间页各一张，路径列出）。

## 工具坑位
- evaluate fn 禁顶层 await（用 (async()=>{})() 包裹返回 Promise）。
- browser act click 不支持 selector → 用 evaluate element.click() 或 snapshot ref。
- navigate 同 URL 空操作，带唯一 ?t=。
- 截图落 /workspace/media/browser/，`ls -t | head -1` 取最新。
- 视觉模型小图判读不可靠，几何争议以 getBoundingClientRect 为准。
- 页面切换机制：pageResults 加/去 'in' 类（覆盖式），pageSearch 永远在底层——检测页面状态用 pageResults.classList 而不是 pageSearch.className。

## 完成后
1. memory/2026-09-26.md 末尾追加「## 顶部搜索栏统一重构（2026-09-27 02:20）」：改动文件、新结构说明、四项功能验证结果、截图路径。
2. 输出报告（≤20 行）：改动文件+版本、新搜索栏结构、四项功能结果、截图路径、遗留问题。
