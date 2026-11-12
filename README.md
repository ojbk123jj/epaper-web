# EPaperFrame Web

ESP32-S3 四色墨水屏相框的网页客户端。纯静态站点，使用 Web Bluetooth 与相框通信，不依赖后端、数据库或图片上传服务器。

## 功能

- **图片换图**：本地选择 JPG/PNG/WebP，裁剪、缩放、四色抖动预览，导出 30,000 字节面板帧并通过 BLE 发送。
- **AI 生图**：输入提示词，通过 BLE 提交给相框；相框自行联网调用 ModelScope 生成并刷屏。网页显示联网、生成、下载、转换、刷屏阶段和已用时间。
- **生成结果预览**：AI 成功后通过 BLE 回读相框实际显示的 30,000 字节四色屏幕帧，在网页中预览并可保存。
- **断线重连**：刷屏期间相框会短暂关闭 BLE，页面自动重连并查询结果。
- 照片只在浏览器本地处理；AI 提示词由相框发送到云端，网页服务器不接收提示词或照片。

## 目录

```text
.
├── index.html             页面结构
├── styles.css             页面样式
├── src/
│   ├── app.js             页面交互、图片编辑、AI 进度与预览
│   ├── ble.js             Web Bluetooth 客户端
│   ├── protocol.js        BLE 协议编码/解析
│   ├── image.js           裁剪、四色量化、帧解包
│   └── selection.js       图片选择/异步解码状态
├── tests/                 Node 内置测试框架
├── deploy/                Nginx、Certbot 配置示例
├── docs/                  部署、验证记录
├── build.mjs              静态构建脚本
└── package.json
```

## 本地开发

需要 Node.js（测试和构建）以及支持 Web Bluetooth 的 Android Chrome。

```bash
npm test
npm run dev
```

电脑打开 `http://127.0.0.1:8080` 只能预览和调试界面；手机通过普通局域网 HTTP 访问不满足 Web Bluetooth 的安全上下文要求。手机联调请使用部署后的 HTTPS 地址。

## 构建

```bash
npm run build
```

产物在 `dist/`，包含：

```text
index.html
styles.css
src/*.js
```

无需在服务器安装 npm 依赖。

## 部署

当前线上地址：

- https://lumicat.top:8443/
- https://103.117.138.156:8443/

服务器为 Ubuntu 24.04，网站根目录 `/srv/epaper-frame/site`，Nginx 使用 8443 端口以保留原有 443 服务。配置示例在 `deploy/`。

替换静态文件即可，不需要重启后端服务：

```bash
npm run build
tar -C dist -czf /tmp/epaper-web.tar.gz .
scp /tmp/epaper-web.tar.gz <user>@103.117.138.156:/tmp/
ssh <user>@103.117.138.156
sudo cp -a /srv/epaper-frame/site /srv/epaper-frame/site.bak-$(date +%F-%H%M%S)
sudo rm -rf /srv/epaper-frame/site
sudo mkdir -p /srv/epaper-frame/site
sudo tar --no-same-owner -C /srv/epaper-frame/site -xzf /tmp/epaper-web.tar.gz
sudo chown -R www-data:www-data /srv/epaper-frame/site
sudo chmod -R a+rX /srv/epaper-frame/site
rm -f /tmp/epaper-web.tar.gz
```

更完整的部署、缓存、证书和验证步骤见 [网页说明](docs/ble-web.md) 与 [验证记录](docs/ble-web-validation.md)。

## BLE 协议

固件侧 BLE 协议以固件仓库为准：

- https://github.com/ojbk123jj/epaper-frame
- `docs/ble-protocol.md`

网页端实现位于 `src/protocol.js` 和 `src/ble.js`。主要特征：

- `f0e10000-...` 服务
- `f0e10001` Info
- `f0e10002` Control
- `f0e10003` Data
- `f0e10004` Status
- `f0e10005` AI Status
- `f0e10006` Preview Data

## 相关仓库

- 固件：[ojbk123jj/epaper-frame](https://github.com/ojbk123jj/epaper-frame)
- 网页：本仓库
