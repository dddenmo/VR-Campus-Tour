# VR漫游校园 开发记录

## 项目概览
- 单文件 HTML（~17MB），所有媒体资源 base64 内嵌
- 框架：A-Frame 1.5.0 + Three.js
- 部署：Cloudflare Pages（主）→ `https://vr-campus-tour.pages.dev/`
- 备用：GitHub Pages → `https://dddenmo.github.io/VR-Campus-Tour/`
- 仓库：`https://github.com/dddenmo/VR-Campus-Tour.git`（branch: main）

---

## 功能说明

### 加载页
- 页面打开立刻显示（`#loading-screen` 是 `<body>` 第一个元素）
- 黑底 + spinner + 进度条（模拟进度至85%，`window.onload` 后跳到100%）
- 加载完成后 0.5s 淡出消失
- 之所以不用模糊封面图做底：会出现闪烁，改为纯黑底更干净

### 场景结构
- 场景1：教室全景（Image01）→ 点击光点进入场景2
- 场景2：走廊全景（Image02）→ 点击光点进入场景3
- 场景3：第三全景（Image03）→ 点击光点触发结尾
- 结尾：ending 视频（含音轨）淡入替代场景3

### 场景切换时序
- 场景1→2：overlay 黑屏 0.3s 淡入 → 切换全景 → 0.7s 淡出
- 场景2→3：同上
- 场景3→结尾：scene-container 0.4s 淡出，同时 ending 视频 0.2s 淡入（延迟20ms启动）

### 气泡对话
- 每个场景有若干气泡按时序出场，超出4条时旧气泡上移并隐藏最旧的
- 场景1第5条气泡：陀螺仪用户显示"转动看看，当年你坐在哪"，非陀螺仪显示"滑动看看，当年你坐哪"
- 场景1最后（16.5s）：陀螺仪用户专属气泡"多转几圈，仔细看看，一闪一闪的"
- 场景切换时清除所有气泡计时器并隐藏气泡

### 粒子特效（lizi）
- 场景3进入后 6s 出现，1.2s 渐显
- 点击光点触发场景结束

### 陀螺仪
- 所有移动端进入时弹出授权弹窗（统一 iOS + Android）
- iOS：调用 `DeviceOrientationEvent.requestPermission()` 系统授权
- Android：无需授权，直接开启
- A-Frame 使用 `magicWindowTrackingEnabled` 属性控制开关
- 设置面板中有陀螺仪开关，可随时切换
- `device-orientation-permission-ui="enabled: false"` 关闭 A-Frame 自带弹窗

### 设置面板
- 音效开关（SFX）：控制气泡音效，通过 `window._sfxEnabled` 标志位判断是否播放（`cloneNode()` 不继承 JS 属性，所以不用 volume 控制）
- BGM音量滑条：同时控制 hero-bgm、bgm、npc-bgm 三个音频元素
- iOS 隐藏音量滑条（iOS `volume` 属性只读，JS无法控制音量）
- 陀螺仪开关：实时控制 A-Frame look-controls

### 全屏按钮
- iOS 无 Fullscreen API → 隐藏全屏按钮
- 非 iOS 移动端 / 桌面端：原生全屏

### 按钮布局
- 设置 + 全屏按钮位于右上角，flex column 竖排（设置在上，全屏在下）
- 容器：`#ui-btn-group`，`position: absolute; top: 4vh; right: 4vh`

---

## 平台限制备忘

| 平台 | 限制 |
|------|------|
| iOS Safari | `volume` 只读，无法 JS 控制音量；无 Fullscreen API；`100vh` ≠ 实际可见高度（用 `inset:0` 替代） |
| Android | 陀螺仪无需授权但需统一弹窗提示用户 |
| Cloudflare Workers | 免费版 10ms CPU 限制，17MB 文件会返回 ERR_EMPTY_RESPONSE → 改用 Cloudflare Pages |

---

## 关键决策

- **`cloneNode()` 的坑**：复制 audio 元素时不会继承 JS 设置的 `.volume` 值，只继承 HTML 属性。SFX 开关改为 `if (window._sfxEnabled !== false)` 判断是否 `.play()`
- **加载页位置**：必须是 `<body>` 第一个元素，否则浏览器先渲染 splash 封面再显示进度条
- **settings 面板 pointer-events**：`#settings-frame` 默认 `pointer-events: none`，打开时必须显式设为 `auto`，否则 iOS Safari 事件无法穿透到子元素
- **部署选择**：Cloudflare Pages 直接托管静态文件，无 CPU 限制，CDN 分发，速度最快

---

## 提交记录摘要（按时间）

1. 替换背景音乐 AreYouLost.mp3 → AreYouLost02.mp3（7MB→2.7MB）
2. 缩小 lizi 粒子特效回原始大小
3. 替换粒子特效 webm01→webm00
4. 修复场景2气泡卡顿与音效不同步
5. 放大 lizi 粒子特效2倍
6. 场景3粒子出现时机 8s→6s
7. 修复加载页（移至 body 首位、黑底、iOS 底部缺口）
8. 修复 SFX 开关、BGM 音量、iOS 音量隐藏、settings pointer-events
9. 统一移动端陀螺仪弹窗（iOS + Android）
10. 场景1陀螺仪专属气泡文案 + 隐藏滑动引导图标
11. 场景1陀螺仪用户新增"多转几圈"提示（16.5s）
12. 修复"多转几圈"气泡在场景2未消失的问题
13. 设置+全屏按钮移至右上角，flex 容器布局
14. 按钮布局改为纵排（column）
15. 加快结尾画面切换：淡入淡出 0.8s→0.4s
16. 加快结尾视频淡入：0.4s→0.2s
