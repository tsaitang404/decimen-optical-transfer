# Decimen Optical Transfer：喷泉码 QR 文件传输

只用**一块屏幕和一只摄像头**就能在两台设备之间传输文件。一个页面把文件显示为无限流动的动画 QR 码流，另一台设备用摄像头对准它，重建出文件。**设备之间没有网络路径，没有应用，没有配对，除摄像头外不需要任何权限。** 载荷以光的形式传播。

**English: [English README](README.en.md) · [中文文档目录](docs/README.zh-CN.md)**

## 试试看

### **→ [decimen.app](https://decimen.app/)**

两台设备打开就能用——无需安装任何东西。首次访问后离线可用，如果想让它在主屏幕占个位置，iOS 和 Android 都可以安装为应用。

最大 64 MB 的文件（或粘贴的文本片段），保留文件名和媒体类型，仅在确实有用时启用 gzip，任何内容展示给用户前都经过 SHA-256 校验——收到的视频还能直接在页面里播放。这个项目是从一个更大的实验里提取出来的，那个实验达到了**手机对手机 128 KB/s**。

<p align="center">
  <img src="docs/receiving.jpg" width="420"
       alt="手机通过光传输接收文件：130.5 KB/s 有效吞吐，解码发送端动画 QR 码流进行到一半" />
</p>
<p align="center"><em>传输中途：手机以 130 KB/s 从空气中拉取一个文件。</em></p>

两种模式都不加密：发送屏幕上显示的任何内容，任何对准它的摄像头都能看到。这个项目给你的是"无网络"，而不是"保密"——见[隐私](docs/user/privacy.zh-CN.md)。

## 文档

**使用** — [快速开始](docs/user/quick-start.zh-CN.md) ·
[发送](docs/user/sending.zh-CN.md) · [接收](docs/user/receiving.zh-CN.md) ·
[故障排查](docs/user/troubleshooting.zh-CN.md) ·
[安装与离线](docs/user/install-and-offline.zh-CN.md) ·
[隐私](docs/user/privacy.zh-CN.md)

**原理** — [架构](docs/technical/architecture.zh-CN.md) ·
[协议](docs/technical/protocol.zh-CN.md) ·
[平台特性](docs/technical/platform-quirks.zh-CN.md) ·
[构建与发布](docs/technical/build-and-release.zh-CN.md)

协议的简短版：屏幕到摄像头的链路没有反向通道，所以发送端流式发送喷泉编码帧（[Luby transform](https://en.wikipedia.org/wiki/Luby_transform_code)）——接收端按任意顺序收集**任意**约 K·1.15 个不同帧，就能把文件剥出来。丢帧只损失时间，绝不影响正确性。

## 自己运行

```bash
npm install
npm run dev               # 带 HMR 的 https 开发服务器
npm run serve             # 构建，然后伺服生产包
npm run demo              # 演示模式：只能发送内置载荷
npm test                  # 金标准线上格式向量与单元测试
npm run build             # 托管站点 → dist/
npm run build:standalone  # 两个自包含页面 → dist-standalone/
npm run build:all         # 全部
```

在发送设备上打开 `https://localhost:5173/send/`，在接收手机上打开打印出来的 `Network` URL（自签名证书点一次通过即可）。完整流程：[快速开始](docs/user/quick-start.zh-CN.md)。

## 类似项目

这里的思路是独立想出来的。事实证明有好几个人有类似想法，他们的方案都值得一看：

- [mohankumarelec/airgapped-qr-code-transfer](https://github.com/mohankumarelec/airgapped-qr-code-transfer)：
  基于浏览器的 QR 文件传输，带压缩和顺序分块。
  在公开演示这个项目之后发现的；趋同演化。
- [divan/txqr](https://github.com/divan/txqr)（2018）：Go 写的动画 QR 加喷泉码，有两篇关于"为什么喷泉编码胜过顺序循环"的精彩文章。
- [sz3/libcimbar](https://github.com/sz3/libcimbar)：完全超越了 QR，用专门为这个信道设计的高密度彩色编码。

由 [Evan Crawley (Bash Alarmist)](https://www.linkedin.com/in/evan-crawley) 构建，使用了
[node-qrcode](https://github.com/soldair/node-qrcode) 和
[zxing-wasm](https://github.com/Sec-ant/zxing-wasm)。

## 许可证

MIT
