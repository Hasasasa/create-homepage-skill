# 技术栈 · 验证 · 上线就绪 · 部署 · 改版诊断

## 技术栈（锁定）

从一次真实上线打磨出来的组合，兼顾手感、性能、可维护：

- **React 18 + Vite 5**（`@vitejs/plugin-react`）——快、生态成熟、脚手架简单。
- **framer-motion**（`LazyMotion` + `domAnimation` + `m.*`）——动效基座，按需加载。见 `motion-kit.md`。
- **Canvas 2D**——环境背景层、点阵/粒子类效果（视觉从库现取，套 `motion-kit.md` 的 `AmbientCanvas` 性能契约包装）。
- **three.js —— 仅当需要 3D 时**才引，且**动态 import + 只在用到的那个组件里**（真实项目里只有开屏地球用它，主站不含），失败要有降级回退。绝不进主 bundle。
- **@paper-design/shaders-react —— 按需**（要生成式 WebGL 背景/封面时才 `npm i`），懒加载、控弱机开销。
- **纯 CSS + CSS 变量**（`:root` token），单一 `index.css`；避免为个别组件引入整套 Tailwind。
- **数据驱动**：所有文案/数据集中 `src/data/site.js`，所有视觉 token 集中 `src/data/theme.js`。

设计 token 规格（`theme.js`：颜色/字阶/间距/spring 预设）见 `motion-kit.md` 第 2 节——**spec 先行、代码在后**，一处定义全局引用，避免 build 漂移成"处处略不一致"的通病。

## 无头验证（检查点跑，不是每改一处都跑）

把相关改动攒成一个完整变更，在有意义的节点（完成一个模块后、给用户看前、提交/部署前）跑一次。用 puppeteer-core 打本机已装 Chrome：

- **脚本放项目根**（避免绝对路径——如 Windows `d:\` 盘符——的 ESM URL scheme 报错），用 bare `import 'puppeteer-core'`。
- `executablePath` 指向本机 Chrome；**逐屏 `scrollIntoView` + 定时等待**再截图（hash 导航常不滚动/不等待）。
- 视口用正常尺寸（如 1440×900），**别用超高视口截全页**（`100vh` 会随视口膨胀）。
- 需要 WebGL 软渲染时加：`--use-gl=angle --use-angle=swiftshader --enable-unsafe-swiftshader --ignore-gpu-blocklist --enable-webgl`。

**每次检查点断言：**
1. `document.scrollWidth <= document.clientWidth`（**无横向溢出**）——1440 与 390 两个宽度都过。
2. **0 控制台报错**（监听 `console`/`pageerror`）。
3. 逐屏截图肉眼过一遍：有没有错位、盖住、糊图、留白撑开。
4. **上线就绪存在性检查**：`<title>`、`meta[property="og:image"]`、favicon 链接都在。

靠真实渲染判断，不靠回读源码。验证完把脚本和 `puppeteer-core` 依赖清理掉。

## 性能清单

- framer-motion 用 `LazyMotion + domAnimation + m.*`（非 `motion.*`）。
- Canvas/WebGL：DPR 上限（Canvas `≤2`、WebGL `≤1.75`）；密度按视口面积算、触屏减半；离屏 / 切后台 `visibilitychange` 暂停 rAF。
- three.js / paper-shaders 动态 import + 懒加载，不进主 bundle。
- 图片：别把用不到的塞进 `public`（每次构建都打包）；大图压缩。
- `npm run build` 通过，留意 JS/CSS gzip 体积。

## 上线就绪（SEO / 社交分享 / favicon）

公开求职/作品站的高可见度刚需，手搭最易漏，部署前补齐：
- **SEO meta**：`<title>`、`meta description`、语言、canonical。
- **社交分享卡**：Open Graph（`og:title`/`og:description`/`og:image`）+ Twitter card；`og:image` 做 1200×630。
- **favicon 全套**：SVG + 各尺寸 PNG + apple-touch-icon。
- **JSON-LD `Person`**：结构化数据，利于搜索呈现。

## 部署（Git + Cloudflare Pages + 域名）

1. `git init` → 提交 → 推到 **GitHub 公开仓库**（`gh` 或手动；`gh auth login` 需用户本人完成）。
2. **Cloudflare Pages** 连仓库自动构建部署（`npx wrangler login` 需用户点浏览器 Allow）。**若目标访客在中国大陆**，Cloudflare + 自有域名可达性较好；否则 Vercel / Netlify / Pages 等任意静态托管皆可。
3. **绑定自有域名**：DNS 记录 / 激活需用户在 Cloudflare 面板操作（wrangler OAuth 通常无 DNS 写权限）。
4. 部署后再跑一次无头验证兜底。

## 改版诊断（用户说"不好看/太单调"时）

**别把设计题甩回去问用户**——自己看页面，给具体诊断，按分级先小后大改。

先做一张 **反应 → 原因 → 修法** 表，自问：

| 用户反应 | 大概率原因 | 先试的修法 |
|---|---|---|
| "太单调 / 交互太少" | 各板块孤立堆叠、没有贯穿动线 | 加一条随滚动推进的主动线（`interview.md` Q3），而非到处加零碎微交互 |
| "太商务 / 太装逼 / 像模板" | 撞了 AI 通病版式（铁律 9），没有专属世界观 | 走"先找后造"定隐喻，重做视觉语言 |
| "还是有点像 / 换个人也能用" | 结构同质：外壳换了皮，版式/动线没差异 | 过"换名换色"自检（铁律 10）；差异要落到**版块顺序 / 构图 / 贯穿动线**，不是只换配色字体 |
| "更丑了 / 麻麻赖赖" | 某个效果参数没调好、或堆叠过度 | 收敛：减数量、调 spring/配色 token，别再加 |
| "不是我的风格" | 气质方向没对齐 | 回参考站校准张扬度，重定 `theme.js` |
| "这里不对 / 不居中" | 具体布局 bug | 无头截图定位，单点修 |

**分级改版，先小后大**：润色（配色/间距/字重）→ 单个区块 → 视觉风格 → 动效体系 → 结构 → 整站翻转。能小改解决就别推倒；但真实项目里也发生过 3 次推倒重来——**当小改反复无效时，果断走结构级或翻转级**，并跑一次 grilling（`interview.md` 阶段 3）把大改收敛成一个连贯方案。
