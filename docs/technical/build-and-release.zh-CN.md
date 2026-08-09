# 构建与发布

## 脚本

```bash
npm run dev               # 带 HMR 的 https 开发服务器（自签名证书）
npm run serve             # 构建，然后伺服生产包
npm run demo              # VITE_DEMO=1 的开发服务器 —— 发送端锁定为内置载荷
npm test                  # 金标准线上格式向量和单元测试（node --test via tsx）
npm run build             # 类型检查（app + node 配置）、托管站点 → dist/
npm run build:standalone  # 两个自包含页面 → dist-standalone/
npm run build:all         # 全部
npm run icons             # 从 logo 重新生成 public/ 图标（需要 librsvg）
```

`npm run icons` 在栅格化前剥离 logo SVG 的注释（注释里的 `--` 是浏览器容忍但 librsvg 拒绝的非法 XML），并对标记做精确匹配手术，如果 logo 形状变了就抛错。

`VITE_SITE_URL` 覆盖烘焙进社交卡片和分享对话框的发布 URL（默认 `https://decimen.app/`，必须以斜杠结尾）。

## PWA / Service Worker

`vite-plugin-pwa`（workbox）预缓存一切，包括 940 KB 的解码 wasm。有两处是自定义的（`build/root-pwa-head.ts`）：manifest/SW 引用被重写为从任意页面深度解析到站点根（构建会校验这一点）；注册脚本做 skip-waiting 握手——新部署只需一次刷新就能接管已打开的页面，而不是永远伺服旧预缓存。一个 workbox `rangeRequests` 路由从 Cache API 伺服接收到的媒体（见[平台特性](platform-quirks.zh-CN.md)）。

## CI（`.github/workflows`）

- **`ci.yml`** — 每次推送到 `main` / `release/*` 和每个 PR 都跑测试和构建。断言伺服出去的 `receive` 块保持在 20 KB 以下（防止内联的 worker/wasm 泄漏进站点构建），并断言 manifest/SW 引用指向存在的文件。
- **`pages.yml`** — 每次推送到 `main` 部署到 GitHub Pages。
- **`release.yml`** — 打 `v*` 标签时：构建一切，附加 `decimen-<tag>-site.zip`、两个独立文件，以及 `SHA256SUMS.txt`。

站点用 `base: "./"` 构建，所以放在项目的任意子路径下都不需要配置。

## 发布流程

1. `git checkout -b release/vX.Y.Z`，升版本（`npm version X.Y.Z --no-git-tag-version`），提交。
2. 在分支上做功能开发 + 文档；PR 到 `main`。
3. 合并后打标签 `vX.Y.Z` — `release.yml` 构建并附加产物。

页脚盖 `v<version> · build <short-hash>`（构建里有未提交改动时带 `-dirty`），所以任何产物都能精确追溯到源码。
