# 连接层（motion-kit）—— skill 自带的具体骨架

这一层**与任何隐喻无关**。它是让"任何人（含啥都没带的小白）"都能得到一个**灵动、协调、不空洞、无障碍达标**的底子的地方——也是上一版 skill"效果太差/单板"缺失的东西。

原则：**动效本身按需从库里取（见 `inspiration.md`），但都要落进这一层。** 这一层负责三件事：① 用共享 token 让拼来的东西不散；② 用可移植原语给一个开箱即用的手感基线；③ 用三层契约保证减动效降级和性能。

下面的代码/参数直接抄自一个真实上线站（React 18 + Vite 5 + framer-motion + Canvas），可粘贴即用，按每个人 recolor / retune。

---

## 1. framer-motion 规范

- 用 `LazyMotion + domAnimation`，全站用 `m.*` 不用 `motion.*`（bundle 从 ~34kb 降到 ~15kb）。
- 在应用根部包一层 `MotionProvider`。

```jsx
import { LazyMotion, domAnimation, m } from 'framer-motion'
export function MotionProvider({ children }) {
  return <LazyMotion features={domAnimation}>{children}</LazyMotion>
}
```

---

## 2. `theme.js` —— 设计 token（一处定义，全局引用）

spec 先行、代码在后。**spring 预设命名化**是关键（外部组件常把 spring 写死在各处，拼进来就手感各异）。下面四个预设是从真实站抽出的、覆盖绝大多数场景的手感档位：

```js
// src/data/theme.js —— 每个人改这里的值，不改组件
export const color = {
  bg: '#08090c', surface: '#0e1014', text: '#f3f4f7',
  muted: '#9aa0ad', muted2: '#6b7180',
  accent: '#e8eaf0',      // 主强调（换成这个人的世界观强调色）
  pop: '#ff8a3c',         // 单一点睛色，克制使用
  line: 'rgba(255,255,255,0.08)',
}
export const type = {
  display: `'Space Grotesk', system-ui, sans-serif`,  // ← 别默认它，见铁律 9
  body: `'Inter', system-ui, sans-serif`,
  mono: `ui-monospace, 'Cascadia Code', monospace`,
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

---

## 3. 可移植动效原语（通用机制，非签名）

下面三个是**近乎人人都用的通用机制**（滚动淡入、逐字入场、数字滚动）——复用机制本身不会撞脸，只要按人 retune spring 数值即可。它们是**通用底层，不是隐喻模块**，也可直接用 `reactbits` 里的同类替换；共同点是**都内建了 reduced-motion 静态回退**（见第 5 节契约）。

> **签名交互和氛围背景不在这里预置**——那是这个人的视觉指纹，从库现取（见本节后半"签名交互 & 氛围背景"）。这一档只放不会造成同质化的通用机制。

### Reveal —— 滚动上浮入场（用 `spring.soft`）

```jsx
import { m, useReducedMotion } from 'framer-motion'
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

无障碍要点：外层挂 `aria-label` + `role="text"`，每个字 `aria-hidden`。

```jsx
export function SplitText({ text, className, delay = 0, step = 0.032 }) {
  const reduced = useReducedMotion()
  if (reduced) return <span className={className}>{text}</span>
  return (
    <span className={className} aria-label={text} role="text">
      {Array.from(text).map((ch, i) => (
        <m.span key={i} aria-hidden="true"
          style={{ display: 'inline-block', whiteSpace: 'pre' }}
          initial={{ opacity: 0, y: '0.65em', scale: 0.86 }}
          whileInView={{ opacity: 1, y: '0em', scale: 1 }}
          viewport={{ once: true, margin: '0px 0px -60px 0px' }}
          transition={{ ...spring.snappy, delay: delay + i * step }}>
          {ch === ' ' ? ' ' : ch}
        </m.span>
      ))}
    </span>
  )
}
```

### CountUp —— 数字滚动仪表盘

进入视口后从 0 滚到目标；`to` 变化时从当前显示值续滚（接 live 数据用）。

```jsx
import { useEffect, useRef } from 'react'
import { useReducedMotion, useInView, animate } from 'framer-motion'

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
  return <span ref={ref}>{reduced ? to : 0}</span>
}
```

### 签名交互 & 氛围背景 —— 不预置，从库现取（这里只给契约）

**签名交互**（卡片 3D 倾斜 / 边缘辉光 / 磁吸这类"高级感"效果）和**氛围背景**是这个人的视觉指纹，**一律不预置**——从 `reactbits`（微交互 / Backgrounds）或 `paper`（生成式 WebGL）现取，从他自己的方向长出来（呼应铁律 11：签名每人只留 2–3 个）。取来后必须套两样：① 第 5 节的减动效三层；② 若是 Canvas 效果，套下面这层**无视觉指纹的性能包装**。

