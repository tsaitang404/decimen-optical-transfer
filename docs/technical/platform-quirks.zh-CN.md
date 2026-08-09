# 平台特性

沉淀进代码里的血泪经验，免得别人重新踩一遍。

## 摄像头

- **iOS 会谎报帧率。** `frameRate: {ideal: 60}` 会静默变成 30；要 `{exact: 60}`（1280 宽度下可用），然后回退到 `ideal`。始终读回 `getSettings()`。
- **iOS 可能拒绝活着的流上的 `applyConstraints`。** 接收端保留正在运行的流并如实告知，而不是拆掉一次传输。
- **能力靠探测，不做 UA 嗅探**（`shared/platform.ts`）。Android Chrome 通过 `getCapabilities()` 暴露 `torch`、`focusMode`、`frameRate.max`；iOS 一个都不暴露。可用时启用连续自动对焦；不可达的 fps 选项被禁用。`torch` 会被报告但刻意不用——发送端是一块发光的屏幕，手电筒只会增加眩光。
- **`requestVideoFrameCallback` 链比流活得久**，并在下一个流上继续；一个代际计数器防止僵尸采集循环。

## QR 解码

Safari 从未发布过 `BarcodeDetector`（WebKit bug 281848），所以解码用的是编译成 WASM 的 [zxing-cpp](https://github.com/zxing-cpp/zxing-cpp)，跑在 worker 里——这是唯一可移植的路径。

## 媒体播放

**iOS Safari 不会可靠地播放 `<video>`/`<audio>` 里的 `blob:` URL** —— AVFoundation 想要真正的 HTTP 语义，包括 Range 请求。接收到的媒体进入 Cache API，通过一个 workbox `rangeRequests` 路由以真实 URL（`received-media`）伺服出去；当没有 service worker 控制页面时，blob URL 是回退方案，另外还有一个 `error` 事件回退，防止 AVFoundation 完全绕过 SW。

## Safari 26 "Liquid Glass" 界面染色

Safari 26 忽略 `theme-color`，改为**采样页面 CSS——尤其是固定定位层——并锁存采样结果**来给界面 / 安全区域条染色。两个后果已经内置处理：

- `html` 带显式 `background-color`（透明根元素会被采样为*白色*）。
- 发送端点击全屏的 QR **不是固定覆盖层**——它是一个页面状态（`body.qr-full`），隐藏其他一切，让舞台在正常文档流里填满视口。流式内容在回流时重绘；没有固定层可以让染色锁存。（所有覆盖变体——固定白色、固定透明带绝对定位白色子层、安全区域 inset 覆盖层——在真实设备上关闭后都会残留白色条带。）

## 杂项 UI

- **16px 输入下限**：移动端 Safari 在更小的控件获得焦点时会放大页面；每个设置控件都付 16px，而不是锁定视口缩放。
- **粘性 `:hover`**：iOS 会把 `:hover` 锁在最后一次点按的目标上——任何在触摸时需要*被看到*的状态必须是静止样式，而不是 hover 样式。
- **`<dialog>` 焦点**：`showModal()` 聚焦第一个按钮，iOS 会把它画成预高亮；焦点改送到标题上（`tabindex="-1" autofocus`）。
- **背景点击关闭必须是几何判断**（`shared/dialog.ts`）：对话框子元素之间的空隙也满足 `event.target === dialog`，所以只查 target 会在普通点击时误关。
- **`hidden` 与 display**：任何同时给带 `hidden` 属性的元素设置 `display` 的规则，都需要显式的 `[hidden] { display: none }` 配套。
