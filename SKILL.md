---
name: create-homepage-skill
description: Build or redesign a distinctive personal homepage / portfolio (个人主页 / 作品集) for a real person, from their brief through a runnable base, reference-driven iteration, verification, and public deployment. Use when someone wants a memorable personal site ranging from restrained static editorial design to scroll storytelling and micro-interactions, NOT a generic template. Preserves an existing site's stack during redesign and chooses the smallest suitable stack for new builds; the React + Vite + framer-motion + Canvas path is reserved for interaction-rich work. Ships a portable "connective layer" of design tokens, motion roles, reduced-motion behavior, and performance rules while the bespoke structure and visual language grow from that person's own evidence and references.
---

# Create Homepage Skill

把**一个真实的人**做成一个**有世界观、会呼吸、能上线**的个人主页。

这套流程不是凭空提炼，是从一次真实的 9 天创作（先做能跑的基础版 → 一轮轮按反馈和参考迭代 → 上线）里抽出来的。它的两个支点：

- **一套与隐喻无关的「连接层」**（设计 token + 可移植动效原语 + 三层减动效/性能契约）——保证任何人一上来就有**灵动、协调、不空洞**的底子。
- **一层专属外壳**（隐喻**或**直白编辑型方向、版式结构、视觉语言、文案）——每个人从**自己的材料和参考**里长出来，绝不预置成模板。

底子由 skill 保证质量，外壳由这个人保证独特。这就是它既"好看灵动"又"每个人都不同"的原因。

## 铁律（先读，贯穿全程）

1. **接住 brief，别当白纸访谈。** 用户通常带着身份、大致方向、甚至技术栈和材料就来了——先**接住并复述**你听到的（"你是 X，想要 Y 气质，5 个板块……对吗？"），别假装他是白纸、从零开始盘问。同时**探测你所在环境里可复用的 skill**（设计/动效/文案；环境无 skill 机制就跳过），少造轮子。见 `references/interview.md`。
2. **先选最小合适技术路径，再出能跑的基础版（base-first）。** 改现有站默认保留原栈，不为套本 skill 强制迁移；新站按任务复杂度选静态/轻交互或 React 强交互路径。只需先确认**硬约束**（脱敏清单、技术路径、语言语气），就用连接层快速搭一个**已经协调、能跑**的基础站（内容可先用简历占位）——让用户第一眼看到的是好东西，不是线框。视觉/动效/隐喻在后面的迭代里收敛。路由见 `references/build-verify-deploy.md`。
3. **隐喻先找后造，且隐喻可选。** 顺序是：① 先问用户有没有想模仿的**参考**（站链接/截图/现成组件代码）——真实项目里那个"航行"世界观就是用户自己贴了 `paperplanes.world` 才有的；② 没有就引导他去 Awwwards / reactbits / paper 挑一个**带回**；③ 到"共创"这步**先判断这个人需不需要隐喻**：职业名称本身不算证据，必须有个人经历、作品材料、受众任务等至少两个彼此独立的信号，且隐喻能帮人看懂，才共创 2–3 个候选，并同时保留一个不靠隐喻的编辑型候选；撑不住就直接走**编辑型方向**（最强证据定内容主次 + 有性格的排版 + 一个签名交互）。**别为了"要有个概念"硬凑世界观**；两条路都过铁律 10 的三层自检。参考只用来校准手感与气质，不是抄版式。
4. **动效从库现取 + 套连接层，但不为动效反选重栈。** 先按任务选技术路径，再决定是否需要动效；静态/轻交互路径用 CSS 或少量原生 JS，React 强交互路径才按需从 `reactbits.dev` / `shaders.paper.design` 取。看懂原理后移植进已选栈，**不预置固定动效引擎**。每个取来的动效都必须过连接层：统一 token，并补上减动效降级与性能保护。见 `references/motion-kit.md`、`references/inspiration.md`。
5. **证据大于修饰，主动要真实源文档。** 个人站最露馅的是通用套话、不是布局。主动向用户要**真实材料**（项目结题 PPT、会议纪要、合同、live 数据源如 GitHub star / App 评分），从里面抠**可量化成果**和**诚实的角色边界**（做了什么、不夸大），砍掉凑数的平淡条目。所有正文定稿前过一遍去 AI 腔检查——装了 `avoid-ai-writing` 这类 skill 就用它，没有就照其原则自查。
6. **减动效必须全静态降级（按技术路径覆盖对应层，无障碍底线）。** 每个动效都要有 `prefers-reduced-motion` 的静态回退：CSS 层（`@media` 中和所有动画）是底线、任何路径都要；有 JS 动效再加 JS 层（`useReducedMotion` 返回静态元素）、有 Canvas 动效再加 Canvas 层（`matchMedia` 不渲染）。React 强交互站三层齐全，静态/轻交互站至少覆盖 CSS 层。不是可选项。见 `references/motion-kit.md`。
7. **公开部署 = 必须脱敏。** 站要上线给陌生人看，真实客户名/雇主名/项目名/电话/涉密信息一律不出现，改成泛称。开工前确认脱敏清单，全程遵守。
8. **在检查点验证，不是每改一处都验证。** 把相关改动攒成一个完整变更，在有意义的节点（完成一个模块后、给用户看前、提交/部署前）跑一次无头浏览器：跳过开屏、逐屏截图、查横向溢出与控制台报错，1440 与 390 两个宽度都过。靠真实渲染而非回读源码。见 `references/build-verify-deploy.md`。
9. **主动避开 AI 通病版式。** 没有明确要求时，别把独特性花在这些默认款上：暖米色 + 衬线 + 陶土色、纯黑配单一荧光色、Inter/Space Grotesk 当"安全牌"、处处 emoji 分节、全部居中、rounded-lg 满屏、紫蓝渐变 hero。用户明确要某一种就照做，否则从他的世界观里长出专属视觉。**另守两条排版硬底线**（详见 `references/motion-kit.md` 第 2a 节）：① 中文大标题禁用为拉丁字母调的负字距 / `<1` 行高 / uppercase，会叠字；② 任何非系统通用字体必须 `@font-face` 自托管 woff2，别裸依赖 `Arial Narrow / Bahnschrift / Cascadia` 等本机字体（换平台即降级），无头验证要专门核这两条。
10. **过三层防同质化自检，不只换名换色。** 每定一版方向、每做完一版都检查：① **换人测试**：只换名字和主色，是否还能套给别人；② **去皮测试**：拿掉配色、字体、职业文案后，信息顺序、Hero 构图、导航与签名交互是否仍和常见方案/其它候选同骨架；③ **职业刻板测试**：概念是否只是职业名词的第一联想，而没有个人经历、作品证据、受众任务等至少两个彼此独立的信号。任一失败就重做。多个方向至少要在**信息主次与版块顺序 / Hero 构图 / 访问者任务与签名交互 / 贯穿动线与视觉语法**中的两项真正不同；换配色、换字体、换职业道具不算新方向。详见 `references/inspiration.md`。
11. **动效分主次、给预算（通篇有动 ≠ 满屏乱动）。** 每个动效先派一个角色（定向 / 反馈 / 解释 / 签名 / 氛围），再守总量：每个大区块给**轻入场**，但**全站只留 2–3 个高光签名动效、一屏最多一个高注意力效果、全局最多一层持续氛围底**。"通篇灵动"指每个区块都活（轻入场 / hover），不是每屏都有大动效抢眼——主次分明才显高级，满屏乱动就成了噪音。预算细则见 `references/motion-kit.md` 第 7 节。

