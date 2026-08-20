# 2026年优质翻墙机场推荐与故障排查综合指南

> 2026年精选稳定高速SSR/V2ray节点机场推荐 | 长期更新，助您畅游全球互联网
> 适用于 Windows/macOS/Android/iOS/Linux 等全平台 Clash 客户端配置

---

## 📌 快速导航

- [ClashVIP](#-clashvip) | [VPSVIP](#-vpsvip) | [nav.clashvip.net](#-navclashvipnet) | [clashvip.net](#-clashvipnet) | [clashhub.net](#-clashhubnet) | [bbs.clashhub.net](#-bbsclashhubnet) | [clash-for-windows.net](#-clash-for-windowsnet) | [vpsvip.net](#-vpsvipnet)

## 🔥 近期重点推荐

| 机场 | 延迟 | 速度 | 推荐指数 | 特色 |
|------|------|------|----------|------|
| ClashVIP | ⚡ 极快 | ⚡ >500Mbps | ⭐⭐⭐⭐⭐ | 旗舰节点 全年稳定 |
| Netflex/Di | ⚡ 极快 | ⚡ >500Mbps | ⭐⭐⭐⭐⭐ | 流媒体解锁专家 |
| VPSVIP 专线 | ⚡ 极快 | ⚡ >500Mbps | ⭐⭐⭐⭐⭐ | IEPL专线 低延迟 |

---

## ⚡ ClASH VIP 专线推荐 ⚡

> **[ClashVIP 官网注册](https://clashvip.net)**

| 套餐 | 流量 | 价格 | 适合场景 |
|------|------|------|----------|
| 月付 | 500GB/月 | ¥29/月 | 尝鲜用户 |
| 年付 | 2TB/月 | ¥199/年 | 主流选择 |
| 旗舰专线 | 5TB/月 | ¥399/年 | 重度用户/团队 |

---

## 📖 目录导航

1. [常见故障清单](#一常见故障清单)
2. [故障诊断流程](#二故障诊断流程)
3. [解决方案库](#三解决方案库)
4. [预防措施](#四预防措施)
5. [故障日志分析](#五故障日志分析)
6. [应急响应预案](#六应急响应预案)
7. [诊断脚本工具箱](#七诊断脚本工具箱)
8. [推广链接汇总](#八推广链接汇总)

---

# 一、常见故障清单

本章节系统梳理使用翻墙服务过程中最常遇到的六类典型故障，帮助用户快速定位问题根因。每类故障均标注了发生概率、影响范围及典型症状，便于在出现异常时快速对号入座。

## 1.1 连接超时

**发生概率**：★★★★☆（非常高）
**影响范围**：所有翻墙客户端
**典型症状**：客户端显示"连接超时"、"Connection Timeout"、"连接失败"、"Dial timeout"等错误信息，长时间无法建立任何连接，节点列表中大量节点显示红色不可用状态。

**常见诱因**：

- 运营商对翻墙流量实施了深度包检测（DPI）并执行封锁策略
- 目标节点服务器的 IP 地址已被长城防火墙（GFW）纳入黑名单
- 本地网络存在全局代理或 DNS 污染，导致域名解析阶段就已失败
- 节点所在服务器的网络带宽达到上限或服务器过载
- 客户端配置的代理端口被防火墙拦截（如 7890、1080 等常用端口被封）

**初步判断方法**：尝试访问 `https://www.google.com`（国内直连测试网站），若直连正常但代理无法连接，则极大概率是翻墙通道被阻断；若直连同样超时，则为本地网络问题而非翻墙服务故障。

## 1.2 节点失效

**发生概率**：★★★☆☆（较高）
**影响范围**：单个或多个特定节点
**典型症状**：部分节点无法连接，表现为延迟显示为"×"或"-1"，连接后立即断开，Speedtest 测速为零或极低。同一机场的其他节点可能完全正常。

**常见诱因**：

- 节点 IP 被 GFW 主动封锁（IP 被墙），这是最常见原因，通常发生在特定敏感时期前后
- 节点服务器遭遇 DDoS 攻击，服务商临时封禁了部分端口
- 节点服务器所在数据中心被墙或出口带宽受限
- 机场主机的上游供应商调整了路由策略
- 节点配置参数（UUID、AlterId 等）发生变更但订阅未同步

**初步判断方法**：访问 [IPinfo.io](https://ipinfo.io) 输入节点 IP，查看该 IP 的归属地及信誉评分；若显示为中国大陆 IP 归属或被标记为"Blocked"，则该节点 IP 已被封锁。在多个地点部署测试（如使用 Looking Glass）可确认是否为区域性封锁。

## 1.3 速度骤降

**发生概率**：★★★★☆（非常高）
**影响范围**：单个或所有节点
**典型症状**：连接正常建立，但实际网速远低于标称速率。例如标称 500Mbps 的节点实测仅 10Mbps 以下，视频播放频繁缓冲，文件下载速度异常缓慢。Speedtest 测速结果与日常基线差距超过 50%。

**常见诱因**：

- GFW 在晚高峰期间对翻墙流量进行限速（晚 8 点至凌晨 1 点为高发时段）
- 节点服务器带宽被大量用户共享导致资源争抢
- 本地网络带宽不足（如 4G 移动网络、共享 WiFi 等）
- 运营商对代理端口实施 QoS 限速（常见于非标准端口）
- 客户端开启了不必要的流量压缩或中转功能
- DNS 污染导致部分请求绕回国内，形成"假代理"状态

**初步判断方法**：对比同一节点在不同时段的速度表现；若仅在晚间速度下降，大概率为 GFW 限速；若全天均慢，需排查本地网络或客户端配置。同时检查是否有多个设备同时使用同一账号。

## 1.4 订阅失效

**发生概率**：★★★☆☆（较高）
**影响范围**：全部节点
**典型症状**：客户端内节点列表突然清空，显示"订阅解析失败"、"Subscription parse error"、节点数量归零或仅剩默认示例节点。重新更新订阅后问题依旧。

**常见诱因**：

- 订阅链接过期或被重置（部分机场每周更换订阅地址以防滥用）
- 订阅更新频率限制，短时间内频繁刷新触发临时封禁
- 订阅文件托管服务（如 GitHub Pages、Coding Pages）在国内无法访问
- 浏览器或客户端缓存了旧的订阅 URL，访问的是过期版本
- 机场服务商更新了订阅加密密钥或格式，但客户端未同步更新
- 账户欠费或被封禁，订阅服务被主动停止

**初步判断方法**：手动复制订阅链接在浏览器中打开（建议使用导入过证书的浏览器或绕过代理访问），观察返回内容是否为 Base64 编码的节点列表；若浏览器无法访问该 URL，说明订阅源不可达。

## 1.5 配置错误

**发生概率**：★★★☆☆（较高）
**影响范围**：单个或多个节点
**典型症状**：特定节点显示配置正确但无法连接，或连接成功后所有流量仍然直连（代理未生效）。TUN 模式开启后系统网络异常，部分应用无法上网。规则配置错误导致目标网站无法访问。

**常见诱因**：

- 配置文件中的 UUID、端口、密码、加密方式与机场提供的信息不匹配（复制粘贴时漏掉字符）
- 混合使用多个机场订阅导致配置冲突
- 规则集中使用了过时的规则语法，导致新版客户端解析失败
- TUN 模式与本地 VPN 冲突（常见于已安装其他 VPN 软件的系统）
- 系统代理设置被其他软件篡改（特别是国产浏览器和安全软件）
- 节点的时间不同步（部分协议对时间敏感，偏差超过 5 分钟会导致验证失败）

## 1.6 证书与加密问题

**发生概率**：★★☆☆☆（中等）
**影响范围**：特定协议节点
**典型症状**：TLS 握手失败提示"certificate verify failed"、"tls: first record does not look like a TLS handshake"，WebSocket 连接建立后立即断开。常见于 VLESS、Trojan 等依赖 TLS 传输的协议。

**常见诱因**：

- 服务器证书过期或被吊销
- 客户端未正确导入根证书，导致无法验证服务器身份
- 使用了自签名证书但未配置跳过证书验证（仅限测试环境）
- TLS 指纹被识别为代理特征被阻断（SNI 探测）
- 伪装域名（域名前置）配置错误

---

# 二、故障诊断流程

当遇到翻墙故障时，遵循标准化的五步诊断流程可以高效定位问题根源，避免盲目操作浪费时间。以下流程适用于 Windows、macOS、Android、iOS 和 Linux 各平台，步骤顺序不可颠倒，因为前一步的结论直接影响后续判断。

## 2.1 症状识别

**目标**：确认故障现象，收集关键信息，建立问题基线。

**操作要点**：

- **记录错误信息**：完整记录客户端显示的错误代码和文字描述，包括时间戳。典型错误信息如"connection refused"、"ETIMEDOUT"、"EHOSTUNREACH"、"BadRequest"、"timeout after 30s"等，各自对应不同的故障类型。
- **确定影响范围**：是单个节点故障还是所有节点均无法使用？是特定应用无法走代理还是全局失效？是间歇性故障还是持续性故障？
- **记录环境信息**：操作系统版本、客户端版本、节点套餐类型、故障开始时间、是否在敏感时期前后。
- **尝试复现**：关闭客户端重新启动，切换节点，切换网络（切换 WiFi/有线/移动数据），观察问题是否持续。

**症状分类矩阵**：

| 症状类型 | 可能原因 | 优先级 |
|----------|----------|--------|
| 所有节点超时 | 网络阻断/订阅失效 | P0 最高 |
| 部分节点失效 | 节点 IP 被墙 | P1 高 |
| 速度骤降 | GFW 限速/服务器过载 | P1 高 |
| 单节点不稳定 | 节点质量/配置错误 | P2 中 |
| 规则失效 | 规则语法/版本问题 | P2 中 |

## 2.2 初步排查

**目标**：通过最小化操作快速排除最常见的故障原因。

**网络连通性测试**：

```powershell
# PowerShell — 测试基础网络连通性
# 第一步：测试 DNS 解析
nslookup www.google.com 8.8.8.8
# 若返回超时 → DNS 故障，检查本地 DNS 设置

# 第二步：测试 TCP 端口连通性（以节点端口为例）
Test-NetConnection -ComputerName "节点域名或IP" -Port 443
# 若 TcpTestSucceeded = False → 端口被阻断或节点离线

# 第三步：测试本地代理端口是否被占用
Get-NetTCPConnection -LocalPort 7890 -ErrorAction SilentlyContinue
# 若无返回 → 代理端口未被监听，检查客户端启动状态
```

**订阅有效性验证**：

```powershell
# PowerShell — 验证订阅链接可访问性
$headers = @{
    "User-Agent" = "ClashForWindows/0.20.39"
}
try {
    $response = Invoke-WebRequest -Uri "你的订阅链接" `
        -Headers $headers `
        -TimeoutSec 15 `
        -UseBasicParsing
    Write-Host "订阅响应状态码: $($response.StatusCode)"
    Write-Host "内容长度: $($response.Content.Length) 字符"
    if ($response.Content.Length -lt 100) {
        Write-Warning "订阅内容过短，可能已失效或被重定向"
    }
} catch {
    Write-Error "订阅链接无法访问: $_"
}
```

**检查时间同步**：

```bash
#!/bin/bash — Linux/macOS 时间同步检查
# 检查系统时间与 NTP 服务器偏差
timedatectl status
ntpdate -q pool.ntp.org 2>&1 | head -5

# 若时间偏差超过 5 分钟，TLS 握手将失败
# 解决方案：sudo ntpdate -s pool.ntp.org
```

## 2.3 深度诊断

**目标**：针对初步排查中发现的疑点进行深入测试，确定精确的故障位置。

**节点连通性深度测试**：

```powershell
# PowerShell — 逐节点深度连通性测试
$nodeList = @(
    @{Name="节点A"; Host="node-a.example.com"; Port=443},
    @{Name="节点B"; Host="node-b.example.com"; Port=443},
    @{Name="节点C"; Host="node-c.example.com"; Port=443}
)

foreach ($node in $nodeList) {
    Write-Host "`n=== 测试 $($node.Name) ===" -ForegroundColor Cyan
    
    # TCP 握手时间测试
    $sw = [Diagnostics.Stopwatch]::StartNew()
    $tcpTest = Test-NetConnection -ComputerName $node.Host -Port $node.Port -WarningAction SilentlyContinue
    $sw.Stop()
    
    if ($tcpTest.TcpTestSucceeded) {
        Write-Host "  [OK] TCP 连通: $($sw.ElapsedMilliseconds)ms" -ForegroundColor Green
    } else {
        Write-Host "  [FAIL] TCP 阻断, RTT=$($sw.ElapsedMilliseconds)ms" -ForegroundColor Red
        Write-Host "  RemoteAddress: $($tcpTest.RemoteAddress)" -ForegroundColor Yellow
    }
    
    # 解析 IP 归属
    try {
        $geo = (Invoke-RestMethod "https://ipinfo.io/$($tcpTest.RemoteAddress)/json" -TimeoutSec 5)
        Write-Host "  IP 归属: $($geo.country) - $($geo.org)" -ForegroundColor Gray
        if ($geo.country -eq "CN") {
            Write-Host "  [警告] IP 归属中国，大概率已被 GFW 封锁!" -ForegroundColor Magenta
        }
    } catch {}
}
```

**TLS 指纹与 SNI 测试**：

```bash
#!/bin/bash — TLS 指纹及 SNI 探测测试
# 用于判断节点是否因 TLS 指纹识别被阻断

TARGET="节点域名"
PORT=443

echo "=== TLS 握手测试 ==="
openssl s_client -connect ${TARGET}:${PORT} -servername ${TARGET} \
    </dev/null 2>&1 | openssl x509 -noout -dates -issuer 2>/dev/null

echo "=== SNI 一致性检测 ==="
# 测试 Host 头与证书 SAN 是否匹配
openssl s_client -connect ${TARGET}:${PORT} \
    -servername ${TARGET} 2>&1 | grep -E "subject|issuer| SAN" | head -10

echo "=== JA3 指纹提取 ==="
# 提取客户端 TLS 指纹用于与正常浏览器对比
echo "请访问 https://ssl-client.fliger.es 验证正常浏览器指纹"

echo "=== 伪装域名可用性 ==="
curl -I --sni-name ${TARGET} \
    -H "Host: ${TARGET}" \
    "https://${TARGET}/" \
    --connect-timeout 5 2>&1 | head -5
```

## 2.4 解决验证

**目标**：确认修复操作生效，防止"表面修复"。

**四层验证法**：

1. **基础连通验证**：客户端状态显示已连接，节点延迟显示正常数值（非 -1 或 ×）。
2. **代理流量验证**：访问 `https://api.ip.sb/ip` 确认出口 IP 已变更为境外 IP，且 IP 地址与节点所在地一致。
3. **应用层验证**：打开被封锁网站（如 Google、YouTube、Twitter），确认可以正常访问且内容加载完整。
4. **持续性验证**：保持连接 30 分钟以上，观察速度是否稳定，是否出现频繁断线。

```powershell
# PowerShell — 完整验证脚本
function Test-ProxyEffective {
    param([string]$ProxyHost, [int]$ProxyPort = 7890)
    
    Write-Host "`n=== 代理有效性四层验证 ===" -ForegroundColor Cyan
    
    # L1: 本地代理端口状态
    $portCheck = Get-NetTCPConnection -LocalPort $ProxyPort -ErrorAction SilentlyContinue
    if ($portCheck) {
        Write-Host "  [L1 OK] 代理端口 $ProxyPort 监听中" -ForegroundColor Green
    } else {
        Write-Host "  [L1 FAIL] 代理端口未监听" -ForegroundColor Red
        return
    }
    
    # L2: 出口 IP 验证（使用系统代理）
    try {
        $beforeProxy = Invoke-RestMethod "https://api.ip.sb/ip" -TimeoutSec 8
        $env:HTTP_PROXY = "http://127.0.0.1:$ProxyPort"
        $env:HTTPS_PROXY = "http://127.0.0.1:$ProxyPort"
        $afterProxy = Invoke-RestMethod "https://api.ip.sb/ip" -TimeoutSec 8
        Remove-Item Env:HTTP_PROXY -ErrorAction SilentlyContinue
        Remove-Item Env:HTTPS_PROXY -ErrorAction SilentlyContinue
        
        Write-Host "  直连 IP: $beforeProxy"
        Write-Host "  代理 IP: $afterProxy"
        if ($beforeProxy -ne $afterProxy) {
            Write-Host "  [L2 OK] 代理通道已建立" -ForegroundColor Green
        } else {
            Write-Host "  [L2 FAIL] 代理未生效，流量仍走直连" -ForegroundColor Red
        }
    } catch {
        Write-Host "  [L2 FAIL] 出口 IP 检测失败: $_" -ForegroundColor Red
    }
    
    # L3: 封锁网站访问验证
    try {
        $env:HTTP_PROXY = "http://127.0.0.1:$ProxyPort"
        $env:HTTPS_PROXY = "http://127.0.0.1:$ProxyPort"
        $test = Invoke-WebRequest "https://www.google.com" -TimeoutSec 10 -UseBasicParsing
        Remove-Item Env:HTTP_PROXY -ErrorAction SilentlyContinue
        Remove-Item Env:HTTPS_PROXY -ErrorAction SilentlyContinue
        Write-Host "  [L3 OK] Google 访问成功 (HTTP $($test.StatusCode))" -ForegroundColor Green
    } catch {
        Write-Host "  [L3 FAIL] Google 访问失败" -ForegroundColor Red
    }
    
    Write-Host "`n=== 验证完成 ===`n" -ForegroundColor Cyan
}

# 使用方法（假设代理端口为 7890）
Test-ProxyEffective -ProxyPort 7890
```

## 2.5 预防建议

**目标**：总结本次故障的经验教训，制定长期预防策略（详见第四章预防措施）。

---

# 三、解决方案库

针对第一章节中列出的各类故障，本章节提供经过实战验证的具体解决步骤、操作命令和配置文件修改方案。每个方案均标注适用场景、操作难度和预计生效时间。

## 3.1 连接超时解决方案

### 方案 A：切换节点域名（CDN 兜底）

**适用场景**：节点 IP 被墙但域名仍可用
**操作难度**：★☆☆☆☆
**预计生效**：即时

**操作步骤**：

1. 联系机场客服获取备用域名（通常为 `backup1.xxx.com`、`cdn.xxx.net` 等）
2. 打开 Clash 客户端，进入「代理」或「配置」页面
3. 找到对应节点的服务器地址（Server Address）字段
4. 将原域名替换为备用域名，端口（Port）和其他参数保持不变
5. 保存配置，重新连接

**PowerShell 批量替换脚本**：

```powershell
# PowerShell — 批量替换订阅中的被墙域名
param(
    [Parameter(Mandatory=$true)]
    [string]$SubscriptionUrl,
    [Parameter(Mandatory=$true)]
    [string]$OldDomain,
    [Parameter(Mandatory=$true)]
    [string]$NewDomain,
    [string]$OutputFile = "$env:TEMP\clash_new.yaml"
)

$headers = @{ "User-Agent" = "ClashForWindows/0.20.39" }

Write-Host "正在获取订阅内容..." -ForegroundColor Cyan
$raw = Invoke-WebRequest -Uri $SubscriptionUrl -Headers $headers -UseBasicParsing
$content = [System.Text.Encoding]::UTF8.GetString(
    [System.Convert]::FromBase64String($raw.Content)
)

Write-Host "原始节点数: $(($content -split 'server:').Count - 1)" -ForegroundColor Gray

# 替换域名（保留端口）
$newContent = $content -replace [regex]::Escape($OldDomain), $NewDomain

$newCount = ($newContent -split 'server:').Count - 1
Write-Host "替换后节点数: $newCount" -ForegroundColor Gray

# 编码输出
$bytes = [System.Text.Encoding]::UTF8.GetBytes($newContent)
$base64 = [System.Convert]::ToBase64String($bytes)
$base64 | Out-File -FilePath $OutputFile -Encoding UTF8

Write-Host "`n已生成新订阅文件: $OutputFile" -ForegroundColor Green
Write-Host "内容长度: $($base64.Length) 字符 (Base64)" -ForegroundColor Gray
```

### 方案 B：更换加密协议（应对 TLS 指纹识别）

**适用场景**：GFW 使用 TLS 指纹识别阻断连接
**操作难度**：★★☆☆☆
**预计生效**：即时

**操作步骤**：

1. 在机场后台切换协议类型，例如从 VLESS + TLS 切换至 Trojan-Go 或 Shadowsocks + v2ray-plugin（WebSocket）
2. 获取新的节点配置信息（协议类型、服务器地址、端口、UUID/密码等）
3. 在客户端中手动添加新节点或更新订阅
4. 启用"自定义 CA 证书"或"跳过证书验证"选项（测试用，正式使用建议使用正确证书）
5. 开启"域名前置"（Domain Fronting），将 SNI 伪装为 `www.bing.com` 或 `www.amazon.com`

**关键配置示例（Clash Meta 格式）**：

```yaml
# Trojan-Go 配置示例（抗 TLS 指纹识别）
proxies:
  - name: "Trojan-伪装Amazon"
    type: trojan
    server: cdngateway.example.com      # 使用 Cloudflare CDN 节点
    port: 443
    password: "你的UUID"
    sni: www.amazon.com                # SNI 伪装为 Amazon
    alpn:
      - h2
      - http/1.1
    skip-cert-verify: false             # 生产环境必须 false
    ws: true                            # 启用 WebSocket 传输混淆
    ws-path: "/trojan-ws"
    ws-headers:
      Host: www.amazon.com             # HTTP Host 头伪装

# 规则示例：强制 Google 系走代理
rules:
  - DOMAIN-SUFFIX,google.com, Trojan-伪装Amazon
  - DOMAIN-SUFFIX,youtube.com, Trojan-伪装Amazon
  - GEOIP,CN,DIRECT
  - MATCH, Trojan-伪装Amazon
```

### 方案 C：启用 UDP over TCP（解决 UDP 被阻断）

**适用场景**：游戏、视频通话等 UDP 应用完全无法使用
**操作难度**：★★☆☆☆
**预计生效**：即时

**操作说明**：在 Clash 客户端设置中，将 `mixed-port` 的流量模式改为"UDP over TCP"或"H2"。部分客户端支持在节点高级设置中启用 `udp-relay` 并配置 `udp-over-tcp: true`。注意：开启后会略微增加延迟，但能有效规避 UDP 阻断。

## 3.2 节点失效解决方案

### 方案 A：一键切换优质节点

**适用场景**：现有节点 IP 已被封锁
**操作难度**：★☆☆☆☆
**预计生效**：即时

**操作步骤**：

1. 打开机场官网个人后台，查看最新公告或节点列表
2. 寻找标记为"新节点"、"备用节点"或"CDN 节点"的选项
3. 优先选择与当前地理位置较近的节点（日本、新加坡、香港为亚太首选）
4. 若机场提供"智能选线"功能，启用该功能让系统自动分配最优节点
5. 订阅链接中通常包含所有节点，无需手动添加，刷新订阅即可获取新节点

**节点选择参考表**：

| 目标用途 | 推荐地区 | 推荐协议 | 注意事项 |
|----------|----------|----------|----------|
| 视频流媒体 | 日本/新加坡 | Trojan/VLESS | 优先选有流媒体解锁的节点 |
| 游戏加速 | 台湾/韩国 | Shadowsocks | 选延迟低于 80ms 的节点 |
| 学术搜索 | 美国/日本 | VLESS | 注意避开版权敏感节点 |
| 日常浏览 | 香港/台湾 | 任意 | 优先选有回国优化的节点 |
| 文件下载 | 新加坡/日本 | Trojan-Go | 选带宽上限较高的节点 |

### 方案 B：订阅刷新与同步

**适用场景**：订阅过期或节点列表未更新
**操作难度**：★☆☆☆☆
**预计生效**：1-5 分钟

**操作步骤**：

1. 复制当前订阅链接
2. 在浏览器中直接打开，查看内容是否为 Base64 编码的 YAML/JSON 文本
3. 若浏览器中内容为空或被重定向，说明订阅链接已失效，联系机场获取新链接
4. 若内容正常，复制浏览器地址栏中的完整 URL（注意不要截断），粘贴到客户端订阅输入框
5. 在客户端中点击"更新订阅"，等待节点列表刷新完成

## 3.3 速度骤降解决方案

### 方案 A：调整 DNS 设置

**适用场景**：DNS 污染导致的"假代理"现象（代理已连接但流量实际直连）
**操作难度**：★☆☆☆☆
**预计生效**：即时

**操作说明**：DNS 污染会让部分域名解析到国内 IP，导致流量绕过代理。修改系统 DNS 为境外可信 DNS（推荐 Google `8.8.8.8` / `8.8.4.4` 或 Cloudflare `1.1.1.1`），同时在 Clash 中启用"Fake-IP"模式（推荐）或"Redir-Host"模式（精确但较慢）。

```powershell
# PowerShell — 一键修改 Windows DNS 并启用 Clash Fake-IP
# 以管理员权限运行

Write-Host "=== 配置 DNS + Clash Fake-IP ===" -ForegroundColor Cyan

# 步骤1：获取当前网络适配器名称
$adapters = Get-NetAdapter | Where-Object {$_.Status -eq "Up" -and $_.InterfaceDescription -notmatch "Virtual|Bluetooth|Loopback"}
Write-Host "可用网络适配器:"
$adapters | Select-Object Name, InterfaceDescription, Status | Format-Table

$adapterName = $adapters[0].Name  # 自动选择第一个活跃适配器

# 步骤2：配置 DNS
Set-DnsClientServerAddress -InterfaceAlias $adapterName -ServerAddresses ("8.8.8.8","8.8.4.4")
Write-Host "DNS 已修改: 8.8.8.8 / 8.8.4.4" -ForegroundColor Green

# 步骤3：修改 Clash 配置启用 Fake-IP
$clashConfig = "$env:APPDATA\Clash for Windows\config.yaml"
if (Test-Path $clashConfig) {
    $content = Get-Content $clashConfig -Raw
    if ($content -notmatch 'dns:') {
        Write-Host "未找到 DNS 配置段，跳过" -ForegroundColor Yellow
    } elseif ($content -match 'enable:\s*true') {
        Write-Host "Fake-IP 模式已启用" -ForegroundColor Green
    }
}

Write-Host "`n请重启 Clash 客户端并刷新订阅使配置生效`n" -ForegroundColor Cyan
```

### 方案 B：更换节点与时段策略

**适用场景**：GFW 晚间限速
**操作难度**：★☆☆☆☆
**预计生效**：即时

**操作说明**：GFW 在北京时间晚 8 点至次日凌晨 1 点对翻墙流量实施较严格的 QoS 限速。建议在此时间段切换至 IEPL/IPLC 专线节点（虽然价格较高但不受 GFW 限速影响），或提前在带宽充裕的时段完成大文件下载。对于持续速度慢的情况，优先选择晚间使用人数较少的新节点（通常标记为"新上线"）。

## 3.4 订阅失效解决方案

### 方案 A：修复订阅链接

**适用场景**：订阅链接格式错误或被截断
**操作难度**：★☆☆☆☆
**预计生效**：即时

**操作步骤**：

1. 登录机场官网 → 个人中心 → 复制完整的订阅地址（以 `https://` 开头，以 `.yaml` 或 `.txt` 结尾，中间无换行）
2. 若使用邮件收到订阅链接，检查是否有换行符被截断，复制全部内容
3. 将订阅链接粘贴到 [Base64 解码网站](https://www.base64decode.org) 验证是否为有效的 YAML/JSON
4. 若解码后为乱码，说明订阅内容被压缩（Gzip），需要使用支持自动解压的客户端

### 方案 B：手动导入节点

**适用场景**：订阅完全不可用
**操作难度**：★★☆☆☆
**预计生效**：即时

**操作说明**：当订阅系统全面故障时，可使用机场提供的"一键订阅"链接备用地址，或直接在节点列表页面复制单个节点的配置信息（服务器地址、端口、协议、UUID、密码等），在客户端中手动逐个添加。

## 3.5 配置错误解决方案

### 方案 A：重置客户端配置

**适用场景**：配置文件损坏或被篡改
**操作难度**：★★☆☆☆
**预计生效**：即时

**操作说明**：

1. 完全关闭 Clash 客户端
2. 备份并删除配置文件夹（Windows：`%APPDATA%\Clash for Windows`；macOS：`~/.config/clash-for-windows`）
3. 重新启动客户端，此时客户端会自动创建默认配置文件
4. 重新添加订阅链接，重新配置基础参数

### 方案 B：禁用安全软件冲突

**适用场景**：系统代理被安全软件拦截
**操作难度**：★★☆☆☆
**预计生效**：即时

**操作说明**：360 安全卫士、腾讯电脑管家、火绒等国产安全软件会主动拦截系统代理设置。建议将 Clash 客户端添加到这些安全软件的信任白名单，或在运行代理时临时关闭安全软件的"流量监控"和"网络防护"功能。强烈推荐卸载国产安全软件，使用系统自带 Windows Defender 或 macOS 的安全功能。

---

# 四、预防措施

"治未病"胜于"治已病"。本章节提供的预防措施可将故障发生概率降低 80% 以上，大幅减少紧急排障的时间成本。

## 4.1 定期检测机制

### 周度节点健康检测

建议每周执行一次节点健康检测，提前发现即将失效的节点。

```powershell
# PowerShell — 周度节点健康检测脚本
param(
    [Parameter(Mandatory=$true)]
    [string]$SubscriptionUrl,
    [int]$TimeoutMs = 3000,
    [int]$MaxConcurrent = 20
)

$ErrorActionPreference = "SilentlyContinue"
$headers = @{
    "User-Agent" = "ClashForWindows/0.20.39"
}

Write-Host "=== 周度节点健康检测 $(Get-Date -Format 'yyyy-MM-dd HH:mm') ===" -ForegroundColor Cyan

# 获取订阅
$raw = Invoke-WebRequest -Uri $SubscriptionUrl -Headers $headers -UseBasicParsing
$yaml = [System.Text.Encoding]::UTF8.GetString(
    [System.Convert]::FromBase64String($raw.Content)
)

# 提取所有 server 节点
$nodeRegex = [regex]'(?m)^  - name: "(.+?)"\n    type: (.+?)\n    server: (.+?)\n    port: (\d+)'
$nodes = $nodeRegex.Matches($yaml)

Write-Host "共检测 $($nodes.Count) 个节点...`n" -ForegroundColor Gray

# 并发检测
$jobs = @()
$scriptBlock = {
    param($node, $timeout)
    $result = Test-NetConnection -ComputerName $node.Groups[3].Value -Port $node.Groups[4].Value -WarningAction SilentlyContinue -InformationLevel Quiet
    [PSCustomObject]@{
        Name    = $node.Groups[1].Value
        Type    = $node.Groups[2].Value
        Server  = $node.Groups[3].Value
        Port    = $node.Groups[4].Value
        Alive   = $result
    }
}

$runspacePool = [RunspaceFactory]::CreateRunspacePool(1, $MaxConcurrent)
$runspacePool.Open()
$tasks = @()

foreach ($node in $nodes) {
    $ps = [PowerShell]::Create().AddScript($scriptBlock).AddParameter('node',$node).AddParameter('timeout',$TimeoutMs)
    $ps.RunspacePool = $runspacePool
    $tasks += [PSCustomObject]@{ Handle = $ps.BeginInvoke(); PS = $ps }
}

$results = @()
foreach ($task in $tasks) {
    $results += $task.PS.EndInvoke($task.Handle)
    $task.PS.Dispose()
}
$runspacePool.Close()

# 输出报告
$alive = ($results | Where-Object {$_.Alive}).Count
$dead  = ($results | Where-Object {-not $_.Alive}).Count

Write-Host "健康节点: $alive / $($results.Count)" -ForegroundColor Green
Write-Host "失效节点: $dead" -ForegroundColor $(if($dead -gt 0){'Red'}else{'Green'})

if ($dead -gt 0) {
    Write-Host "`n失效节点详情:" -ForegroundColor Yellow
    $results | Where-Object {-not $_.Alive} | Format-Table Name, Server, Port -AutoSize
}

# 建议切换
$recommend = $results | Where-Object {$_.Alive} | Select-Object -First 5
Write-Host "`n推荐切换节点:" -ForegroundColor Cyan
$recommend | Format-Table Name, Type, Server -AutoSize
```

## 4.2 自动切换策略

### 多机场冗余配置

建议同时订阅 2-3 个不同机场，使用分流规则实现自动故障转移。当主机场全部节点不可用时，自动切换至备用机场。

```yaml
# Clash 分流规则 — 自动切换机场
proxy-groups:
  # 主机场代理池
  - name: "主机场"
    type: url-test
    proxies:
      - 香港-01
      - 日本-01
      - 新加坡-01
      - 台湾-01
    url: "https://www.google.com/generate_204"
    interval: 300        # 每5分钟检测一次
    tolerance: 50        # 延迟容差

  # 备用机场代理池
  - name: "备用机场"
    type: url-test
    proxies:
      - 备用香港
      - 备用日本
    url: "https://www.google.com/generate_204"
    interval: 300
    tolerance: 80

  # 自动故障转移组
  - name: "自动选路"
    type: fallback        # 故障自动切换
    proxies:
      - 主机场
      - 备用机场
    url: "https://www.google.com/generate_204"
    interval: 180         # 每3分钟检测
    # 注意: fallback 模式按顺序优先使用第一个可用的组

rules:
  # Google 系走主机场
  - DOMAIN-SUFFIX,google.com, 自动选路
  - DOMAIN-SUFFIX,googleapis.com, 自动选路
  - DOMAIN-SUFFIX,youtube.com, 自动选路
  - DOMAIN-SUFFIX,github.com, 自动选路
  # Netflix 等流媒体走备用机场（如主机场有解锁能力则优先主机场）
  - DOMAIN-KEYWORD,netflix, 主机场
  - DOMAIN-KEYWORD,disney, 备用机场
  # 国内直连
  - GEOIP,CN,DIRECT
  # 默认兜底
  - MATCH, 自动选路
```

## 4.3 订阅自动刷新

### 定时订阅更新（Cron / Task Scheduler）

订阅更新频率建议：

- 普通机场订阅：每 6-12 小时自动刷新一次
- 高频更新机场：每 1-2 小时刷新一次
- 紧急备机场：每 30 分钟刷新一次（敏感时期）

```bash
#!/bin/bash — Linux/macOS 自动订阅刷新
# 添加到 crontab: */60 * * * * /path/to/update-clash.sh

SUBSCRIPTION_URL="你的订阅链接"
CLASH_CONFIG_DIR="$HOME/.config/clash"
CONFIG_FILE="$CLASH_CONFIG_DIR/config.yaml"
LOCK_FILE="/tmp/clash-update.lock"

# 防止并发执行
if [ -f "$LOCK_FILE" ]; then
    echo "[$(date)] 上次更新未完成，跳过"
    exit 0
fi
touch "$LOCK_FILE"

echo "[$(date)] 开始更新订阅..."

# 下载并解码订阅
curl -s -L -A "ClashForWindows/0.20.39" "$SUBSCRIPTION_URL" | \
    base64 -d > "$CONFIG_FILE.new"

if [ $? -eq 0 ] && [ -s "$CONFIG_FILE.new" ]; then
    mv "$CONFIG_FILE.new" "$CONFIG_FILE"
    echo "[$(date)] 订阅更新成功" | tee -a /var/log/clash-update.log
else
    echo "[$(date)] 订阅更新失败" | tee -a /var/log/clash-update.log
    rm -f "$CONFIG_FILE.new"
fi

rm -f "$LOCK_FILE"
```

## 4.4 日志监控告警

配置客户端日志级别为 `info` 或 `warning`，定期检查异常日志模式。

```yaml
# Clash 日志配置（config.yaml 中添加）
# 日志级别: silent / error / warning / info / debug
log-level: info

# 日志输出到文件（需要客户端支持）
# external-controller: 127.0.0.1:9090
# secret: "your-secret-password"
```

---

# 五、故障日志分析

日志是故障诊断最权威的信息来源。翻墙客户端的日志中通常包含完整的请求路径、响应状态和错误堆栈，学会解读日志能够将故障定位时间从数小时缩短至数分钟。

## 5.1 日志收集方法

### Windows 客户端日志导出

Clash for Windows（CFW）客户端日志路径：

- 主日志：`%APPDATA%\Clash for Windows\logs\clash-win32.log`
- 系统托盘日志：右键托盘图标 →  Logs

```powershell
# PowerShell — 导出 Clash 日志并提取错误信息
$logDir = "$env:APPDATA\Clash for Windows\logs"
$exportDir = "$env:USERPROFILE\Desktop\clash-logs-$(Get-Date -Format 'yyyyMMdd-HHmmss')"
New-Item -ItemType Directory -Path $exportDir -Force | Out-Null

# 复制最近24小时的日志
Get-ChildItem $logDir -Filter "*.log" | Where-Object {
    $_.LastWriteTime -gt (Get-Date).AddDays(-1)
} | Copy-Item -Destination $exportDir

Write-Host "日志已导出到: $exportDir" -ForegroundColor Cyan

# 提取错误关键词
$logs = Get-ChildItem $exportDir -Filter "*.log" | Get-Content -Raw
$errorPatterns = @(
    @{Name="连接超时"; Pattern="timeout|TIMEOUT|ETIMEDOUT"},
    @{Name="TLS错误"; Pattern="tls:.*handshake|x509|certificate"},
    @{Name="认证失败"; Pattern="auth|unauthorized|permission denied"},
    @{Name="DNS失败"; Pattern="dns.*fail|lookup.*error|NXDOMAIN"},
    @{Name="代理拒绝"; Pattern="connection refused|ECONNREFUSED|refused to proxy"}
)

foreach ($p in $errorPatterns) {
    $matches = [regex]::Matches($logs, $p.Pattern, [Text.RegularExpressions.RegexOptions]::IgnoreCase)
    if ($matches.Count -gt 0) {
        Write-Host "`n[$($p.Name)] 出现 $($matches.Count) 次" -ForegroundColor Yellow
        $matches | Select-Object -First 3 | ForEach-Object {
            Write-Host "  ...$($_.Value)..." -ForegroundColor Gray
        }
    }
}
```

### macOS / Linux 日志收集

```bash
#!/bin/bash — macOS/Linux 日志收集
LOG_DIR="$HOME/.config/clash-for-windows/logs"
EXPORT_FILE="/tmp/clash-log-$(date +%Y%m%d-%H%M%S).tar.gz"

if [ -d "$LOG_DIR" ]; then
    tar -czf "$EXPORT_FILE" "$LOG_DIR"/*.log 2>/dev/null
    echo "日志已打包: $EXPORT_FILE"
else
    echo "日志目录不存在: $LOG_DIR"
fi

# 实时查看日志（Ctrl+C 退出）
echo "实时日志监控 (最近50行):"
tail -n 50 "$HOME/.config/clash-for-windows/logs/clash.log" 2>/dev/null || \
    echo "日志文件不存在或为空"
```

## 5.2 异常模式识别

以下是在翻墙日志中最常见的错误模式及其对应的根因分析：

**模式一："Dial failed"**

```
[WARNING] [xxxx] dial [节点名] error: 
  connection refused: /x.x.x.x:443: NextProtoDot negotiation failed
```

**根因分析**：目标端口被主动拒绝（非超时），说明 GFW 已确认该端口为代理端口并执行 Reset。该节点已完全失效，需立即切换。

**模式二："TLS handshake timeout"**

```
[WARNING] [xxxx] xxxx tcp connection failed: 
  i/o timeout after 30s
```

**根因分析**：TCP 三次握手超时完成，说明连接被路由至 GFW 层面的黑洞路由（数据包被丢弃而非回复 RST）。GFW 正在进行主动探测或已封锁该 IP 段。

**模式三："Invalid HTTP response"**

```
[WARN] http: proxy response &{Status:403 Forbidden} 
  without a body, requires "X-Forwarded-For" to access
```

**根因分析**：代理服务器返回 403，常见于节点 IP 被识别为数据中心 IP 而非住宅 IP（Google 等平台对数据中心 IP 有严格限制）。

**模式四："subscription update failed"**

```
[ERROR] [subscriber] failed to fetch subscription: 
  context deadline exceeded
```

**根因分析**：订阅 URL 访问超时。通常是运营商对 GitHub 或特定订阅托管商实施了 DNS 劫持或 TCP 阻断。解决方案：使用代理访问订阅链接或更换浏览器。

---

# 六、应急响应预案

本预案旨在为突发性、大规模翻墙故障提供系统化的响应框架，确保在紧急情况下能够快速恢复服务。

## 6.1 故障分级标准

| 级别 | 名称 | 描述 | 响应时限 | 处置策略 |
|------|------|------|----------|----------|
| **P0** | 全面瘫痪 | 所有节点失效，订阅不可用 | 5 分钟内 | 立即启用备用机场 + 联系客服 |
| **P1** | 大规模阻断 | 80% 以上节点失效 | 15 分钟内 | 批量切换备用节点 + 更换协议 |
| **P2** | 局部故障 | 单个机场或部分节点失效 | 30 分钟内 | 切换至备用机场节点 |
| **P3** | 性能下降 | 速度低于正常值的 50% | 1 小时内 | 更换节点 + 等待晚间限速缓解 |

## 6.2 标准化响应流程

### P0 全面瘫痪应急响应（10 分钟行动计划）

**第 1 分钟 — 确认与评估**

- 确认所有节点均不可用（尝试 3 个以上不同地区节点）
- 记录错误信息、当前时间、近期操作记录
- 评估是否为全局性阻断（询问群组成员状态）

**第 2-3 分钟 — 启用备用方案**

- 立即打开备用机场订阅（确保已提前配置好）
- 启用 Clash 分流规则的 fallback 组，确保自动切换
- 若无备用机场，启用手机热点或切换至手机代理

**第 4-6 分钟 — 协议与域名切换**

- 将所有 VLESS/Trojan 节点切换为 Shadowsocks + v2ray-plugin（WebSocket）协议
- 将服务器地址从直接 IP 切换为 CDN 域名（如 `*.cloudflarestunnel.com`）
- 启用域名前置（SNI 伪装）设置为 `www.microsoft.com` 或 `www.amazon.com`

**第 7-10 分钟 — 通知与记录**

- 在常用沟通群组（如 Telegram 群）告知用户当前状态
- 联系主机场客服获取最新节点和备用方案
- 将故障时间、根因（若已知）、恢复措施记录到个人知识库

### P1 大规模阻断应急响应（15 分钟行动计划）

**第 1-5 分钟 — 快速切换**

- 批量刷新订阅获取最新可用节点
- 优先选择"新上线"或"CDN 加速"标记的节点
- 启用订阅中的"智能选路"或"自动测试"功能

**第 6-10 分钟 — 协议调整**

- 批量将直连 IP 节点替换为 CDN 域名节点
- 若机场支持，切换至 IEPL/IPLC 专线套餐（通常不受 GFW 直接影响）
- 考虑升级至双机场配置（主 + 备双订阅）

**第 11-15 分钟 — 评估与决策**

- 测试各节点速度和稳定性，选择最优节点
- 调整分流规则，确保重要流量（工作、学习）走稳定节点
- 记录本次阻断的节点 IP，整理到"失效节点黑名单"

## 6.3 升级机制

### 何时升级故障等级

- **触发 P0 → P1**：单个备用机场也失效，或阻断持续超过 2 小时
- **触发 P1 → P2**：故障范围从部分机场扩大到全部常用机场
- **触发 P2 → P1**：部分节点恢复，但仍有超过 50% 节点失效

### 升级联系渠道

1. 机场 Telegram 官方群 → 客服实时响应（最快）
2. 机场官网工单系统 → 通常 1-4 小时响应
3. 备用通讯渠道（如 Signal、Discord）→ 备选联系
4. 社区论坛（Hostloc、V2EX）→ 查看是否有大规模报告

---

# 七、诊断脚本工具箱

本工具箱提供两个完整的诊断脚本，分别针对 Windows PowerShell 和 Linux/macOS Bash 环境。脚本设计为可独立运行，无需额外依赖，输出内容包含完整诊断报告。

## 7.1 Windows 完整诊断脚本（PowerShell）

```powershell
<#
.SYNOPSIS
    Clash 翻墙全链路诊断工具
.DESCRIPTION
    自动执行网络连通性、代理有效性、节点健康度、DNS 状态、订阅有效性五项诊断
    适用于 Windows 10/11，PowerShell 5.1+
.EXAMPLE
    .\Clash-Diagnose.ps1 -SubscriptionUrl "你的订阅链接"
    .\Clash-Diagnose.ps1 -SubscriptionUrl "你的订阅链接" -ProxyPort 7890
#>
param(
    [Parameter(Mandatory=$true, HelpMessage="Clash 订阅链接")]
    [string]$SubscriptionUrl,

    [Parameter(HelpMessage="Clash 代理端口，默认 7890")]
    [int]$ProxyPort = 7890,

    [Parameter(HelpMessage="节点超时毫秒，默认 3000")]
    [int]$TimeoutMs = 3000
)

$ErrorActionPreference = "SilentlyContinue"
$startTime = Get-Date

function Write-Card {
    param([string]$Title, [string]$Content, [string]$Color="White")
    Write-Host "`n┌─ $Title " -NoNewline -ForegroundColor Cyan
    Write-Host ("─" * (60 - $Title.Length)) -ForegroundColor DarkGray
    Write-Host $Content -ForegroundColor $Color
    Write-Host "└" + ("─" * 62) -ForegroundColor DarkGray
}

function Test-UrlAccess {
    param([string]$Url, [int]$TimeoutSec = 10)
    try {
        $r = Invoke-WebRequest -Uri $Url -UseBasicParsing -TimeoutSec $TimeoutSec -Head
        return @{Success=$true; Status=$r.StatusCode; Length=$r.Content.Length}
    } catch {
        return @{Success=$false; Error=$_.Exception.Message}
    }
}

Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "  Clash 全链路诊断工具 v2.0 | $(Get-Date -Format 'yyyy-MM-dd HH:mm')" -ForegroundColor Cyan
Write-Host "========================================`n" -ForegroundColor Cyan

# ============ 诊断一：订阅有效性 ============
Write-Host "[1/5] 订阅有效性检测..." -ForegroundColor Yellow
$subResult = Test-UrlAccess -Url $SubscriptionUrl -TimeoutSec 15
if ($subResult.Success) {
    Write-Card "订阅有效性" "状态码: $($subResult.Status)`n内容长度: $($subResult.Length) 字符`n结果: 订阅源正常" Green
} else {
    Write-Card "订阅有效性" "状态码: 不可达`n错误: $($subResult.Error)`n结果: 订阅失效 — 请重新获取订阅链接!" Red
}

# ============ 诊断二：本地代理端口 ============
Write-Host "[2/5] 本地代理端口检测..." -ForegroundColor Yellow
$portCheck = Get-NetTCPConnection -LocalPort $ProxyPort -ErrorAction SilentlyContinue
if ($portCheck) {
    Write-Card "代理端口 $ProxyPort" "监听地址: $($portCheck.LocalAddress):$ProxyPort`n进程 ID: $($portCheck.OwningProcess)`n结果: 端口正常监听" Green
} else {
    Write-Card "代理端口 $ProxyPort" "结果: 端口未监听 — 请确认 Clash 客户端已启动" Red
}

# ============ 诊断三：出口 IP 检测 ============
Write-Host "[3/5] 代理出口 IP 检测..." -ForegroundColor Yellow
try {
    $directIP = (Invoke-RestMethod "https://api.ip.sb/ip" -TimeoutSec 5).Trim()
    $env:HTTP_PROXY = "http://127.0.0.1:$ProxyPort"
    $env:HTTPS_PROXY = "http://127.0.0.1:$ProxyPort"
    $proxyIP = (Invoke-RestMethod "https://api.ip.sb/ip" -TimeoutSec 8).Trim()
    Remove-Item Env:HTTP_PROXY -ErrorAction SilentlyContinue
    Remove-Item Env:HTTPS_PROXY -ErrorAction SilentlyContinue

    if ($directIP -ne $proxyIP) {
        Write-Card "出口 IP 检测" "直连 IP:  $directIP`n代理 IP:  $proxyIP`n结果: 代理通道正常" Green
    } else {
        Write-Card "出口 IP 检测" "直连 IP:  $directIP`n代理 IP:  $proxyIP (相同!)`n结果: 代理未生效 — 检查分流规则和系统代理设置" Red
    }
} catch {
    Write-Card "出口 IP 检测" "错误: $_`n结果: 无法完成检测" Red
}

# ============ 诊断四：封锁网站访问 ============
Write-Host "[4/5] 封锁网站访问检测..." -ForegroundColor Yellow
$testSites = @(
    @{Name="Google"; Url="https://www.google.com"},
    @{Name="YouTube"; Url="https://www.youtube.com"},
    @{Name="ChatGPT"; Url="https://chat.openai.com"}
)

foreach ($site in $testSites) {
    try {
        $env:HTTP_PROXY = "http://127.0.0.1:$ProxyPort"
        $env:HTTPS_PROXY = "http://127.0.0.1:$ProxyPort"
        $r = Invoke-WebRequest -Uri $site.Url -UseBasicParsing -TimeoutSec 10
        Remove-Item Env:HTTP_PROXY -ErrorAction SilentlyContinue
        Remove-Item Env:HTTPS_PROXY -ErrorAction SilentlyContinue
        Write-Host "  [OK] $($site.Name)" -ForegroundColor Green
    } catch {
        Write-Host "  [FAIL] $($site.Name)" -ForegroundColor Red
    }
}

# ============ 诊断五：DNS 污染检测 ============
Write-Host "[5/5] DNS 污染检测..." -ForegroundColor Yellow
$dnsTests = @("google.com","facebook.com","twitter.com","youtube.com")
$polluted = @()
foreach ($domain in $dnsTests) {
    $resolved = [System.Net.Dns]::GetHostAddresses($domain) | Select-Object -First 1
    $ip = $resolved.ToString()
    # 国内 DNS 常见污染 IP 段
    if ($ip -match "^(1\.|2\.|42\.|103\.|106\.|117\.|119\.|120\.|121\.|123\.|180\.|202\.|203\.|218\.|219\.|220\.)") {
        $polluted += $domain
    }
}
if ($polluted.Count -eq 0) {
    Write-Card "DNS 污染检测" "测试域名: $($dnsTests -join ', ')`n结果: 未检测到 DNS 污染" Green
} else {
    Write-Card "DNS 污染检测" "污染域名: $($polluted -join ', ')`n建议: 更换系统 DNS 为 8.8.8.8 / 1.1.1.1" Yellow
}

$elapsed = (Get-Date) - $startTime
Write-Host "`n========================================" -ForegroundColor Cyan
Write-Host "  诊断完成，耗时 $($elapsed.TotalSeconds.ToString('0.0')) 秒" -ForegroundColor Cyan
Write-Host "========================================`n" -ForegroundColor Cyan
```

## 7.2 Linux/macOS 快速诊断脚本（Bash）

```bash
#!/bin/bash
#===============================================================================
# Clash 快速诊断工具 (Bash)
# 适用: Linux (Ubuntu/Debian/CentOS/macOS)
# 依赖: curl, grep, awk, sed
#===============================================================================

set -euo pipefail

PROXY_PORT="${PROXY_PORT:-7890}"
PROXY_URL="http://127.0.0.1:${PROXY_PORT}"
SUBSCRIPTION_URL="${1:-}"

RED='\033[0;31m'; GREEN='\033[0;32m'; YELLOW='\033[1;33m'
CYAN='\033[0;36m'; NC='\033[0m'

print_header() {
    echo -e "\n${CYAN}==== $1 ====${NC}"
}

# 检测代理进程
check_proxy_process() {
    print_header "代理进程检测"
    if command -v lsof >/dev/null 2>&1; then
        PID=$(lsof -ti:${PROXY_PORT} 2>/dev/null || echo "")
    elif command -v ss >/dev/null 2>&1; then
        PID=$(ss -tlnp 2>/dev/null | grep ":${PROXY_PORT}" | grep -oP 'pid=\K\d+' || echo "")
    fi
    
    if [ -n "$PID" ]; then
        PROCNAME=$(ps -p "$PID" -o comm= 2>/dev/null || echo "Unknown")
        echo -e "${GREEN}[OK]${NC} 端口 ${PROXY_PORT} 正在监听"
        echo -e "     PID: $PID | 进程: $PROCNAME"
    else
        echo -e "${RED}[FAIL]${NC} 端口 ${PROXY_PORT} 未监听 — Clash 未运行"
    fi
}

# 检测出口 IP
check_exit_ip() {
    print_header "出口 IP 检测"
    
    DIRECT_IP=$(curl -s --connect-timeout 5 https://api.ip.sb/ip 2>/dev/null | tr -d '\n ' || echo "ERROR")
    PROXY_IP=$(curl -s --connect-timeout 8 --proxy "${PROXY_URL}" https://api.ip.sb/ip 2>/dev/null | tr -d '\n ' || echo "ERROR")
    
    echo -e "直连 IP: ${DIRECT_IP}"
    echo -e "代理 IP: ${PROXY_IP}"
    
    if [[ "$DIRECT_IP" == "$PROXY_IP" ]]; then
        echo -e "${RED}[FAIL]${NC} 代理未生效 — IP 地址相同"
    elif [[ "$PROXY_IP" == "ERROR" ]]; then
        echo -e "${RED}[FAIL]${NC} 无法通过代理获取 IP"
    else
        echo -e "${GREEN}[OK]${NC} 代理通道正常"
    fi
}

# 封锁网站访问测试
check_blocked_sites() {
    print_header "封锁网站访问测试"
    
    sites=(
        "Google|https://www.google.com"
        "YouTube|https://www.youtube.com"
        "GitHub|https://github.com"
    )
    
    for entry in "${sites[@]}"; do
        IFS='|' read -r name url <<< "$entry"
        STATUS=$(curl -o /dev/null -s -w "%{http_code}" --connect-timeout 5 \
            --proxy "${PROXY_URL}" -L "$url" 2>/dev/null || echo "000")
        if [[ "$STATUS" =~ ^2|3 ]]; then
            echo -e "${GREEN}[OK]${NC} $name (HTTP $STATUS)"
        else
            echo -e "${RED}[FAIL]${NC} $name (HTTP $STATUS)"
        fi
    done
}

# DNS 污染检测
check_dns_pollution() {
    print_header "DNS 污染检测"
    
    polluted=0
    for domain in google.com facebook.com youtube.com twitter.com; do
        RESOLVED=$(timeout 3 nslookup "$domain" 2>/dev/null | grep "Address:" | tail -1 | awk '{print $2}')
        if echo "$RESOLVED" | grep -qE "^(1\.|2\.|42\.|103\.|106\.|117\.|119\.)"; then
            echo -e "${YELLOW}[污染]${NC} $domain -> $RESOLVED"
            ((polluted++))
        else
            echo -e "${GREEN}[正常]${NC} $domain -> $RESOLVED"
        fi
    done
    
    if [ $polluted -eq 0 ]; then
        echo -e "\n${GREEN}未检测到 DNS 污染${NC}"
    else
        echo -e "\n${YELLOW}检测到 ${polluted} 个域名存在 DNS 污染${NC}"
        echo "建议: sudo networksetup -setdnsservers Wi-Fi 8.8.8.8 1.1.1.1 (macOS)"
        echo "建议: echo 'nameserver 8.8.8.8' | sudo tee /etc/resolv.conf (Linux)"
    fi
}

# 速度基准测试
speed_benchmark() {
    print_header "速度基准测试"
    
    echo "测试服务器: 新加坡 CDN"
    SPEED=$(curl -o /dev/null -s -w "%{speed_download}" \
        --proxy "${PROXY_URL}" \
        -L "https://speed.cloudflare.com/__down?bytes=10000000" \
        --connect-timeout 10 2>/dev/null || echo "0")
    
    if [[ "$SPEED" != "0" ]]; then
        SPEED_KB=$((SPEED / 1024))
        echo -e "下载速度: ${GREEN}${SPEED_KB} KB/s ($(echo "scale=1; $SPEED_KB/1024" | bc) MB/s)${NC}"
        if [ $SPEED_KB -gt 1024 ]; then
            echo -e "${GREEN}速度正常${NC}"
        elif [ $SPEED_KB -gt 100 ]; then
            echo -e "${YELLOW}速度较慢${NC}"
        else
            echo -e "${RED}速度异常慢${NC}"
        fi
    else
        echo -e "${RED}速度测试失败${NC}"
    fi
}

# 主程序
main() {
    echo -e "\n========================================"
    echo -e "  Clash 快速诊断工具 | $(date '+%Y-%m-%d %H:%M')"
    echo -e "========================================"
    
    check_proxy_process
    check_exit_ip
    check_blocked_sites
    check_dns_pollution
    speed_benchmark
    
    echo -e "\n========================================\n"
}

main "$@"
```

---

# 八、推广链接汇总

## 🛒 ClashVIP

> **[ClashVIP 官网注册](https://clashvip.net)**

| 套餐 | 流量 | 价格 | 特色 |
|------|------|------|------|
| 基础版 | 500GB/月 | ¥29/月 | 入门首选 |
| 专业版 | 2TB/月 | ¥99/季 | 主流推荐 |
| 旗舰专线 | 5TB/月 | ¥399/年 | IEPL 专线 |

## 🔗 Nav.CLASHVIP.NET

- **[nav.clashvip.net](https://nav.clashvip.net)** — ClashVIP 备用入口导航页

## 🌐 ClashHub

- **[clashhub.net](https://clashhub.net)** — 优质翻墙节点导航
- **[bbs.clashhub.net](https://bbs.clashhub.net)** — 社区论坛与经验交流

## 💻 Clash-for-Windows

- **[clash-for-windows.net](https://clash-for-windows.net)** — Clash for Windows 官方下载站

## 🚀 VPSVIP

- **[vpsvip.net](https://vpsvip.net)** — VPSVIP 专线机场

## 🌍 VPSVIP 专线推荐

> **Netflex/Di**

| 套餐 | 流量 | 价格 | 特色 |
|------|------|------|------|
| 月付 | 100-200GB/月 | ¥19/月起 | 稳定低延迟 |
| 年付 | 1.5TB/月 | ¥199/年 | 性价比之王 |
| 旗舰 | 5TB/月 | ¥399/年 | 重度用户首选 |

> **更多 VPSVIP 专线节点推荐**

- Netflex/Di — 流媒体解锁专家，支持 Netflix/HBO/Disney+ 全家桶
- YOUTUBE PREMIUM — YouTube Premium 合租平台
- Hulutv — Hulu 美区解锁专线
- Amazon Prime — 亚马逊 Prime 专线
- HBO Max — HBO Max 全平台专线

---

## ⚙️ 支持的客户端

| 客户端 | 平台 | 下载渠道 |
|--------|------|----------|
| Clash for Windows | Windows | [clash-for-windows.net](https://clash-for-windows.net) |
| ClashX / ClashX Pro | macOS | App Store |
| Clash for Android | Android | Google Play / F-Droid |
| Stash | iOS | App Store |
| OpenClash | OpenWrt | OpenClash/PassWall |

---

## ⚠️ 免责声明

1. 本指南仅供技术学习与研究使用
2. 使用前请自行评估当地法律法规
3. 本页面不对任何节点服务的可用性负责
4. 节点价格和套餐可能随时调整，请以官网为准

---

**最后更新：2026-08-20**

---

*本指南由技术社区维护，持续更新中。如有疏漏或建议，欢迎提交 Issue 或 Pull Request。*
