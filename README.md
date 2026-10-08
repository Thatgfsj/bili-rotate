# b站 播放器旋转插件（bilibili player rotate）

B 站视频播放器旋转油猴脚本：播放器控制栏多出一个旋转按钮，点一下顺时针转 90°，
竖屏视频、网页全屏、真全屏下都能正确铺满并居中。

- **本仓库（当前维护版本）**：https://github.com/Thatgfsj/bili-rotate
- 原版仓库：https://github.com/yukinotech/bili-rotate （原作者 yukinotech）
- 原版 Greasy Fork：https://greasyfork.org/zh-CN/scripts/438224

本仓库是原版的 fork，脚本头部 `@author` / `@namespace` / `@github` / `@updateURL` /
`@downloadURL` 均已指向本仓库 main 分支。

## 安装

安装地址（Tampermonkey 打开即会弹出安装页）：

```
https://raw.githubusercontent.com/Thatgfsj/bili-rotate/main/index.js
```

1. 浏览器先装好 Tampermonkey（油猴）扩展；
2. 打开上面的地址安装；也可以在 Tampermonkey 面板 → **实用工具** → **从 URL 安装**，填入该地址；
3. 之后在 Tampermonkey 里点「检查更新」即可从本仓库拉取新版本。

> **从 Greasy Fork 装过旧版的话请覆盖安装一次。** 旧记录的 `@updateURL` 指向
> Greasy Fork，油猴的自动更新会把 fork 上的改动覆盖回原版；用上面的地址覆盖安装后，
> 更新源才会切到本仓库。

## 使用说明

安装完成后打开 B 站视频页（`http*://*.bilibili.com/video/*`），播放器底栏右侧会多出一个
旋转图标，点击即可旋转（0° → 90° → 180° → 270° 循环）。

![image](readme-img/img1.jpg)

## 目录结构

```
.
├── index.js             # 唯一产物：完整的油猴脚本（= Tampermonkey 里保存的内容）
├── dev.js               # 开发用引导脚本：只负责把 index.js 从本地 dev server 注入页面
├── dev-server/main.js   # fastify 静态服务器，在 3000 端口提供 index.js
├── rotate.svg           # 旋转图标（脚本内联 SVG，同时用作脚本 @icon）
├── readme-img/          # README 配图
├── package.json         # 版本号 / 作者 / dev 依赖
└── CHANGELOG.md         # 版本改动记录
```

## 主要模块说明（index.js）

| 模块 | 作用 |
| --- | --- |
| `waitToGet(fn, time)` | 轮询等待 DOM 出现（B 站播放器是异步、渐进构建的） |
| `getRatio()` | 取视频原始高宽比：优先 `videoWidth/videoHeight`，退化到 computed style，最后回落到上次结果 |
| `videoInit()` | 定位 `.bilibili-player-video` / `.bpx-player-video-wrap`，等真实 `video/bwp-video` 就绪后设置定位与基础样式，并挂 `ResizeObserver` |
| `resetHW()` | 按当前角度和容器尺寸重新计算元素宽高（contain 贴合），并写 `translate(-50%,-50%) rotate3d(...)` |
| `rotate()` | 角度 +90° 后调用 `resetHW()` |
| `ensureRotateButton()` | 幂等地把旋转按钮插入控制栏，已被移除就补回 |
| `buttonInit()` | 等底栏出现 → 插入按钮 → 5 秒兜底 |
| `MutationObserver` × 3 | ① 播放器容器尺寸/属性变化 ② 站内切集、画中画触发重新初始化 ③ 播放器子树变化时补回按钮 |

### 几个关键实现细节

- **绝对定位 + `translate(-50%,-50%)` 居中旋转**：元素中心始终钉在容器中心，
  任意角度、任意容器宽高比（含非 16:9 的网页全屏）下画面都居中且完整可见，
  不依赖 flex 对齐和 translate 偏移修正。
- **用 `rotate3d(0,0,1,deg)` 而不是 2D `rotate()`**，并给视频元素加
  `mask-image: linear-gradient(#fff,#fff)`：强制视频离开硬件视频叠加层（MPO），
  解决 Edge/AMD 显卡下旋转 90°/270° 整块黑屏（布局正确但像素不显示）的问题。
- **`ResizeObserver` 监听容器实际尺寸**：真全屏有展开动画，MutationObserver 的
  延迟重算会量到过渡中的尺寸，等容器尺寸稳定后再算一次才能铺满。
- **事件驱动补按钮**：控制栏会被 B 站重建（渐进构建、切清晰度、进出全屏），
  用 `MutationObserver(subtree)` 立即补回，避免定时轮询的开销。

## 开发指南

### 方式一：dev server（推荐，改完刷新页面即生效）

```bash
npm install
npm run dev        # fastify 在 http://localhost:3000 提供 index.js
```

把 `dev.js` 的内容粘贴到 Tampermonkey 里新建脚本并保存，此后只改 `index.js`、
刷新 B 站页面即可生效，不必反复编辑油猴里的脚本。

### 方式二：直接改 Tampermonkey 里的脚本

把 `index.js` 全文粘贴进 Tampermonkey 编辑器保存。改完记得同步回本仓库并 `git push`，
否则油猴「检查更新」会把线上版本覆盖回来。

## 版本

见 [CHANGELOG.md](CHANGELOG.md)，当前版本 **1.1.6**。

## License

MIT。原版版权归 yukinotech 所有，本 fork 的改动同样以 MIT 发布。