## 流程

> **进度清单**（建议复制进你的回复，逐项勾选）：
> `[ ] 0 接住 brief + 探测本地 skill` `[ ] 1 先出能跑的基础版` `[ ] 2 找隐喻/参考(先找后造)` `[ ] 3 按反馈迭代 + 证据 + 一次 grilling` `[ ] 4 动效来源 + 连接层落地` `[ ] 5 脱敏 + 上线就绪 + 部署`

### 阶段 0 · 接住 brief + 探测本地 skill
1. **接住并复述用户的 brief**：身份、想要的气质/风格方向、参考、板块结构、现有技术栈/是否新建、语言语气——把你听到的复述一遍让他确认，缺的关键项补问（一次一个）。别当白纸从头访谈。清单见 `references/interview.md`。
2. **探测可复用的 skill**（仅当你所在环境有 skill 机制时）：如 Claude Code 列出 `~/.claude/skills/` 与项目级 `.claude/skills/` 下的目录，挑出**美化/设计/动效/去 AI 腔文案**相关的，能直接复用少造轮子；环境没有 skill 机制就跳过这步。方法见 `references/inspiration.md` 的"复用本地 skill"。
3. 按 `references/build-verify-deploy.md` 先定**技术路径**：改站保留原栈；新站选择静态/轻交互或 React 强交互。只锁**硬约束**（脱敏、技术路径、语言）就够进阶段 1；视觉/动效/隐喻不必现在定死。

### 阶段 1 · 先出能跑的基础版（base-first）
用阶段 0 选定的技术路径搭基础版（见 `references/build-verify-deploy.md`），铺上**连接层契约**：共享 token、动效角色与预算、减动效和性能保护。静态/轻交互路径用 CSS 变量 + 必要的 CSS/原生 JS；React 强交互路径才使用 `references/motion-kit.md` 的 `theme.js`、动效原语和 Canvas 包装。做出一个**协调、能跑、复杂度与任务匹配**的基础站；签名交互/氛围背景不预置，到阶段 2/4 按需选择。内容先用简历/占位填。跑一次真实渲染验证再给用户看。

