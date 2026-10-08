# 更新记录

本仓库 fork 自 [yukinotech/bili-rotate](https://github.com/yukinotech/bili-rotate)，
1.0.9 起为 Thatgfsj 维护分支上的改动。

## 1.1.6 — 归仓与更新源切换

- 脚本元数据改为本仓库：`@author` / `@namespace` / `@github` / `@homepageURL` /
  `@supportURL` 指向 `Thatgfsj/bili-rotate`，新增 `@originalAuthor yukinotech` 保留署名；
- 新增 `@updateURL` / `@downloadURL` 指向本仓库 main 分支的 `index.js`，
  自动更新不再回落到 Greasy Fork 的原版；
- 新增 `@icon`；README、`package.json` 同步更新，补充 `CHANGELOG.md`。

> 注意：旧版从 Greasy Fork 安装，其 `@updateURL` 指向 Greasy Fork，
> 需要覆盖安装一次才能把更新源切到本仓库。

## 1.1.5 — 控制栏渐进构建导致按钮再次丢失

- 全新加载页面时 B 站播放器渐进构建控制栏，5 秒兜底执行后控制栏仍会再重建一次
  （视频元数据就绪时），按钮再次丢失；
- 插入逻辑抽成 `ensureRotateButton`（按钮存在即零开销返回）；
- 对播放器容器挂 `MutationObserver(subtree)`，控制栏被重建、按钮被移除时
  事件驱动地立即补回，替代任何形式的定时轮询。

## 1.1.4 — 站内跳转时视频标签晚于容器出现

- 从推荐/选集进入视频（SPA 切换，不整页刷新）时，B 站先创建视频容器、
  视频标签晚几秒才插入；
- `videoInit` 等容器内出现真实视频标签（`video` / `bwp-video`）再初始化，
  不再命中文本节点抛错；
- `buttonInit` 与 `videoInit` 改为并行执行（`Promise.all` + 容错），
  按钮只依赖控制栏，不再被视频加载阻塞；切集回调同样并行化。

## 1.1.3 — 去掉轮询式补按钮

- 1.1.2 的 2 秒轮询补按钮属于打补丁方案，恢复原版
  「`waitToGet` 等底栏 + 插入一次 + 5 秒兜底」结构（兜底时重新查找底栏节点）；
- `videoInit` 同步恢复原版时序；旋转渲染与尺寸自适应修复保持不变。

## 1.1.2 — 控制栏重建后按钮丢失

- 切清晰度/进出全屏等操作后 B 站会重建控制栏 DOM，按钮被一并移除，
  原脚本只插入一次 + 5 秒兜底（且引用的可能是已脱离文档的旧底栏节点），之后无法恢复；
- 按钮插入抽成 `ensureRotateButton`：每次重新查找底栏、检查按钮是否存在，
  丢失即补回并重新绑定事件；
- 脚本尾部加 2 秒间隔的定期兜底（1.1.3 已去掉，1.1.5 改为事件驱动）。

## 1.1.1 — 真全屏展开动画期间量到过渡尺寸

- 进入真全屏后画面只占屏幕中间（全屏前的播放器大小）：真全屏有展开动画，
  `MutationObserver` 的 +100ms 延迟重算量到的是过渡中/旧尺寸，动画结束后无机制再触发；
- 对播放器容器挂 `ResizeObserver`，容器实际尺寸稳定后自动重新执行 `resetHW`；
  `videoInit` 重建播放器时先 `disconnect` 旧监听。

## 1.1.0 — Edge/AMD 硬件视频叠加层下 90°/270° 黑屏

- 实测（Edge + AMD 显卡）：小窗口播放时旋转正常，网页全屏等大画面场景下
  旋转到 90°/270° 后播放器整块黑屏；DevTools 实测几何位置完全正确、无元素遮挡，
  属于渲染合成层问题——硬件视频叠加层（MPO）无法合成被 2D `rotate` 旋转的视频平面，
  `will-change` / `filter` / `translateZ` 都无法强制其脱离叠加层；
- 旋转改用 `rotate3d(0,0,1,deg)` 走 3D 合成路径；
- 视频元素加 `mask-image: linear-gradient(#fff,#fff)` 强制脱离视频叠加层，按普通纹理合成；
- 容器加 `perspective` 开启 3D 上下文配合 `rotate3d`。

## 1.0.9 — 旋转后黑屏/画面错位，增强健壮性

- 宽高比不再只在初始化时测量一次：优先读取 `videoWidth/videoHeight`，
  且每次 `resetHW` 重新校验，避免元数据晚到、切集后用错比例走错分支；
- 视频改为绝对定位 + `translate(-50%,-50%)` + rotate 居中旋转，任意角度、
  任意容器宽高比下画面都居中且完整可见，替代原先依赖 flex 对齐 + translate 偏移修正的方式；
- contain 宽高用统一公式计算，修复横屏视频旋转时 `-5px` 魔法数导致的偏差；
- 用 `querySelector('video,bwp-video')` 替代 `childNodes[0]`，避免命中文本节点；
- `resetHW` 增加节点有效性守卫，播放器重建期间不再对失效节点操作。

## 1.0.8 及以前

原版 yukinotech/bili-rotate 的历史版本，见
https://github.com/yukinotech/bili-rotate/commits/main 。
