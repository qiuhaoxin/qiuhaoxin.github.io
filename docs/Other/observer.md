## 前端中的几种观察者模式

### ResizeObserver

### PerformanceObserver

### MutationObserver

### IntersectionObserver

用于观察两个 HTML DOM 元素之间的交集。当 DOM 进入或离开可见的视窗(Visible Viewport)时，在 DOM 中观察一个元素是很有用的，可以在以下的场景使用该 API：

- 1、当一个元素在视窗中可见时，延迟加载图像或其他资源
- 2、识别广告的可见性并计算广告收入
- 3、实现网站的无限滚动，当用户向下滚动页面时，不必要浏览不同的页面
- 4、当一个元素在视窗中时，加载和自动播放视频或动画

使用方法的步骤：

- 1、创建观察者
- 2、定义要观察的目标对象
- 3、定义回调事件
