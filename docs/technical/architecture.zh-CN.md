# 架构

三个页面、一个共享核心、少量单一用途的构建插件。没有框架，没有状态库——每个页面就是一个 TypeScript 模块，把 DOM 接到共享代码上。

## 页面

| 目录 | 页面 | 入口 |
|---|---|---|
| `/` | 首页：卡片、分享对话框 | `home/main.ts` |
| `send/` | 文件/片段 → canvas 上的喷泉编码 QR 码流 | `send/main.ts` |
| `receive/` | 摄像头 → worker 里的 WASM QR 解码 → 喷泉解码 → 文件 | `receive/main.ts`、`receive/worker.ts` |

## 共享模块（`shared/`）

- `fountain.ts` — LT 编码器/解码器，确定性 soliton 分布（见[协议](protocol.zh-CN.md)）。
- `protocol.ts` — 帧头打包/解析、文件容器、SHA-256 校验、流标识。
- `frame-capacity.ts` — QR 容量计算：载荷大小 → 块长度/数量上限。
- `qr-raster.ts` — QR 模块矩阵 → RGBA 光栅。
- `display.ts` — 根据视口适配 QR 显示尺寸。
- `platform.ts` — `isIOS`/`isAndroid` 探测和摄像头能力探测（闪光灯、连续对焦、最大帧率）。策略：能探测就探测；只有不可探测的行为才用 UA 嗅探。
- `worker-pool.ts` — 解码 worker 池；繁忙的 worker 丢帧，喷泉码吸收掉。
- `no-signal.ts` — "没动静？"提示的纯计时策略（首次短暂延迟，关闭后变长）。
- `progress.ts` — 已收集帧数进度估算和喷泉开销模型。
- `send-settings.ts` — 标准发送设置列表；发送端下拉菜单和"无信号"建议都从这里渲染。
- `snippet.ts` — 文本片段容器类型。
- `dialog.ts` — `<dialog>` 的几何背景点击关闭。
- `share-dialog.ts` — 首页和发送端都携带的分享对话框（QR + 复制 + 系统分享面板）。
- `status-line.ts`、`wake-lock.ts`、`format.ts`、`style.css`。

## 构建插件（`build/`）

每个文件一个功能，做精确匹配的字符串手术，**匹配不到就抛错**——标记漂移会打断构建，而不是带着坏输出发布。

- `html-tokens.ts` — `%TOKEN%` 替换（站点 URL、设置选项、版本、构建 ID）。
- `root-pwa-head.ts` — 负责每个页面上的 manifest 链接和 SW 注册；校验 URL 在任意子路径下都能解析到站点根。
- `rewrite-standalone-links.ts` — 为单文件构建剥离/重写托管站点引用。
- `inline-zxing-wasm.ts`、`use-inline-variants.ts` — 为独立版内联解码 wasm/worker。
- `standalone-csp.ts`、`emit-as.ts` — 独立版 CSP 和输出命名。
- `make-icons.ts` — 从 logo 重新生成 `public/` 图标（`npm run icons`，需要 librsvg）。

类型检查：`tsconfig.json` 覆盖页面和 `shared/`；`tsconfig.node.json` 覆盖 `build/` 和 `vite.config.ts`（都在 `npm run build` 里执行）。
