# 网络出口检测

无需构建、没有后端的单页网络检测工具。浏览器会访问不同类别的公开 IP 探针，比较它们看到的公网出口，用于辅助观察国内直连、国际代理、特殊规则和多出口状态。

线上地址：<https://ip.foxtang.com>

## 原理与探针

页面分别请求当前、国内、国际和特殊线路。若分流软件对这些域名采用不同规则，探针可能看到不同 IP。页面只根据 IP 差异作“疑似”判断，相同 IP 不代表未用代理。

| 类别 | 首选 | fallback |
| --- | --- | --- |
| 当前 | `www.cloudflare.com/cdn-cgi/trace` | — |
| 国内 | `myip.ipip.net/json` | IPIP 文本、IPInfo |
| 国际 | `api64.ipify.org` | ident.me、icanhazip |
| Google 可达性 | `www.google.com/generate_204` | 只判断实际连接成功/失败，不伪造出口 IP |
| 协议 | `api4.ipify.org` / `api6.ipify.org` | — |

请求禁用缓存且有 5 秒超时；某一接口失败不会中断其他卡片。HTTP 延迟不是 ICMP Ping。地理信息失败时仍显示 IP。

## 部署

本仓库可直接启用 GitHub Pages（`main` 分支根目录），也可运行 `npx wrangler deploy` 部署到 Cloudflare Workers Static Assets。`wrangler.jsonc` 通过 Worker Route 接管 `ip.foxtang.com/*`，并保留该主机现有 DNS 记录。

## 已知限制

- Google 官方没有提供可供任意网页跨域读取的访客出口 IP 回显接口，因此页面只实际检测 Google 可达性和 HTTP 耗时，不声称能取得 Google 出口 IP。
- 国内/国际出口结果取决于代理软件对探针域名的规则，不代表服务器物理位置或精确规则。
- 免费接口可能限流、失效或调整 CORS；国内 fallback 不保证仍命中同类国内规则。
- DNS、CDN 和 IPv4/IPv6 连接策略会影响结果；三个线路返回的 IP 协议不一致时，页面不会强行比较；地理、ASN、ISP 数据可能有误差。
- 本站没有后端或数据库；第三方探针会按各自政策处理请求。

支持最新版 Chrome、Edge、Safari（macOS / iOS）。复制功能需要安全上下文与 Clipboard API。

本地预览：`python3 -m http.server 8080`，打开 <http://localhost:8080>。
