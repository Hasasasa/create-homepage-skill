# 连接层（motion-kit）—— skill 自带的具体骨架

这一层**与任何隐喻无关**。它为交互型个人站提供**灵动、协调、不空洞，并具备动效可访问性底线**的底子——也是上一版 skill"效果太差/单板"缺失的东西。

> 本文件的代码是 **React 强交互路径**实现，不是所有个人站的技术栈要求。改造现有站时保留原栈；静态/轻交互站只迁移这里的契约（token、动效角色/预算、减动效、性能保护），用 CSS 或少量原生 JS 实现。
>
> 本文件只覆盖**动效相关**的可访问性，不能据此宣称整站符合无障碍标准。交付前仍要单独验证语义结构、键盘操作、焦点可见与管理、颜色对比度、替代文本和触控目标。

原则：**动效本身按需从库里取（见 `inspiration.md`），但都要落进这一层。** 这一层负责三件事：① 用共享 token 让拼来的东西不散；② 用可移植原语给一个开箱即用的手感基线；③ 对实际使用到的 CSS、JS、Canvas/WebGL 层分别保证减动效降级和性能。

下面的代码/参数整理自一个 React + Motion + Canvas 站，选中 React 强交互路径时可作为起点，并按每个人 recolor / retune。它们不绑定 React、构建工具或 Motion 的固定版本；先核对项目已安装依赖的 API，再迁移示例。

---

## 1. Motion for React 规范

> **动效引擎按需选，别默认只有 Motion 一种。** 组件级入场 / 手势 / 布局动画用 Motion（本节及第 3 节示例）；**时间线编排的开场序列、滚动 pin / scrub / parallax（高端作品集、创意机构官网那一类）用 GSAP + ScrollTrigger 更顺手**，两者可共存。GSAP 路径同样过连接层：token 统一、reduced-motion 下不注册 ScrollTrigger（第 5 节）、组件卸载 `ctx.revert()` 清理、字体/图片加载完成后 `ScrollTrigger.refresh()`。引擎选择与按需引入见 `build-verify-deploy.md`「React 强交互路径」。

- 沿用项目已经安装的 `motion` 或 `framer-motion` 及其导入路径，不为统一示例迁移依赖；先核对当前版本是否支持下列 API。
- 支持时用 `LazyMotion + domAnimation`，在该 Provider 内统一用 `m.*`。不要写死体积收益；用项目的生产构建和 bundle analyzer 实测。
- 在应用根部包一层 `MotionProvider`。

```jsx
// 若项目已使用 framer-motion，沿用原包名并按已安装版本核对 API。
import { LazyMotion, domAnimation } from 'motion/react'

export function MotionProvider({ children }) {
  return <LazyMotion features={domAnimation}>{children}</LazyMotion>
}
```

---

## 2. 共享设计 token（一处定义，全局引用；以下以 `theme.js` 为例）

spec 先行、代码在后。**spring 预设命名化**是关键（外部组件常把 spring 写死在各处，拼进来就手感各异）。下面四个预设是从真实站抽出的、覆盖绝大多数场景的手感档位：

```js
// src/data/theme.js —— 每个人改这里的值，不改组件
export const color = {
  bg: '#08090c', surface: '#0e1014', text: '#f3f4f7',
  muted: '#9aa0ad', muted2: '#6b7180',
  accent: '#e8eaf0',      // 主强调（换成这个人的方向强调色）
  pop: '#ff8a3c',         // 单一点睛色，克制使用
  line: 'rgba(255,255,255,0.08)',
}
export const type = {
  // 任何非系统通用字体都必须有获授权、跨平台的 Web 加载路径（见 2a）；
  // fallback 里带一款 CJK 字体，避免中文回退本机黑体后度量错乱、行高崩坏。
  display: `'YourDisplay', 'Noto Sans SC', system-ui, sans-serif`,  // ← 别默认 Space Grotesk，见铁律 9
  body:    `'YourBody', 'Noto Sans SC', system-ui, sans-serif`,
  mono:    `'YourMono', ui-monospace, monospace`,
  // 压缩体/紧排的拉丁标题另设一档，别拿它排中文（会叠字，见 2a）：
  displayLatin: `'YourCondensed', system-ui, sans-serif`,
}
export const layout = { maxw: '1700px', pad: '80px', ease: 'cubic-bezier(0.22, 1, 0.36, 1)' }

// spring 预设 —— 全站动效都引用这四档，禁止在组件里散写 stiffness/damping
export const spring = {
  soft:   { type: 'spring', stiffness: 130, damping: 17, mass: 0.9 }, // 滚动上浮入场
  snappy: { type: 'spring', stiffness: 340, damping: 22, mass: 0.7 }, // 逐字/小元素弹入
  tilt:   { type: 'spring', stiffness: 220, damping: 18, mass: 0.6 }, // 指针跟随类（倾斜/磁吸）起点手感——效果本身从库取
  scroll: { type: 'spring', stiffness: 55,  damping: 16, mass: 0.5 }, // 滚动进度平滑
}
```

