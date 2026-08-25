<p align="center">
  <img src="docs/assets/whale-girl-ios-app-promo-16x9.png" alt="DeepSeek Harness Mobile 与移动网关" width="100%">
</p>

# dsh-plugin-mobile-gateway

让 iPhone 通过经过设备鉴权的 WebSocket 连接 DeepSeek Harness。安装后，Harness WebUI 左侧边栏会出现“移动设备”入口，可直接开启网关、生成配对二维码和管理可信设备。

- WebSocket：`/ws/mobile`
- 局域网：`ws://<局域网 IP>:3081/ws/mobile`
- Linux 服务器公网：`wss://<公网 IP>/ws/mobile`
- 协议文档：[PROTOCOL.md](PROTOCOL.md)

## 配套 iOS 客户端

[DeepSeek Harness Mobile](https://github.com/Clarklevis1995/dsh-mobile) 是本仓库的兄弟项目。它是面向 iOS 17+ 的 SwiftUI 原生客户端，支持工作区与会话、工作区内创建文件夹、历史和实时对话、图片、Agent 执行轨迹、Human-in-the-loop、模型与权限设置。

<table>
  <tr>
    <td width="33.33%" align="center"><img src="https://raw.githubusercontent.com/Clarklevis1995/dsh-mobile/main/Docs/images/screenshots/home.png" alt="iOS 工作区首页" width="100%"></td>
    <td width="33.33%" align="center"><img src="https://raw.githubusercontent.com/Clarklevis1995/dsh-mobile/main/Docs/images/screenshots/conversation-dark.png" alt="iOS 深色对话界面" width="100%"></td>
    <td width="33.33%" align="center"><img src="https://raw.githubusercontent.com/Clarklevis1995/dsh-mobile/main/Docs/images/screenshots/pairing-dark.png" alt="iOS 设备配对界面" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><strong>工作区首页</strong></td>
    <td align="center"><strong>实时对话</strong></td>
    <td align="center"><strong>设备配对</strong></td>
  </tr>
</table>

## 安装插件

前提：已经安装 `dsh` CLI 和 `pnpm`，并能正常启动 `dsh web`。可先执行 `pnpm --version` 确认当前用户的环境能够找到 `pnpm`。

局域网使用只需安装插件：

```bash
dsh plugin --profile web add dsh-plugin-mobile-gateway@latest
```

需要在 Linux 服务器通过公网 IP 接入时，推荐执行统一初始化命令。它会安装/更新插件，并请求一次 sudo 权限安装系统 Helper：

```bash
npx --yes dsh-plugin-mobile-gateway@latest init
```

`init` 会把当前正在执行的 npm 包精确版本安装到 DSH profile，并仅对该版本跳过 pnpm 的新版本等待期，确保插件与 Helper 版本一致。

安装后停止并重新启动 WebUI：

```bash
dsh web
```

打开 WebUI，确认左侧边栏底部出现“移动设备”。

## 局域网配对

适用于 DSH 电脑和 iPhone 位于同一个可互访的局域网。

1. 打开 WebUI 的“移动设备”。
2. 开启“允许移动设备连接”。
3. 保持“设备鉴权”开启。
4. 确认面板显示 `ws://<电脑局域网 IP>:3081/ws/mobile`。
5. 填写设备名称并点击“生成配对二维码”。
6. 在 iOS 客户端打开“设备认证”，扫描二维码。
7. WebUI 的可信设备显示“在线”后即完成。

如果系统防火墙拦截连接，只允许私有网络访问 TCP `3081`。不要把 3081 开放到公网。

## Linux 服务器公网 IP 配对

> [!NOTE]
> 本文所说的“公网 IP 配对”特指 Linux 服务器。一键公网安装从 `v0.6.4` 开始提供，适用于带固定公网 IPv4 的 Ubuntu/Debian 服务器。服务器需要已经安装 Node.js、`pnpm` 和 `dsh` CLI；当前尚不支持 CentOS。

### 1. 准备公网端口

在云厂商控制台复制服务器的公网 IPv4，并在安全组中放行入站 TCP `80` 和 `443`。不要将 DSH WebUI 端口或 TCP `3081` 开放到公网。

### 2. 一键初始化

先确认普通 DSH 用户可以直接调用 `pnpm`：

```bash
pnpm --version
```

然后使用同一个普通用户执行（不要使用 `root` 或 `sudo npx`）：

```bash
npm_config_registry=https://registry.npmjs.org \
npx --yes dsh-plugin-mobile-gateway@latest init
```

该命令会安装或更新插件，并请求一次 sudo 权限安装 Nginx、Certbot、系统 Helper 和证书续期定时器。`init` 会确保 DSH profile 与 Helper 使用同一个精确版本。

完成后启动或重新启动 WebUI：

```bash
dsh web
```

### 3. 打开远程 WebUI

优先使用 VS Code、Cursor 等 IDE 自带的端口转发。也可以在自己的电脑执行：

```bash
ssh -N -L <本地端口>:127.0.0.1:<DSH 实际端口> <服务器用户名>@<服务器公网 IP>
```

然后在本地浏览器打开：

```text
http://127.0.0.1:<本地端口>
```

### 4. 在 UI 配置公网入口

打开左侧的“移动设备”，在“公网接入”填写云厂商控制台提供的公网 IPv4，然后点击“配置公网接入”或“更新公网配置”。Helper 会自动读取当前 `dsh web` 端口并配置 Nginx、TLS 证书和 `wss://<公网 IP>/ws/mobile`。

<p align="center">
  <img src="docs/assets/public-access-ui.png" alt="在移动设备面板配置公网接入" width="420">
</p>

### 5. 配对移动设备

1. 开启“允许移动设备连接”，保持“设备鉴权”开启。
2. 填写设备名称并点击“生成配对二维码”。
3. iPhone 打开“设备认证”并扫描二维码。
4. WebUI 的可信设备显示“在线”后即完成。

二维码只能使用一次，并会在 5 分钟后过期；超时后在 WebUI 重新生成即可。

## Windows / macOS 家用电脑远程连接

家用电脑通常没有固定公网 IP，不建议配置路由器端口转发。可以使用 Tailscale 长期连接，或使用 Cloudflare Quick Tunnel 临时调试。两种方式都转发到插件专用的 `3081` 端口，不会公开 DSH WebUI。

使用前先启动 `dsh web`，并在“移动设备”面板开启“允许移动设备连接”和“设备鉴权”。

### Tailscale（推荐长期使用）

1. 在电脑和 iPhone 安装 [Tailscale](https://tailscale.com/download)，并登录同一个 Tailnet。
2. 在 Windows PowerShell 或 macOS 终端执行：

```bash
tailscale serve --bg 3081
```

3. 执行 `tailscale serve status` 查看生成的 `https://<设备名>.<tailnet>.ts.net` 地址。
4. 将地址改为 `wss://<设备名>.<tailnet>.ts.net/ws/mobile`，填入 WebUI 的“WebSocket 地址”，再生成二维码配对。

Tailscale Serve 只允许同一 Tailnet 中符合访问规则的设备连接，并自动提供 HTTPS。可用 `tailscale serve reset` 停止转发。参见 [Tailscale Serve 文档](https://tailscale.com/docs/reference/tailscale-cli/serve)。

### Cloudflare Quick Tunnel（仅临时调试）

1. 安装 [cloudflared](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/downloads/)。
2. 在 Windows PowerShell 或 macOS 终端执行：

```bash
cloudflared tunnel --url http://127.0.0.1:3081
```

3. 命令行会显示随机的 `https://<随机名称>.trycloudflare.com` 地址。
4. 将地址改为 `wss://<随机名称>.trycloudflare.com/ws/mobile`，填入 WebUI 的“WebSocket 地址”，再生成二维码配对。

保持该命令运行；停止命令后隧道立即失效。Quick Tunnel 的地址每次可能变化，且没有可用性保证，不适合正式或长期使用。公网调试时必须保持“设备鉴权”开启。参见 [Cloudflare Quick Tunnel 文档](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/)。

## 网关配置方式总览

| 使用场景 | 推荐入口 | iOS WebSocket 地址 | 需要的额外配置 | 端口与鉴权 |
|---|---|---|---|---|
| 同一局域网 | 插件局域网入口 | `ws://<电脑局域网 IP>:3081/ws/mobile` | 无需 Helper 或 Nginx；电脑与 iPhone 位于可互访的局域网 | 仅对私有网络放行 TCP `3081`；保持鉴权开启 |
| 本机 iOS 模拟器 | DSH WebUI 本地入口 | `ws://127.0.0.1:<DSH WebUI 端口>/ws/mobile` | 无需 Helper、Nginx 或独立的 `3081` 端口 | 不开放任何外部端口；仅 Debug 时可关闭鉴权 |
| Linux 公网服务器 | 插件 Helper + Nginx + TLS | `wss://<服务器公网 IPv4>/ws/mobile` | 执行 `init`，再从 WebUI 填写公网 IPv4 | 云安全组放行 TCP `80/443`；不要公开 DSH 端口和 `3081`；必须鉴权 |
| 家用 Windows / macOS | Tailscale Serve；临时调试可用 Quick Tunnel | `wss://<Tailscale 域名>/ws/mobile` 或 `wss://<随机名称>.trycloudflare.com/ws/mobile` | 隧道转发到 `127.0.0.1:3081`，将生成的地址填入 WebUI | 无需路由器端口转发；保持鉴权开启 |

## Linux 服务器公网入口管理

查看状态：

```bash
sudo env "PATH=$PATH" npx --yes dsh-plugin-mobile-gateway@latest status
```

移除公网入口：

```bash
sudo env "PATH=$PATH" npx --yes dsh-plugin-mobile-gateway@latest remove
```

## 更新插件

重新运行初始化命令会同时更新插件和系统 Helper：

```bash
npx --yes dsh-plugin-mobile-gateway@latest init
```

随后停止并重新启动 `dsh web`。

## 卸载插件

```bash
dsh plugin --profile web remove dsh-plugin-mobile-gateway
```

如需同时移除系统 Helper（不会删除现有 Nginx 公网配置）：

```bash
sudo env "PATH=$PATH" npx --yes dsh-plugin-mobile-gateway@latest remove-helper
```

## 常见问题

| 现象 | 处理方式 |
|---|---|
| WebUI 没有“移动设备” | 确认安装在 `web` profile，并完整重启 `dsh web` |
| iOS 收到 `503` | 回到 WebUI 开启“允许移动设备连接” |
| iOS 收到 `401` | 在 WebUI 重新生成二维码并配对 |
| Linux 服务器公网连接超时 | 检查云安全组、服务器防火墙和 TCP `80/443` |
| Linux 服务器公网地址没有显示 | 在“移动设备 → 公网接入”填写公网 IPv4 并点击更新 |
| 需要查看服务端日志 | 执行 `tail -f /tmp/mobile-gateway.log` |

## 源码开发

```bash
dsh plugin --profile web add file:/absolute/path/to/dsh-plugin-mobile-gateway
npm test
```

源码修改后需要重新安装插件并重启 `dsh web`。
