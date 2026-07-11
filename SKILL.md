---
name: create-homepage-skill
description: Build or redesign a distinctive, animation-rich personal homepage / portfolio (个人主页 / 作品集) — for a real person, from their brief through a runnable base, reference-driven iteration, headless verification, and public deployment. Use when someone wants a memorable, lively personal site with scroll-driven storytelling and micro-interactions, NOT a generic template. Ships a concrete motion "connective layer" (design tokens + portable animation primitives + a reduced-motion/performance contract) so the result is never empty or 单板; the bespoke world (metaphor, structure, visual language) grows from that person's own materials and references. Locks a proven React + Vite + framer-motion + Canvas + lazy three.js stack.
---

# Create Homepage Skill

把**一个真实的人**做成一个**有世界观、会呼吸、能上线**的个人主页。

这套流程不是凭空提炼，是从一次真实的 9 天创作（先做能跑的基础版 → 一轮轮按反馈和参考迭代 → 上线）里抽出来的。它的两个支点：

- **一套与隐喻无关的「连接层」**（设计 token + 可移植动效原语 + 三层减动效/性能契约）——保证任何人一上来就有**灵动、协调、不空洞**的底子。
- **一层专属外壳**（隐喻**或**直白编辑型方向、版式结构、视觉语言、文案）——每个人从**自己的材料和参考**里长出来，绝不预置成模板。

底子由 skill 保证质量，外壳由这个人保证独特。这就是它既"好看灵动"又"每个人都不同"的原因。

## 铁律（先读，贯穿全程）

1. **接住 brief，别当白纸访谈。** 用户通常带着身份、大致方向、甚至技术栈和材料就来了——先**接住并复述**你听到的（"你是 X，想要 Y 气质，5 个板块……对吗？"），别假装他是白纸、从零开始盘问。同时**探测你所在环境里可复用的 skill**（设计/动效/文案；环境无 skill 机制就跳过），少造轮子。见 `references/interview.md`。
2. **先出能跑的基础版（base-first）。** 别把所有决策锁死才动手。只需先确认**硬约束**（脱敏清单、技术栈、语言语气），就用连接层快速搭一个**已经灵动、协调、能跑**的基础站（内容可先用简历占位）——让用户第一眼看到的是好东西，不是线框。视觉/动效/隐喻在后面的迭代里收敛。
3. **隐喻先找后造，且隐喻可选。** 顺序是：① 先问用户有没有想模仿的**参考**（站链接/截图/现成组件代码）——真实项目里那个"航行"世界观就是用户自己贴了 `paperplanes.world` 才有的；② 没有就引导他去 Awwwards / reactbits / paper 挑一个**带回**；③ 到"共创"这步**先判断这个人需不需要隐喻**：身份/内容/受众有 ≥2 个信号支持、且隐喻能帮人看懂，才共创 2–3 个候选让他选；撑不住就走**直白的编辑型方向**（用最强证据定内容主次 + 有性格的排版 + 一个签名交互），同样要过铁律 10 的换名换色测试。**别为了"要有个概念"硬凑世界观**——硬凑正是"太装逼/像模板"的来源；但"不用隐喻"也绝不等于"可以平庸"。参考只用来校准手感与气质，不是抄版式。
4. **动效从库现取 + 套连接层。** 具体动效**按需从 `reactbits.dev` / `shaders.paper.design` 取**，看懂原理移植进锁定技术栈——**不预置一套固定动效引擎**（那会诱导换皮同质化）。但每个取来的动效都必须**过一遍连接层**：统一到 `theme.js` 的 token，并补上外部组件通常不带的减动效降级与性能保护。见 `references/motion-kit.md`、`references/inspiration.md`。
5. **证据大于修饰，主动要真实源文档。** 个人站最露馅的是通用套话、不是布局。主动向用户要**真实材料**（项目结题 PPT、会议纪要、合同、live 数据源如 GitHub star / App 评分），从里面抠**可量化成果**和**诚实的角色边界**（做了什么、不夸大），砍掉凑数的平淡条目。所有正文定稿前过一遍去 AI 腔检查——装了 `avoid-ai-writing` 这类 skill 就用它，没有就照其原则自查。
6. **减动效必须全静态降级（三层契约，无障碍底线）。** 每个动效都要有 `prefers-reduced-motion` 的静态回退：JS 层（`useReducedMotion` 返回静态元素）+ Canvas 层（`matchMedia` 不渲染）+ CSS 层（`@media` 中和所有动画）。不是可选项。见 `references/motion-kit.md`。
7. **公开部署 = 必须脱敏。** 站要上线给陌生人看，真实客户名/雇主名/项目名/电话/涉密信息一律不出现，改成泛称。开工前确认脱敏清单，全程遵守。
8. **在检查点验证，不是每改一处都验证。** 把相关改动攒成一个完整变更，在有意义的节点（完成一个模块后、给用户看前、提交/部署前）跑一次无头浏览器：跳过开屏、逐屏截图、查横向溢出与控制台报错，1440 与 390 两个宽度都过。靠真实渲染而非回读源码。见 `references/build-verify-deploy.md`。
9. **主动避开 AI 通病版式。** 没有明确要求时，别把独特性花在这些默认款上：暖米色 + 衬线 + 陶土色、纯黑配单一荧光色、Inter/Space Grotesk 当"安全牌"、处处 emoji 分节、全部居中、rounded-lg 满屏、紫蓝渐变 hero。用户明确要某一种就照做，否则从他的世界观里长出专属视觉。
10. **过"换名换色"自检，方向要在结构上真不同。** 每定一版方向、每做完一版，自问一句：**"如果只把名字和主色换掉，这站是不是随便套给另一个人也成立？"** 成立 = 同质化，重做。给用户的多个方向候选，必须在**版块顺序 / 构图 / 动效母题**上真不一样——**换配色、换字体不算一个新方向**。这条和铁律 9 互补：9 管"别撞大众款"，10 管"每个人、每一版之间要真不同"。这是防结构同质的硬闸门。
11. **动效分主次、给预算（通篇有动 ≠ 满屏乱动）。** 每个动效先派一个角色（定向 / 反馈 / 解释 / 签名 / 氛围），再守总量：每个大区块给**轻入场**，但**全站只留 2–3 个高光签名动效、一屏最多一个高注意力效果、全局最多一层持续氛围底**。"通篇灵动"指每个区块都活（轻入场 / hover），不是每屏都有大动效抢眼——主次分明才显高级，满屏乱动就成了噪音。预算细则见 `references/motion-kit.md` 第 7 节。