> 这四组数值是**起点手感，不是固定签名**。按每个人的 Q7（张扬/克制、快/慢）retune——同一套原语换掉 spring 数值，手感就完全不同，这也是避免"每版手感都一样"的一环。

> 提醒：`Space Grotesk` / `Inter` 是 AI 通病"安全牌"字体（铁律 9）。除非用户明确要，否则换成更贴这个人气质的字体。

### 2a. 中文排版护栏 + 字体交付/许可（硬规则）

产出几乎都是中文站，但压缩体大标题那套手感参数是给拉丁字母调的，直接套到中文全角字会**叠字/碰撞**（真实产出里 hero 标题「信号/噪声」糊成一团、摄影师多行标题上下压字，都是这么来的）。两条硬底线：

**① 中文排版护栏**
- 多行中文大标题：`line-height ≥ 1.05`、`letter-spacing ≈ 0`（最多 `-0.01em`）。**禁用**为 Latin 调的 `line-height: .7~.95` / `letter-spacing: -.06~-.09em`——那是叠字根源。
- `text-transform: uppercase` 对中文**无效**，别指望它出效果；只有确实是拉丁字母的 eyebrow/标题才用。
- 想要"压缩体/紧排"的工程感时，拉丁标题走 `type.displayLatin` 那一档，中文标题走 `type.display`，**分开设**。
- **桌面和移动断点都要覆盖**——叠字常常只在某一个断点漏改（真实产出里就有移动端标题忘了抬行高而叠字）。

**② 字体交付与许可（强制）**
`type` 里任何非系统通用字体都必须有**获授权、跨平台的 Web 加载路径**，并在部署前核对许可；不要只靠作者机器已安装：
- `Arial Narrow / Bahnschrift / Aptos Narrow / Cascadia Mono` 等常见于特定系统，在 mac/iOS/Linux/Android 可能直接回退——压缩体美学丢失，中文还可能触发上面的叠字。
- 许可允许自托管和子集化时，可用项目内 woff2；许可要求指定提供商/CDN 时，按授权方式加载，并评估 CSP、隐私、离线与网络失败回退。**不要在许可不明时自行转换、子集化或重新分发商业字体。**
- 选开源字体也要读取对应版本的许可证与保留要求，不把“开源”理解成没有交付义务。
  ```css
  @font-face{ font-family:'YourDisplay'; src:url('/fonts/your-display.woff2') format('woff2');
    font-weight:400 800; font-display:swap; }
  ```
- **别被作者本机骗了**：截图机器上恰好装了这些字，问题不显形——见 `build-verify-deploy.md` 浏览器验证的「中文不叠字 / 字体不靠本机」两条断言。上面的 `@font-face` 仅是许可允许自托管时的示例。

---

## 3. 可移植动效原语（通用机制，非签名）

下面三个是常见的通用机制（滚动淡入、逐字入场、数字滚动），不是每个站都要启用。复用时仍要按内容任务选择，并调整节奏、幅度和 token；只换 spring 数值不足以证明方向已经不同。也可评估 React Bits 里的同类组件，但移植前必须核对 imports、peer dependencies、许可证和版本兼容性，不能假设拷贝即用。共同点是**都内建了 reduced-motion 静态回退**（见第 5 节契约）。

> **签名表达不在这里预置**——静态方向靠信息结构、排版、图像或构图形成记忆点；动态方向才按需选签名交互与氛围背景（见本节后半）。这一档只提供通用机制候选，仍要按任务筛选，不能因为“通用”就全部默认启用。

> 下列 React 示例使用 `motion/react` 展示 API；项目已使用 `framer-motion` 或其它兼容方案时，沿用现有包与导入路径，并按已安装版本核对 API。token 文件路径也只是示例。

### Reveal —— 滚动上浮入场（用 `spring.soft`）

```jsx
import { m, useReducedMotion } from 'motion/react'
import { spring } from '../data/theme'

export function Reveal({ as = 'div', delay = 0, y = 30, className, style, children, ...rest }) {
  const reduced = useReducedMotion()
  const Tag = m[as] || m.div
  if (reduced) {
    const Plain = as
    return <Plain className={className} style={style} {...rest}>{children}</Plain>
  }
  return (
    <Tag className={className} style={style}
      initial={{ opacity: 0, y }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true, margin: '0px 0px -70px 0px' }}
      transition={{ ...spring.soft, delay }}
      {...rest}>{children}</Tag>
  )
}
```