### 阶段 2 · 找隐喻/参考（先找后造）
基础版给用户后，定这个站的**专属世界观和视觉方向**，顺序严格是"先找后造"：
1. **先找**：问用户有没有想模仿的站/截图/现成组件代码。有 → 按 `references/inspiration.md` 的三点拆解（喜欢哪点 / 绝不抄哪点 / 可迁移的想法），以他的参考为准。
2. **引导找**：没有 → 让他去 Awwwards / reactbits / paper 逛一圈，带回 2–3 个"想要这种感觉"的例子。
3. **才共创（且先判断需不需要隐喻）**：实在没有参考 → 先看这个人**撑不撑得起隐喻**。职业身份本身不算证据；必须有个人经历、作品材料、反复出现的动作、受众任务等至少两个彼此独立的信号，而且隐喻确实帮助理解。撑得起 → 推导 2–3 个候选，并同时给一个不靠隐喻的编辑型候选，避免被职业第一联想锚定；撑不起 → **走直白的编辑型方向**：用最强证据定内容主次 + 有性格的排版构图 + 一个签名交互，不套主题外壳。两条路都过铁律 10 的三层自检——无隐喻 ≠ 平庸。

无论走哪条路，产出的方向候选都必须**过铁律 10 的三层自检**。候选至少在信息结构、Hero 构图、访问者任务/签名交互、贯穿动线/视觉语法中的两项不同；不要拿同一套“大标题 + 右侧演示器 + 微型英文标签 + 网格线”只换职业道具。定稿也要做一次同职业反事实：想象另一个同职业、但经历与受众不同的人，若仍能原样套用，就是职业模板。

### 阶段 3 · 按反馈迭代 + 证据 + 一次 grilling
这是真实质量引擎，允许推倒重来：
- **接住直白的审美否决就改**（"太商务""更丑了""太单调"这类反馈比正面 brief 更能逼出好版本）；自己看页面给具体诊断，别把设计题甩回去问。
- **要真实源文档抠证据**（见铁律 5、`references/asset-checklist.md`）：把通用套话换成可量化、可核实的事实；能接 live 数据（GitHub star、App 评分）就接。
- **当已有具体成品、但仍"太单调"时，跑一次结构化 grilling**（`references/interview.md` 的问题树）：把零散抱怨收敛成**一个连贯的大改**（真实项目里就是这样定下"继续飞行"的滚动叙事），而不是打无数零碎补丁。

### 阶段 4 · 动效来源 + 连接层落地
需要某个具体动效时，先问它是否服务访问者任务，再按已选技术路径实现：静态/轻交互站优先 CSS 或少量原生 JS；React 强交互站可从 `reactbits.dev` / `shaders.paper.design` 按需取（见 `references/inspiration.md`），看懂原理后移植。每个动效都要过连接层：统一 token，补 `prefers-reduced-motion` 静态降级和相应性能保护，并守动效预算。React/Canvas 的具体规格见 `references/motion-kit.md`。

### 阶段 5 · 脱敏 + 上线就绪 + 部署
部署前：确认**脱敏**全程落实；补齐**上线就绪**（SEO meta、社交分享卡 OG + `og:image`、favicon 全套、JSON-LD `Person`）；在检查点跑无头验证 + 过性能清单。然后 `git init` → GitHub 公开仓库 → Cloudflare Pages → 绑定域名。命令与用户需自己操作的授权见 `references/build-verify-deploy.md`。

## 参考文件

- `references/motion-kit.md` — **连接层**：`theme.js` token（含 spring 预设）、通用动效原语（Reveal/SplitText/CountUp）真实代码、`AmbientCanvas` 环境层契约包装（视觉从库现取）、三层减动效契约、动效预算、从两库移植的注意。签名交互/背景不预置。
- `references/interview.md` — 阶段 0 接住 brief 的复述清单；grilling 问题树（**在已有基础版之后**用来做一次连贯大跃迁）。
- `references/inspiration.md` — 先探测复用本地 skill；两库=动效来源与移植注意；隐喻"先找后造 + 可选"的参考拆解法。
- `references/asset-checklist.md` — 向用户索取的内容、**真实源文档**与账号清单 + 脱敏 + 图片分诊。
- `references/build-verify-deploy.md` — 技术路径路由、无头验证、性能清单、上线就绪、Git + Cloudflare 部署、改版诊断。
- `evals/scenarios.md` — 回归/触发自测场景（含极简静态、保留现有栈、同职业防模板），改完 skill 后对照自检。
