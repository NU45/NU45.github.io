# 钉钉样式还原 · PWA 全屏版

把原来的单文件 HTML 改造成**可安装、可离线、全屏运行（无地址栏 / 搜索框）**的 PWA。

## 目录结构

```
dingtalk-pwa/
├── index.html              # 原页面 + 注入的 PWA 头 / 安全区适配 / 安装引导
├── manifest.json           # 应用清单：display = standalone（隐藏地址栏与搜索框）
├── sw.js                   # Service Worker：预缓存全部资源，离线可用
├── icons/
│   ├── icon-192.png
│   ├── icon-512.png
│   ├── icon-512-maskable.png
│   └── apple-touch-icon.png
└── README.md
```

## 一、部署（必须先 HTTPS）

Service Worker 只在 **https** 或 **localhost** 下生效，直接双击用 `file://` 打开不会全屏。任选一种：

- **GitHub Pages**：把整个目录推到仓库，Settings → Pages 选分支根目录，得到 `https://xxx.github.io/...`
- **Vercel / Netlify**：把目录整个拖进部署框，一键出 https 域名
- **自有服务器**：Nginx 指向该目录，配好 SSL；确保 `.json` 返回 `application/json`
- **本地调试**：`cd dingtalk-pwa && python3 -m http.server 8000`，手机与电脑同 WiFi 时访问 `http://电脑IP:8000`（注意：http 下 SW 不注册，但 Android Chrome 仍可直接安装）

## 二、手机上全屏打开

### Android（Chrome / Edge / 三星浏览器）
1. 用 https 打开页面，1 秒后自动弹出「安装到桌面」引导
2. 点「添加到主屏幕」，或点浏览器菜单里的「安装应用」
3. 桌面生成图标 → **从图标进入即全屏**，无地址栏、无搜索框

### iOS（Safari）
1. 用 Safari 打开页面，自动弹出引导
2. 点底部（或右上角）**分享按钮** → 选择 **「添加到主屏幕」**
3. **从桌面图标打开**才是全屏；在 Safari 里直接输网址访问，地址栏和搜索框仍在（这是 iOS 的限制，网页无法绕过）

## 三、关键实现点

| 需求 | 实现 |
|---|---|
| 隐藏搜索框 / 地址栏 | `manifest.json` 的 `"display": "standalone"` + `display_override: ["standalone","fullscreen","minimal-ui"]` |
| iOS 全屏 | `<meta name="apple-mobile-web-app-capable" content="yes">` + `apple-mobile-web-app-status-bar-style` |
| 可安装 + 离线 | `sw.js` 预缓存 `index.html` / `manifest.json` / 图标，stale-while-revalidate 更新 |
| 刘海屏 / 底部小黑条 | `viewport-fit=cover` + `env(safe-area-inset-*)` 补丁（已加在固定顶栏、底部导航、详情页底栏） |
| 图标 | 蓝渐变圆角方块 + 「钉」字，含 maskable 版本（Android 自适应图标不会被裁切） |

## 四、可调项

- 想更彻底全屏（连状态栏都盖住）：`manifest.json` 里改 `"display": "fullscreen"`，并把 `apple-mobile-web-app-status-bar-style` 改成 `black-translucent`（此时顶部内容会被状态栏压住，需自行加 `padding-top`）。
- 不想要安装引导浮层：删掉 `index.html` 中 `<!-- ===== PWA：注册 Service Worker + 添加到主屏幕引导 ===== -->` 那一整段。
- 改应用名 / 主题色：编辑 `manifest.json` 的 `short_name`、`theme_color`，同步改 `index.html` 里的 `<meta name="theme-color">`。

## 五、常见问题

- **页面还是显示地址栏**：说明是在浏览器里直接访问，不是从桌面图标启动。Android 请完成安装，iOS 请添加到主屏幕后再从图标进入。
- **没有弹出安装提示**：Android Chrome 需 https + SW 注册成功；确认 Console 无 `SW 注册失败`。
- **更新后内容没变**：SW 是 stale-while-revalidate，第二次进入才是新版；或改 `sw.js` 里的 `CACHE` 版本号强制刷新。