```jsx
import { useEffect, useRef } from 'react'

// Canvas 环境层「性能 + 无障碍」契约包装 —— 视觉不预置，从库里取来画进 draw()。
// 这段本身没有任何长相，只负责：减动效不渲染 / DPR 上限 / 密度按面积+触屏减半 / 切后台暂停 / 清理。
export default function AmbientCanvas() {
  const canvasRef = useRef(null)
  useEffect(() => {
    if (window.matchMedia?.('(prefers-reduced-motion: reduce)').matches) return   // ① 减动效直接不渲染
    const canvas = canvasRef.current, ctx = canvas.getContext('2d')
    const dpr = Math.min(window.devicePixelRatio || 1, 2)                          // ② DPR 上限 2
    const coarse = window.matchMedia?.('(pointer: coarse)').matches
    let W = 0, H = 0, raf = 0, running = true
    let items = []                                                                // ← 你从库取来的效果的粒子/状态
    const build = () => {
      W = innerWidth; H = innerHeight
      canvas.width = W * dpr; canvas.height = H * dpr
      canvas.style.width = W + 'px'; canvas.style.height = H + 'px'
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0)
      const count = Math.round((W * H) / 22000 / (coarse ? 2 : 1))                 // ③ 密度按面积、触屏减半
      items = Array.from({ length: count }, () => ({ /* 你的粒子初始状态 */ }))
    }
    const draw = () => {
      if (!running) return
      ctx.clearRect(0, 0, W, H)
      // ──④ 从 reactbits / paper 取来的视觉画在这里（雪 / 尘 / 网格 / 波纹 / mesh 皆同理）──
      raf = requestAnimationFrame(draw)
    }
    const onVis = () => { running = document.visibilityState === 'visible'         // ⑤ 切后台暂停 rAF
      if (running) { cancelAnimationFrame(raf); raf = requestAnimationFrame(draw) } }
    build(); raf = requestAnimationFrame(draw)
    addEventListener('resize', build); document.addEventListener('visibilitychange', onVis)
    return () => { cancelAnimationFrame(raf); removeEventListener('resize', build)  // ⑥ 清理
      document.removeEventListener('visibilitychange', onVis) }
  }, [])
  return <canvas ref={canvasRef} aria-hidden="true"
    style={{ position: 'fixed', inset: 0, zIndex: 0, pointerEvents: 'none' }} />
}
```

> 若用 `paper` 的 WebGL 背景（`MeshGradient` 等）而非手写 Canvas，同样要：懒加载、弱机降开销、`visibilitychange` 暂停、reduced-motion 时换静帧 / 纯色。

---

## 4. 其它可直接抄的手感细节（CSS）

- **减动效全局兜底**（放在 `index.css` 底部，是三层契约的 CSS 层）：
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { animation: none !important; transition: none !important; scroll-behavior: auto !important; }
  }
  ```
- **过冲弹性缓动**（列表标记滑入等）：`transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);`
- **胶片颗粒氛围**：内联 SVG `feTurbulence fractalNoise` 作 `::after` 固定层，`opacity: .035`（避免 `mix-blend-mode`，合成开销大）。
- **Canvas / WebGL DPR 上限**：Canvas `Math.min(devicePixelRatio, 2)`，WebGL `1.75`。

---

## 5. 三层减动效契约（无障碍底线，强制）

任何动效落地都必须同时满足这三层，缺一层就漏掉一类用户：

| 层 | 判定 | 行为 |
|---|---|---|
| **JS（framer-motion）** | `useReducedMotion()` | 组件返回**静态元素**（Reveal/SplitText 已内建；从库取的交互也要补） |
| **Canvas** | `matchMedia('(prefers-reduced-motion: reduce)').matches` | **直接不渲染** / 快照静帧（`AmbientCanvas` 包装已内建 return） |
| **CSS** | `@media (prefers-reduced-motion: reduce)` | 全局 `animation/transition: none`（第 4 节兜底） |

从外部库（reactbits / paper）取来的组件**基本都不带这三层**——移植时必须由你补上。这是它们"直接能用"但"不能直接上线"的关键差别。

---

## 6. 从两库移植的注意（详见 `inspiration.md`）

- **reactbits.dev**：拷贝源码进项目，天生贴 framer-motion/Canvas 栈，几乎零改造——但要补第 5 节的减动效层。拷了实质源码要**保留版权注释 + 记入项目 `THIRD_PARTY_NOTICES.md`**。
- **shaders.paper.design**：`npm i @paper-design/shaders-react`（`MeshGradient`/`Warp`/`GrainGradient`）。WebGL 较重，**懒加载 + 弱机降开销 + 切后台暂停**。

## 7. 动效预算（用多少、怎么排 —— 通篇有动 ≠ 满屏乱动）

"通篇灵动"最容易翻车成"什么都在动 = 全是噪音、眼睛没有落点"。用下面这套主次纪律防止它变廉价。

**先给每个动效派一个角色**（一个动效只担一个角色，担不上就删）：

| 角色 | 干什么 | 例 |
|---|---|---|
| 定向 | 让人知道"在哪、往哪走" | 区块入场、滚动进度、导航状态 |
| 反馈 | 回应操作 | hover、点击、复制成功、切换类别 |
| 解释 | 帮人看懂 | 展开一个流程 / 对比 / 时间线 |
| 签名 | 这个人专属的记忆点 | 贯穿动线那类（真实项目里的"继续飞行"） |
| 氛围 | 低存在感的背景底 | Canvas 环境层（套 `AmbientCanvas` 包装）、paper 生成式底 |

**再守总量预算（硬约束）：**
- 每个大区块都给一点**轻入场或状态变化**（`Reveal` 最小量即可）——这才是"通篇有动"。
- **全站只留 2–3 个高光签名动效**，别处处高潮。
- **一屏之内最多一个高注意力效果**——别在同一屏里放俩东西抢眼。
- **全局最多一层持续氛围底**——别叠三层粒子/shader。
- 外部现取的签名效果也**≤2–3 个**；其余日常入场/反馈一律用本节原语 + `theme.js` token，别每处造轮子。

**"通篇灵动"落在哪个层级**：指每个区块都活（轻入场 / hover 反馈），**不是**每屏都有大动效抢眼。主次分明的"活"，比满屏乱动更显高级。