## 流程

> **进度清单**（建议复制进你的回复，逐项勾选）：
> `[ ] 0 接住 brief + 探测本地 skill` `[ ] 1 先出能跑的基础版` `[ ] 2 找隐喻/参考(先找后造)` `[ ] 3 按反馈迭代 + 证据 + 一次 grilling` `[ ] 4 动效来源 + 连接层落地` `[ ] 5 脱敏 + 上线就绪 + 部署`

### 阶段 0 · 接住 brief + 探测本地 skill
1. **接住并复述用户的 brief**：身份、想要的气质/风格方向、参考、板块结构、技术栈、语言语气——把你听到的复述一遍让他确认，缺的关键项补问（一次一个）。别当白纸从头访谈。清单见 `references/interview.md`。
2. **探测可复用的 skill**（仅当你所在环境有 skill 机制时）：如 Claude Code 列出 `~/.claude/skills/` 与项目级 `.claude/skills/` 下的目录，挑出**美化/设计/动效/去 AI 腔文案**相关的，能直接复用少造轮子；环境没有 skill 机制就跳过这步。方法见 `references/inspiration.md` 的"复用本地 skill"。
3. 只锁**硬约束**（脱敏、技术栈、语言）就够进阶段 1；视觉/动效/隐喻不必现在定死。

### 阶段 1 · 先出能跑的基础版（base-first）
用锁定技术栈脚手架（见 `references/build-verify-deploy.md`），铺上**连接层**（`references/motion-kit.md`：`theme.js` token + `Reveal`/`SplitText`/`CountUp` 通用原语 + `AmbientCanvas` 环境层契约包装 + 三层减动效契约），做出一个**已经灵动、协调、能跑**的基础站。签名交互/氛围背景不预置，到阶段 2/4 从库现取。内容先用简历/占位填。目标：用户第一眼看到的是一个有手感的真站，不是空线框——这是防"效果太差/单板"的关键一步。跑一次无头验证再给用户看。