### SplitText —— 逐字弹入标题（用 `spring.snappy`）

可访问性要点：保留一份完整文本供辅助技术读取，把逐字动画副本整体设为 `aria-hidden`；外层保持原生文本语义，不添加额外 `role`。

```jsx
import { m, useReducedMotion } from 'motion/react'
import { spring } from '../data/theme'

const visuallyHidden = {
  position: 'absolute',
  width: 1,
  height: 1,
  padding: 0,
  margin: -1,
  overflow: 'hidden',
  clip: 'rect(0, 0, 0, 0)',
  whiteSpace: 'nowrap',
  border: 0,
}

export function SplitText({ text, className, delay = 0, step = 0.032 }) {
  const reduced = useReducedMotion()
  if (reduced) return <span className={className}>{text}</span>
  return (
    <span className={className}>
      <span style={visuallyHidden}>{text}</span>
      <span aria-hidden="true">
        {Array.from(text).map((ch, i) => (
          <m.span key={i}
            style={{ display: 'inline-block', whiteSpace: 'pre' }}
            initial={{ opacity: 0, y: '0.65em', scale: 0.86 }}
            whileInView={{ opacity: 1, y: '0em', scale: 1 }}
            viewport={{ once: true, margin: '0px 0px -60px 0px' }}
            transition={{ ...spring.snappy, delay: delay + i * step }}>
            {ch === ' ' ? ' ' : ch}
          </m.span>
        ))}
      </span>
    </span>
  )
}
```

### CountUp —— 数字滚动仪表盘

进入视口后从 0 滚到目标；`to` 变化时从当前显示值续滚（接 live 数据用）。

```jsx
import { useEffect, useRef } from 'react'
import { useReducedMotion, useInView, animate } from 'motion/react'

export function CountUp({ to, duration = 1.4, delay = 0 }) {
  const ref = useRef(null), fromRef = useRef(0)
  const reduced = useReducedMotion()
  const inView = useInView(ref, { once: true, margin: '0px 0px -40px 0px' })
  useEffect(() => {
    const el = ref.current; if (!el) return
    if (reduced || !inView) { if (reduced) el.textContent = String(to); return }
    const controls = animate(fromRef.current, to, {
      duration, delay: fromRef.current === 0 ? delay : 0, ease: [0.16, 1, 0.3, 1],
      onUpdate: (v) => { fromRef.current = v; el.textContent = String(Math.round(v)) },
    })
    return () => controls.stop()
  }, [inView, reduced, to, duration, delay])
  return (
    <span>
      <span style={{
        position: 'absolute', width: 1, height: 1, padding: 0, margin: -1,
        overflow: 'hidden', clip: 'rect(0, 0, 0, 0)', whiteSpace: 'nowrap', border: 0,
      }}>{to}</span>
      <span ref={ref} aria-hidden="true">{reduced ? to : 0}</span>
    </span>
  )
}
```

实际项目优先把内联隐藏样式换成已有 `sr-only` 工具类。辅助技术直接读取最终数值，不跟随每一帧的动画值。

### 动态方向的签名交互 & 氛围背景 —— 不预置，从库现取（这里只给契约）

静态方向不要求签名交互：用信息结构、排版、图像处理或静态构图形成**签名表达**，不要为了“签名”硬加 hover、Canvas 或 WebGL。动态方向确实需要时，签名交互（卡片 3D 倾斜 / 边缘辉光 / 磁吸等）和氛围背景也**一律不预置**——从 React Bits（微交互 / Backgrounds）或 Paper（生成式 WebGL）按需评估，让它从这个人的方向长出来。取来后必须套两样：① 第 5 节中实际使用层的减动效契约；② 若是 Canvas 效果，套下面这层**无视觉指纹的性能包装**。