### 阶段 2 · 找隐喻/参考（先找后造）
基础版给用户后，定这个站的**专属世界观和视觉方向**，顺序严格是"先找后造"：
1. **先找**：问用户有没有想模仿的站/截图/现成组件代码。有 → 按 `references/inspiration.md` 的三点拆解（喜欢哪点 / 绝不抄哪点 / 可迁移的想法），以他的参考为准。
2. **引导找**：没有 → 让他去 Awwwards / reactbits / paper 逛一圈，带回 2–3 个"想要这种感觉"的例子。
3. **才共创（且先判断需不需要隐喻）**：实在没有参考 → 先看这个人**撑不撑得起隐喻**（身份/内容/受众 ≥2 个信号支持、且能帮人看懂）。撑得起 → 从身份、内容证据、受众里推导 2–3 个隐喻候选让他选，说清每个怎么渗透进文案/图标/动效，别预设某职业只能对应某隐喻。撑不起（如以文字/证据取胜的顾问、研究者）→ **走直白的编辑型方向**：用最强证据定内容主次 + 有性格的排版构图 + 一个签名交互，不套主题外壳。两条路都要过铁律 10 的换名换色测试——无隐喻 ≠ 平庸。

无论走哪条路，产出的方向候选都必须**过铁律 10 的"换名换色"自检**：几个候选之间要在**版块顺序 / 构图 / 动效母题**上真不同，不是同一版式换配色。定稿一版也先自问"换掉名字和主色还成立吗"，成立就是同质化。

### 阶段 3 · 按反馈迭代 + 证据 + 一次 grilling
这是真实质量引擎，允许推倒重来：
- **接住直白的审美否决就改**（"太商务""更丑了""太单调"这类反馈比正面 brief 更能逼出好版本）；自己看页面给具体诊断，别把设计题甩回去问。
- **要真实源文档抠证据**（见铁律 5、`references/asset-checklist.md`）：把通用套话换成可量化、可核实的事实；能接 live 数据（GitHub star、App 评分）就接。
- **当已有具体成品、但仍"太单调"时，跑一次结构化 grilling**（`references/interview.md` 的问题树）：把零散抱怨收敛成**一个连贯的大改**（真实项目里就是这样定下"继续飞行"的滚动叙事），而不是打无数零碎补丁。

### 阶段 4 · 动效来源 + 连接层落地
需要某个具体动效时，从 `reactbits.dev` / `shaders.paper.design` 按需取（各库特点与移植注意见 `references/inspiration.md`），看懂原理**移植进锁定栈**（framer-motion + Canvas + 懒加载 three；paper-shaders 按需 npm）。每个动效落地都要**过连接层**：统一到 `theme.js` 的颜色/spring/间距 token，补上 `prefers-reduced-motion` 静态降级 + 离屏暂停 / DPR 上限 / 触屏降密的性能契约，并守**动效预算**（铁律 11 / `motion-kit.md` 第 7 节：每区块轻入场、全站 2–3 个高光、一屏一个高注意力、一层氛围底）。规格见 `references/motion-kit.md`。

### 阶段 5 · 脱敏 + 上线就绪 + 部署
部署前：确认**脱敏**全程落实；补齐**上线就绪**（SEO meta、社交分享卡 OG + `og:image`、favicon 全套、JSON-LD `Person`）；在检查点跑无头验证 + 过性能清单。然后 `git init` → GitHub 公开仓库 → Cloudflare Pages → 绑定域名。命令与用户需自己操作的授权见 `references/build-verify-deploy.md`。

## 参考文件

- `references/motion-kit.md` — **连接层**：`theme.js` token（含 spring 预设）、通用动效原语（Reveal/SplitText/CountUp）真实代码、`AmbientCanvas` 环境层契约包装（视觉从库现取）、三层减动效契约、动效预算、从两库移植的注意。签名交互/背景不预置。
- `references/interview.md` — 阶段 0 接住 brief 的复述清单；grilling 问题树（**在已有基础版之后**用来做一次连贯大跃迁）。
- `references/inspiration.md` — 先探测复用本地 skill；两库=动效来源与移植注意；隐喻"先找后造 + 可选"的参考拆解法。
- `references/asset-checklist.md` — 向用户索取的内容、**真实源文档**与账号清单 + 脱敏 + 图片分诊。
- `references/build-verify-deploy.md` — 技术栈、无头验证脚本、性能清单、上线就绪、Git + Cloudflare 部署、改版诊断。
- `evals/scenarios.md` — 回归/触发自测场景（带 brief+参考 / 小白啥都没有 / 要极简无 3D），改完 skill 后对照自检。