```jsx
import { useEffect, useRef } from 'react'

// Canvas 环境层「性能 + 减动效」契约包装 —— 视觉不预置，从库里取来画进 draw()。
// 只负责：监听减动效 / DPR 上限 / 密度按面积+触屏减半 / 切后台暂停 / 清理。
export default function AmbientCanvas() {
  const canvasRef = useRef(null)
  useEffect(() => {
    const canvas = canvasRef.current
    const ctx = canvas?.getContext('2d')
    if (!canvas || !ctx) return

    const reduceQuery = window.matchMedia('(prefers-reduced-motion: reduce)')
    const coarseQuery = window.matchMedia('(pointer: coarse)')
    let W = 0, H = 0, raf = 0, running = false
    let items = []                                                                // ← 你从库取来的效果的粒子/状态
    const build = () => {
      const dpr = Math.min(window.devicePixelRatio || 1, 2)                        // ① DPR 上限 2
      W = innerWidth; H = innerHeight
      canvas.width = W * dpr; canvas.height = H * dpr
      canvas.style.width = W + 'px'; canvas.style.height = H + 'px'
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
      const count = Math.round((W * H) / 22000 / (coarseQuery.matches ? 2 : 1))    // ② 密度按面积、触屏减半
      items = Array.from({ length: count }, () => ({ /* 你的粒子初始状态 */ }))
    }
    const draw = () => {
      if (!running) return
      ctx.clearRect(0, 0, W, H)
      // ──③ 从外部库取来的视觉画在这里（雪 / 尘 / 网格 / 波纹 / mesh 皆同理）──
      raf = requestAnimationFrame(draw)
    }
    const stop = () => {
      running = false
      cancelAnimationFrame(raf)
      raf = 0
      ctx.clearRect(0, 0, W, H)
    }
    const start = () => {
      if (running || reduceQuery.matches || document.visibilityState !== 'visible') return
      build()
      running = true
      raf = requestAnimationFrame(draw)
    }
    const onMotionChange = () => reduceQuery.matches ? stop() : start()             // ④ 设置变化时立即停/启
    const onVisibilityChange = () => document.visibilityState === 'visible' ? start() : stop() // ⑤ 切后台暂停
    const onResize = () => { if (running) build() }

    start()
    addEventListener('resize', onResize)
    document.addEventListener('visibilitychange', onVisibilityChange)
    reduceQuery.addEventListener('change', onMotionChange)
    return () => {
      stop()
      removeEventListener('resize', onResize)
      document.removeEventListener('visibilitychange', onVisibilityChange)
      reduceQuery.removeEventListener('change', onMotionChange)                    // ⑥ 清理
    }
  }, [])
  return <canvas ref={canvasRef} aria-hidden="true"
    style={{ position: 'fixed', inset: 0, zIndex: 0, pointerEvents: 'none' }} />
}
```

> 若用外部 WebGL 背景而非手写 Canvas，同样要：懒加载、弱机降开销、标签页切后台时通过 `visibilitychange` 暂停、reduced-motion 时换静帧 / 纯色。不是固定全屏背景、可能滚出视口的组件还要用 `IntersectionObserver` 管理离屏暂停；不要把两种可见性事件混为一谈。

---

## 4. 其它可复用的手感细节（CSS）

- **渐进增强 + 减动效全局兜底**：基础样式必须是可见的最终状态，只在 `no-preference` 下添加初始隐藏/位移和动画。不能把 `opacity: 0`、裁切或离屏位移写成基础状态，再指望 `animation: none` 自动恢复内容。
  ```css
  .reveal {
    opacity: 1;
    transform: none;
  }

  @media (prefers-reduced-motion: no-preference) {
    .reveal { animation: reveal-in 560ms var(--ease) both; }
    @keyframes reveal-in {
      from { opacity: 0; transform: translateY(24px); }
      to { opacity: 1; transform: none; }
    }
  }

  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { animation: none !important; transition: none !important; scroll-behavior: auto !important; }
    .reveal { opacity: 1 !important; transform: none !important; }
  }
  ```
  对每个自定义 reveal/clip-path 选择器都保留同样的最终状态，并在真实 `reduce` 媒体模拟下验证正文、导航和 CTA 全部可见。
- **过冲弹性缓动**（列表标记滑入等）：`transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);`
- **胶片颗粒氛围**：内联 SVG `feTurbulence fractalNoise` 作 `::after` 固定层，`opacity: .035`（避免 `mix-blend-mode`，合成开销大）。
- **Canvas / WebGL DPR 上限**：Canvas `Math.min(devicePixelRatio, 2)`，WebGL `1.75`。

---

## 5. 分层减动效契约（覆盖用到的层，强制）

项目用了 CSS 动画/过渡时保留 CSS 全局兜底；JS、Canvas/WebGL 只在项目实际使用该层时补。完全无动效的站无需添加空的降级层。**覆盖用到的每一层，不为凑层数引入新技术。**

| 层 | 判定 | 行为 |
|---|---|---|
| **CSS（共同底线）** | `no-preference` 渐进增强 + `reduce` 兜底 | 基础状态保持内容可见；关闭动画后仍显式恢复最终状态（第 4 节） |
| **JS 动效引擎（Motion / GSAP 等）** | Motion 的 reduced-motion API；GSAP 用 `matchMedia` 判定 | Motion 组件返回**静态元素**（Reveal/SplitText 已内建）；GSAP 时间线不 play、ScrollTrigger 不注册（或 `.kill()`），直接渲染终态；从库取的交互也要补 |
| **Canvas / WebGL** | `matchMedia(...).matches` + `change` 监听 | **停止渲染** / 快照静帧；偏好在页面打开期间变化也立即响应 |

从外部库（reactbits / paper）取来的组件通常没有完整的 reduced-motion 处理——移植时检查它用了哪些层，并逐层补齐。这是它们“直接能用”但不能原样上线的关键差别。

---

## 6. 从两库移植的注意（详见 `inspiration.md`）

- **React Bits**：不要假设“拷贝即用”。每次移植前检查组件源码的 imports、安装说明、peer dependencies、React/Motion 版本兼容性、样式或 Tailwind 前提，以及当时的许可证与署名要求；按许可证保留通知并记录来源。适配现有栈后再跑生产构建、交互与减动效验证。
- **Paper shaders**：按其当前文档核对包名、API、peer dependencies、许可证和版本兼容性后再安装。WebGL 效果要**懒加载 + 弱机降开销 + 切后台暂停 + 减动效静帧/纯色回退**。

## 7. 动效预算（仅在选择动态方向时使用）

静态方向直接跳过本节，零动画是有效选择；它的记忆点由信息结构、排版、图像与静态构图承担，不需要“签名交互”。选择动态方向后，用下面这套主次纪律防止“什么都在动”变成噪音。

**先给每个动效派一个角色**（一个动效只担一个角色，担不上就删）：

| 角色 | 干什么 | 例 |
|---|---|---|
| 定向 | 让人知道"在哪、往哪走" | 区块入场、滚动进度、导航状态 |
| 反馈 | 回应操作 | hover、点击、复制成功、切换类别 |
| 解释 | 帮人看懂 | 展开一个流程 / 对比 / 时间线 |
| 签名 | 这个人专属的记忆点 | 贯穿动线那类（真实项目里的"继续飞行"） |
| 氛围 | 低存在感的背景底 | Canvas 环境层（套 `AmbientCanvas` 包装）、paper 生成式底 |

**再守总量预算（硬约束）：**
- 普通区块**最多**给轻入场或必要状态变化；没有作用就保持静态。
- **全站只留 2–3 个高光签名动效**，别处处高潮。
- **一屏之内最多一个高注意力效果**——别在同一屏里放俩东西抢眼。
- **全局最多一层持续氛围底**——别叠三层粒子/shader。
- 外部现取的签名效果也**≤2–3 个**；其余日常入场/反馈优先复用本节原语与项目共享 token，别每处造轮子。

动态方向的“灵动”来自有意义的定向、反馈和少数签名效果，**不是**要求每个区块都动。主次分明比满屏乱动更显高级。

## 7a. 预算花在哪：分配 > 天花板

上面第 7 节是“别超量”的天花板；这一节是“该往哪花”。真实翻车常常是**反着来的**——氛围底堆太多、内容入场太糊：满屏都是常驻的氛围装饰，而每个区块只有一模一样的“整块淡入（translateY＋opacity）”。结果“很忙，但没人编排过”，动效不少却显得平。把预算从背景挪到内容：

- **氛围压到一层、且低存在感**（铁律 11 已定上限；好几层氛围装饰同时常驻＝已超标）。装饰性循环别用盲计时器乱闪——要么让它**被触发／有意义**（加载时、hover 名字时），要么删掉。
- **开场编排（差距最大的一处）**：首屏别“啪一下全在那”。给一个约 1–1.5 秒的进场序列（eyebrow→标题→定位→联系方式→指标依次进），或标题遮罩揭开／位移归位。要时间线编排时 GSAP 顺手，见 `build-verify-deploy.md`「React 强交互路径」。
- **错峰（stagger）入场（性价比最高的一处 craft）**：reveal 别整块挂在容器上让子元素同时淡入；给列表／卡片／时间线子项递增 40–80ms 延迟（用 Reveal 原语的 `delay`，或容器级 stagger）。
- **滚动驱动**：滚动时别只有二值淡入。要不要视差／scrub／pin 按方向定，但至少让滚动“带动”点东西，而不是静止页面偶发淡入；用 ScrollTrigger 或 `IntersectionObserver`＋进度，别裸挂 `scroll` 事件。

一句话：**灵动来自内容怎么进场、怎么随滚动展开，不是背景一直在动。**
