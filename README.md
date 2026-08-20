# 🩺 Clash 故障排查完全手册 — 从症状到根治，从诊断到预防

> 无论是连接超时、速度骤降、解锁失败还是规则失效，本手册提供系统化的诊断思路、逐级排查流程、完整解决方案库和预防措施。面向所有 Clash 用户，无论你是新手还是老手，都能在这里找到故障的根本原因和最优解。

---

## 📋 目录

- [诊断前置准备](#诊断前置准备)
- [常见故障分类速查](#常见故障分类速查)
- [故障诊断流程图](#故障诊断流程图)
- [故障详解与解决方案](#故障详解与解决方案)
- [PowerShell 诊断脚本](#powershell-诊断脚本)
- [预防措施体系](#预防措施体系)
- [日志分析方法](#日志分析方法)
- [常见问题 FAQ](#常见问题-faq)
- [推荐资源](#推荐资源)

---

## 🔧 诊断前置准备

### 必备诊断工具

在开始排查前，请确保安装了以下工具：

```powershell
# 诊断工具清单 - 全部安装后开始排查
$diagnosticTools = @(
    @{ Name = "Clash for Windows"; Check = "C:\Program Files\Clash for Windows\Clash for Windows.exe" },
    @{ Name = "Clash Verge"; Check = "$env:LOCALAPPDATA\Clash Verge\Clash Verge.exe" },
    @{ Name = "Node.js (v18+)"; Check = "node --version" },
    @{ Name = "Python 3.8+"; Check = "python --version" },
    @{ Name = "curl"; Check = "curl --version" },
    @{ Name = "PowerShell 5.1+"; Check = "$PSVersionTable.PSVersion" }
)

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "   Clash 诊断工具就绪检查" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

foreach ($tool in $diagnosticTools) {
    $status = "❓"
    $color = "Yellow"
    
    if ($tool.Check -like "*.*") {
        $exists = Test-Path $tool.Check
        if ($exists) { $status = "✅"; $color = "Green" }
        else { $status = "❌"; $color = "Red" }
    } else {
        try {
            $result = Invoke-Expression "$($tool.Check) 2>&1"
            if ($result) { $status = "✅"; $color = "Green" }
        } catch {
            $status = "❌"; $color = "Red"
        }
    }
    
    Write-Host "$($status) $($tool.Name)" -ForegroundColor $color
}
```

### 获取关键信息

```powershell
# 系统与网络基本信息收集
Write-Host "`n========== 系统信息 ==========" -ForegroundColor Yellow
Write-Host "OS: $([System.Environment]::OSVersion.VersionString)"
Write-Host "PowerShell: $($PSVersionTable.PSVersion)"
Write-Host "当前时间: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')"

Write-Host "`n========== 网络信息 ==========" -ForegroundColor Yellow
Write-Host "DNS服务器: $(Get-DnsClientServerAddress | Where-Object { $_.AddressFamily -eq 2 }).ServerAddresses"
Write-Host "网关: $(Get-NetRoute -DestinationPrefix '0.0.0.0/0' | Select-Object -First 1 -ExpandProperty NextHop)"

Write-Host "`n========== Clash 进程 ==========" -ForegroundColor Yellow
Get-Process | Where-Object { $_.Name -like "*Clash*" -or $_.Name -like "*clash*" } | Format-Table Name, Id, CPU, WorkingSet -AutoSize

Write-Host "`n========== Clash 端口监听 ==========" -ForegroundColor Yellow
Get-NetTCPConnection | Where-Object { $_.LocalPort -in @(7890, 7891, 9090, 8443) } | Format-Table LocalAddress, LocalPort, State -AutoSize
```

---

## 🚨 常见故障分类速查

| 故障类型 | 典型表现 | 紧急程度 | 平均解决时间 |
|---------|---------|---------|------------|
| 🔴 完全无法连接 | 代理端口无响应 | 🔥 高 | 5-15 分钟 |
| 🟠 速度极慢 | 网页加载 >10秒 | 🟡 中 | 10-30 分钟 |
| 🟡 特定网站打不开 | 部分正常部分异常 | 🟡 中 | 15-45 分钟 |
| 🟢 解锁失败 | Netflix 等无法解锁 | 🔵 低 | 5-20 分钟 |
| 🟣 规则不生效 | 分流规则走错线路 | 🔵 低 | 5-10 分钟 |
| 🔵 订阅更新失败 | 无法拉取节点列表 | 🟡 中 | 10-30 分钟 |
| ⚫ 频繁断线 | 每隔几分钟就断开 | 🟡 中 | 20-60 分钟 |

---

## 🗺️ 故障诊断流程图

```
发现故障
    ↓
┌─────────────────────────────────────┐
│ 步骤1：确认故障范围                  │
│  • 所有网站都慢？→ 网络层问题        │
│  • 特定网站慢？→ 规则/DNS问题        │
│  • 完全无法访问？→ 连接问题          │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ 步骤2：检查 Clash 状态               │
│  • 进程是否运行？                   │
│  • 系统代理是否开启？                │
│  • API 是否正常响应？                │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ 步骤3：分段排查                      │
│  • 直连访问目标 → 网络是否通         │
│  • 通过代理访问 → 节点是否可用        │
│  • 指定规则访问 → 规则是否正确       │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ 步骤4：逐一验证                      │
│  • 更换节点测试                      │
│  • 更换 DNS 测试                     │
│  • 重置规则测试                      │
│  • 更新订阅测试                      │
└─────────────────────────────────────┘
    ↓
找到根因 → 应用解决方案 → 记录并预防
```

---

## 🔍 故障详解与解决方案

### 故障 1：完全无法连接（最紧急）

#### 症状表现
- Clash 已开启但浏览器无法访问任何网站
- 代理端口 `127.0.0.1:7890` 无响应
- `curl -x http://127.0.0.1:7890 https://www.google.com` 超时

#### 排查路径

**Step 1：检查 Clash 进程**
```powershell
# 方法A：进程检查
tasklist | findstr /i "Clash clash"

# 方法B：端口检查
netstat -an | findstr "7890 7891"

# 方法C：服务检查（Clash Verge/Mihomo）
Get-Process | Where-Object { $_.Name -match "clash" } | Select-Object Name, Id, Responding
```

**Step 2：检查 Clash 日志**
```powershell
# Clash for Windows 日志路径
$logPaths = @(
    "$env:APPDATA\Clash for Windows\logs",
    "$env:LOCALAPPDATA\Clash for Windows\logs",
    "$env:APPDATA\Clash Verge\logs"
)

foreach ($path in $logPaths) {
    if (Test-Path $path) {
        Write-Host "📁 日志目录: $path" -ForegroundColor Cyan
        Get-ChildItem $path -File | Sort-Object LastWriteTime -Descending | Select-Object -First 5 | ForEach-Object {
            Write-Host "  → $($_.Name) ($(Get-Date $_.LastWriteTime -Format 'HH:mm:ss'))"
            Get-Content $_.FullName -Tail 30
        }
    }
}
```

**Step 3：常见原因及解决方案**

| 原因 | 诊断方法 | 解决方案 |
|------|---------|---------|
| Clash 配置文件语法错误 | 查看日志中 "yaml" 或 "parse" 关键词 | 用 [YAML 验证器](https://www.yamllint.com/) 检查并修复 |
| 订阅 URL 已过期/失效 | 重新访问订阅链接 | 到机场官网重新获取订阅 |
| 端口被占用 | `netstat -ano \| findstr 7890` | 关闭占用进程或改端口 |
| 系统代理未开启 | 检查系统代理设置 | 开启系统代理（设置为 127.0.0.1:7890）|
| TUN 模式冲突 | 检查虚拟网卡 | 关闭 TUN 或更新 TUN 驱动 |
| 防火墙拦截 | 暂时关闭防火墙测试 | 添加 Clash 到防火墙白名单 |

**Step 4：快速修复脚本**

```powershell
# fix-no-connection.ps1 - 修复完全无法连接
Write-Host "========== 修复: 完全无法连接 ==========" -ForegroundColor Yellow

# 1. 重启 Clash 进程
Write-Host "[1/5] 重启 Clash 进程..." -NoNewline
try {
    Stop-Process -Name "Clash for Windows" -Force -ErrorAction SilentlyContinue
    Stop-Process -Name "Clash Verge" -Force -ErrorAction SilentlyContinue
    Start-Sleep -Seconds 2
    Write-Host "已停止" -ForegroundColor Green
    
    # 重新启动（根据实际路径）
    $cfw = "C:\Program Files\Clash for Windows\Clash for Windows.exe"
    if (Test-Path $cfw) { Start-Process $cfw }
    
    Write-Host "已启动" -ForegroundColor Green
} catch {
    Write-Host "失败: $_" -ForegroundColor Red
}

# 2. 重置系统代理
Write-Host "[2/5] 重置系统代理设置..." -NoNewline
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -Name "ProxyEnable" -Value 0
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings" -Name "AutoConfigURL" -Value ""
Write-Host "完成" -ForegroundColor Green

# 3. 刷新网络配置
Write-Host "[3/5] 刷新 DNS 缓存..." -NoNewline
ipconfig /flushdns | Out-Null
Write-Host "完成" -ForegroundColor Green

# 4. 测试连通性
Write-Host "[4/5] 测试本地代理..." -NoNewline
try {
    $test = Invoke-WebRequest -Uri "http://127.0.0.1:9090/proxies" -TimeoutSec 5 -UseBasicParsing
    Write-Host "✅ Clash API 正常" -ForegroundColor Green
} catch {
    Write-Host "⚠️ API 无响应，请手动检查 Clash 是否运行" -ForegroundColor Red
}

# 5. 提示更新订阅
Write-Host "[5/5] 建议：重新获取订阅链接 → https://clashvip.net" -ForegroundColor Cyan

Write-Host "`n修复完成，如仍无法连接请查看 Clash 日志或提交 Issue" -ForegroundColor Green
```

---

### 故障 2：速度极慢（最常见）

#### 症状表现
- 网页加载时间 >10 秒
- YouTube 视频 1080P 也频繁缓冲
- 下载速度远低于套餐承诺带宽

#### 诊断流程

```powershell
# speed-diagnosis.ps1 - 速度慢诊断脚本

param(
    [string]$ProxyHost = "127.0.0.1",
    [int]$ProxyPort = 7890,
    [int]$TestDurationSec = 30
)

Write-Host "========== 速度慢诊断工具 ==========" -ForegroundColor Cyan

# Step 1: 测试本地到代理的网络延迟
Write-Host "`n[Step 1] 延迟测试..." -ForegroundColor Yellow
$latencies = @()
foreach ($i in 1..5) {
    try {
        $sw = [System.Diagnostics.Stopwatch]::StartNew()
        Invoke-WebRequest -Uri "http://www.gstatic.com/generate_204" `
            -Proxy "http://${ProxyHost}:${ProxyPort}" -TimeoutSec 10 -UseBasicParsing | Out-Null
        $lat = $sw.ElapsedMilliseconds
        $latencies += $lat
        Write-Host "  尝试 $i : $lat ms" -ForegroundColor Gray
    } catch {
        Write-Host "  尝试 $i : 超时" -ForegroundColor Red
    }
}

if ($latencies.Count -gt 0) {
    $avgLat = ($latencies | Measure-Object -Average).Average
    $color = if ($avgLat -lt 100) { "Green" } elseif ($avgLat -lt 300) { "Yellow" } else { "Red" }
    Write-Host "平均延迟: $avgLat ms" -ForegroundColor $color
    
    if ($avgLat -gt 300) {
        Write-Host "⚠️ 延迟过高！建议更换节点或检查本地网络" -ForegroundColor Red
    }
}

# Step 2: 带宽测试
Write-Host "`n[Step 2] 带宽测试..." -ForegroundColor Yellow
$testUrls = @(
    @{ Name = "Cloudflare"; Url = "https://speed.cloudflare.com/__down?bytes=10000000" },
    @{ Name = "OVH"; Url = "https://proof.ovh.net/files/10Mb.dat" }
)

foreach ($t in $testUrls) {
    try {
        $sw = [System.Diagnostics.Stopwatch]::StartNew()
        $resp = Invoke-WebRequest -Uri $t.Url -Proxy "http://${ProxyHost}:${ProxyPort}" `
            -TimeoutSec $TestDurationSec -UseBasicParsing
        $elapsed = $sw.ElapsedMilliseconds / 1000
        $sizeMB = $resp.Content.Length / 1MB
        $speedMbps = [math]::Round(($sizeMB * 8) / $elapsed, 2)
        
        $color = if ($speedMbps -gt 50) { "Green" } elseif ($speedMbps -gt 20) { "Yellow" } else { "Red" }
        Write-Host "  $($t.Name): $speedMbps Mbps" -ForegroundColor $color
        
        if ($speedMbps -lt 10) {
            Write-Host "    ⚠️ 速度过低！检查节点质量或联系机场客服" -ForegroundColor Red
        }
    } catch {
        Write-Host "  $($t.Name): 测试失败 - $($_.Exception.Message)" -ForegroundColor Red
    }
}

# Step 3: 检查当前节点
Write-Host "`n[Step 3] 当前节点信息..." -ForegroundColor Yellow
try {
    $api = Invoke-RestMethod -Uri "http://${ProxyHost}:9090/proxies/GLOBAL" -TimeoutSec 5
    Write-Host "  当前节点: $($api.now)" -ForegroundColor Cyan
    Write-Host "  所有节点数: $($api.all.Count)" -ForegroundColor Gray
} catch {
    Write-Host "  无法获取节点信息（Clash API 可能未开启）" -ForegroundColor Red
}

# Step 4: 丢包率检测
Write-Host "`n[Step 4] 连接稳定性..." -ForegroundColor Yellow
$success = 0
$total = 10
for ($i = 1; $i -le $total; $i++) {
    try {
        Invoke-WebRequest -Uri "http://www.gstatic.com/generate_204" `
            -Proxy "http://${ProxyHost}:${ProxyPort}" -TimeoutSec 5 -UseBasicParsing | Out-Null
        $success++
        Write-Host "." -NoNewline -ForegroundColor Green
    } catch {
        Write-Host "x" -NoNewline -ForegroundColor Red
    }
}
Write-Host ""
$lossRate = [math]::Round((1 - $success / $total) * 100, 1)
Write-Host "丢包率: $lossRate% （成功率: $([math]::Round($success/$total*100))%）" -ForegroundColor $(if($lossRate -gt 5){"Red"}elseif($lossRate -gt 1){"Yellow"}else{"Green"})

Write-Host "`n========== 诊断结果总结 ==========" -ForegroundColor Cyan
Write-Host "建议操作:"
Write-Host "  1. 更换节点（到 Clash 客户端手动选择一个低延迟节点）"
Write-Host "  2. 更新订阅（旧的节点列表可能已劣化）"
Write-Host "  3. 检查运营商限速（晚高峰期间运营商可能限速）"
Write-Host "  4. 切换协议（VMess→Trojan 或 Hysteria2）"
Write-Host "  5. 前往 https://nav.clashvip.net 寻找优质节点"
```

#### 速度慢原因速查表

| 原因 | 诊断信号 | 解决方案 |
|------|---------|---------|
| 节点劣化/过载 | 晚高峰尤其慢 | 切换到其他节点 |
| 订阅过期 | 节点列表为空或很少 | 重新获取订阅 |
| DNS 污染 | 特定网站打不开 | 修改 DNS 为 8.8.8.8 / 1.1.1.1 |
| MTU 问题 | 网页加载一半卡住 | 设置 `mtu: 1500` 或 `1280` |
| 加密算法过重 | CPU 占用高但速度慢 | 改用 aes-128-gcm 替代 chacha20 |
| 运营商 QoS | 特定端口被限速 | 切换到 TLS 443 端口 |
| TUN 模式性能差 | CPU 高占用 | 关闭 TUN 或换用 fake-ip |

---

### 故障 3：特定网站打不开

#### 诊断思路

特定网站打不开（其他正常），通常由以下原因导致：

```
特定网站异常
    ↓
┌─ DNS 污染 ──────────────────────────────┐
│  诊断：直接 ping 目标域名                │
│  解法：Clash DNS 配置改为 8.8.8.8        │
└─────────────────────────────────────────┘
    ↓
┌─ 规则误匹配 ─────────────────────────────┐
│  诊断：检查规则中是否有该域名 DIRECT    │
│  解法：添加规则或检查规则顺序           │
└─────────────────────────────────────────┘
    ↓
┌─ 节点不支持该地区 ───────────────────────┐
│  诊断：换用目标地区节点测试              │
│  解法：使用指定地区节点                 │
└─────────────────────────────────────────┘
    ↓
┌─ TLS/SNI 指纹 ───────────────────────────┐
│  诊断：浏览器开发者工具查看 TLS 错误    │
│  解法：开启 TLS 伪装或使用 WARP 插件    │
└─────────────────────────────────────────┘
```

#### 诊断脚本

```powershell
# website-issue.ps1 - 特定网站打不开诊断
param(
    [string]$TargetUrl = "",
    [string]$ProxyHost = "127.0.0.1",
    [int]$ProxyPort = 7890
)

if (-not $TargetUrl) {
    $TargetUrl = Read-Host "请输入打不开的网站URL (例如: https://chat.openai.com)"
}

Write-Host "========== 网站访问诊断: $TargetUrl ==========" -ForegroundColor Cyan

# 1. 提取域名
$domain = ([System.Uri]$TargetUrl).Host
Write-Host "`n域名: $domain"

# 2. 直连测试（绕过代理）
Write-Host "`n[1] 直连测试（绕过代理）..." -ForegroundColor Yellow
try {
    $sw = [System.Diagnostics.Stopwatch]::StartNew()
    $resp = Invoke-WebRequest -Uri $TargetUrl -TimeoutSec 10 -UseBasicParsing
    $elapsed = $sw.ElapsedMilliseconds
    Write-Host "  ✅ 直连成功 ($elapsed ms)" -ForegroundColor Green
    Write-Host "  状态码: $($resp.StatusCode)"
} catch {
    Write-Host "  ❌ 直连失败: $($_.Exception.Message)" -ForegroundColor Red
}

# 3. 代理测试
Write-Host "`n[2] 代理访问测试..." -ForegroundColor Yellow
try {
    $sw = [System.Diagnostics.Stopwatch]::StartNew()
    $resp = Invoke-WebRequest -Uri $TargetUrl -Proxy "http://${ProxyHost}:${ProxyPort}" -TimeoutSec 15 -UseBasicParsing
    $elapsed = $sw.ElapsedMilliseconds
    Write-Host "  ✅ 代理成功 ($elapsed ms)" -ForegroundColor Green
    Write-Host "  状态码: $($resp.StatusCode)"
} catch {
    Write-Host "  ❌ 代理失败: $($_.Exception.Message)" -ForegroundColor Red
    
    # 4. 诊断 SNI 指纹问题
    Write-Host "`n[3] SNI 指纹检测..." -ForegroundColor Yellow
    Write-Host "  提示: 如果报错包含 'ssl' 或 'handshake'，可能是 SNI 问题"
    Write-Host "  解决方案: 在 Clash 配置中添加 sni 字段或使用 TLS 伪装"
}

# 5. DNS 解析测试
Write-Host "`n[4] DNS 解析测试..." -ForegroundColor Yellow
try {
    $dnsResult = Resolve-DnsName -Name $domain -Type A -Server 8.8.8.8 -TimeoutSec 5
    Write-Host "  DNS 解析结果: $($dnsResult.IPAddress -join ', ')" -ForegroundColor Cyan
    
    # 检查是否被 DNS 污染（解析到奇怪 IP）
    $suspicious = $dnsResult | Where-Object { $_.IPAddress -match "^0\." -or $_.IPAddress -eq "" }
    if ($suspicious) {
        Write-Host "  ⚠️ 疑似 DNS 污染，建议在 Clash 中使用 fake-ip 模式" -ForegroundColor Red
    }
} catch {
    Write-Host "  ❌ DNS 解析失败: $($_.Exception.Message)" -ForegroundColor Red
}

# 6. 检查规则匹配
Write-Host "`n[5] 规则检查..." -ForegroundColor Yellow
Write-Host "  建议检查 Clash 配置中的 rules 字段"
Write-Host "  是否存在 DOMAIN/$domain 或 DOMAIN-SUFFIX/$domain 规则"
Write-Host "  如果有 DIRECT 规则，该域名会被直连 → 异常"
Write-Host "  正确做法: 规则应为 'DOMAIN-SUFFIX,$domain,🔧 手动选择'"

Write-Host "`n========== 建议操作 ==========" -ForegroundColor Cyan
Write-Host "  1. 打开 Clash 客户端 → 代理 → 手动选择 → 尝试其他节点"
Write-Host "  2. 检查该网站的地区限制，确认节点是否在允许地区"
Write-Host "  3. 尝试开启/关闭 TLS 伪装"
Write-Host "  4. 清除浏览器缓存和 DNS 缓存: ipconfig /flushdns"
```

---

### 故障 4：Netflix 等流媒体解锁失败

#### 解锁失败原因分析

| 症状 | 根本原因 | 解决方案 |
|------|---------|---------|
| Netflix 显示"在您所在地区不可用" | 节点 IP 被 Netflix 识别 | 切换到解锁节点 |
| Disney+ 播放报错 | 节点不在 Disney+ 支持地区 | 使用对应地区节点 |
| YouTube Premium 无法登录 | DNS 污染或 IP 问题 | 清除 Cookie 或换节点 |
| HBO Max 加载缓慢 | 节点带宽不足 | 更换高速节点 |
| 解锁不稳定，一会有一会没有 | 共享节点 IP 被标记 | 使用独享/专线节点 |

#### 解锁测试脚本

```python
#!/usr/bin/env python3
"""
stream_unlock_test.py - 流媒体解锁检测工具
pip install httpx playwright
"""

import asyncio
import json
from dataclasses import dataclass
from typing import Dict, Optional

try:
    import httpx
except ImportError:
    print("请先安装: pip install httpx")
    exit(1)


@dataclass
class UnlockResult:
    platform: str
    status: str  # "解锁" / "未解锁" / "部分解锁" / "未知"
    detail: str = ""
    region: str = ""


class StreamUnlockTester:
    """流媒体解锁检测器"""
    
    # 测试 URL（各平台 API 端点）
    TEST_ENDPOINTS = {
        "Netflix": {
            "check": "https://www.netflix.com/title/60021922",
            "headers": {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"},
            "keywords": ["NETFLIX", "nflxVideo", "seriesTitle", "playError"],
        },
        "Disney+": {
            "check": "https://api.disney.com/disney+",
            "headers": {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"},
            "keywords": ["disney+", "disneyplus", "contentAvailable"],
        },
        "YouTube": {
            "check": "https://www.youtube.com/premium",
            "headers": {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"},
            "keywords": ["YouTube Premium", "premium", "ytInitialData"],
        },
        "TikTok": {
            "check": "https://www.tiktok.com/",
            "headers": {"User-Agent": "Mozilla/5.0 (iPhone; CPU iPhone OS 14_0 like Mac OS X)"},
            "keywords": ["tiktok", "sessionId"],
        },
        "Amazon Prime": {
            "check": "https://www.primevideo.com/",
            "headers": {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"},
            "keywords": ["primevideo", "amazon", "av-na"],
        }
    }
    
    def __init__(self, proxy_url: str = "http://127.0.0.1:7890", timeout: int = 15):
        self.proxy_url = proxy_url
        self.timeout = timeout
    
    async def test_platform(self, platform: str, config: dict) -> UnlockResult:
        """测试单个平台的解锁状态"""
        result = UnlockResult(platform=platform, status="未知")
        
        try:
            async with httpx.AsyncClient(
                proxies=self.proxy_url,
                timeout=self.timeout,
                follow_redirects=True,
                headers=config["headers"]
            ) as client:
                response = await client.get(config["check"])
                content = response.text
                status_code = response.status_code
                
                # 分析响应内容
                unlocked_kw = any(kw.lower() in content.lower() for kw in config["keywords"])
                blocked_kw = any(kw in content.lower() for kw in ["not available", "unavailable", "playError", "geo-block"])
                
                if status_code == 200 and unlocked_kw and not blocked_kw:
                    result.status = "解锁"
                    result.detail = f"HTTP {status_code}, 内容包含平台标识"
                elif status_code == 403:
                    result.status = "未解锁"
                    result.detail = "HTTP 403 - IP 被限制"
                elif status_code == 200 and blocked_kw:
                    result.status = "未解锁"
                    result.detail = "内容显示地区限制"
                else:
                    result.status = "部分解锁"
                    result.detail = f"HTTP {status_code}, 需进一步验证"
                    
        except httpx.TimeoutException:
            result.status = "未解锁"
            result.detail = "连接超时"
        except Exception as e:
            result.status = "未知"
            result.detail = str(e)
        
        return result
    
    async def run_all(self) -> Dict[str, UnlockResult]:
        """测试所有平台"""
        tasks = [
            self.test_platform(name, cfg) 
            for name, cfg in self.TEST_ENDPOINTS.items()
        ]
        results = await asyncio.gather(*tasks)
        return {r.platform: r for r in results}


async def main():
    import argparse
    parser = argparse.ArgumentParser(description="流媒体解锁检测")
    parser.add_argument("--proxy", default="http://127.0.0.1:7890", help="代理地址")
    args = parser.parse_args()
    
    print("=" * 50)
    print("   流媒体解锁检测工具")
    print("=" * 50)
    print(f"代理: {args.proxy}\n")
    
    tester = StreamUnlockTester(proxy_url=args.proxy)
    results = await tester.run_all()
    
    for platform, result in results.items():
        icon = {"解锁": "✅", "未解锁": "❌", "部分解锁": "⚠️", "未知": "❓"}.get(result.status, "❓")
        print(f"{icon} {platform:15} | {result.status:10} | {result.detail}")
    
    unlocked_count = sum(1 for r in results.values() if r.status == "解锁")
    total = len(results)
    print(f"\n解锁率: {unlocked_count}/{total}")
    
    if unlocked_count == 0:
        print("\n💡 建议: 更换到支持流媒体的优质节点")
        print("   推荐: https://clashvip.net - 全节点解锁 Netflix/Disney+")
        print("   导航: https://nav.clashvip.net - 查看各平台解锁状态")


if __name__ == "__main__":
    asyncio.run(main())
```

---

### 故障 5：规则不生效

#### 问题场景

明明配置了 `DOMAIN-SUFFIX,netflix.com,香港节点`，但 netflix.com 还是走了其他节点。

#### 排查步骤

**1. 检查规则语法**
```yaml
# 常见错误
- DOMAIN-SUFFIX,netflix.com,香港节点  # ✅ 正确
- DOMAIN,netflix.com,香港节点          # ❌ netflix.com 是域名而非完整域名
- DOMAIN-SUFFIX,NETFLIX.COM,香港节点   # ❌ 大小写敏感
- DOMAIN-KEYWORD,netflix,DIRECT        # ⚠️ 这会让所有含 netflix 的都直连
```

**2. 检查规则顺序**
```yaml
# 规则是按顺序匹配的，第一个命中的规则生效！
rules:
  - DOMAIN-SUFFIX,netflix.com,香港节点  # ← 放前面 ✅
  - MATCH,自动选择                       # ← 兜底放最后 ✅
  
  # 如果顺序反了：
  - MATCH,自动选择                       # ← 这条先命中，后面的永远不执行 ❌
  - DOMAIN-SUFFIX,netflix.com,香港节点
```

**3. 验证规则生效**
```powershell
# 检查 Clash 日志中的规则匹配
# 在 Clash 客户端设置中开启 "Rule Debug" 日志
# 然后在 Clash 日志中搜索域名，观察匹配的规则名
```

---

### 故障 6：订阅更新失败

#### 常见原因与解决方案

| 错误类型 | 原因 | 解决 |
|---------|------|------|
| 400 Bad Request | URL 格式错误 | 检查订阅链接是否有空格或特殊字符 |
| 401 Unauthorized | 订阅需要认证 | 添加 Authorization 头 |
| 403 Forbidden | 订阅地址被墙 | 使用代理访问机场官网重新获取 |
| 404 Not Found | 订阅已失效 | 登录机场官网重新生成订阅 |
| 5xx Server Error | 机场服务器问题 | 等待或联系机场客服 |

#### 订阅诊断脚本

```powershell
# subscription-diagnosis.ps1
param(
    [string]$SubscriptionUrl = ""
)

if (-not $SubscriptionUrl) {
    Write-Host "请在 Clash 客户端复制订阅链接，粘贴到下方" -ForegroundColor Yellow
    Write-Host "(或直接按回车使用当前配置中的订阅)" -ForegroundColor Gray
    $SubscriptionUrl = Read-Host "订阅链接"
}

Write-Host "========== 订阅诊断 ==========" -ForegroundColor Cyan
Write-Host "订阅URL: $SubscriptionUrl"

# URL 有效性检查
Write-Host "`n[1] URL 格式检查..." -ForegroundColor Yellow
if ([System.Uri]::TryCreate($SubscriptionUrl, [System.UriKind]::Absolute, $null)) {
    Write-Host "  ✅ URL 格式正确" -ForegroundColor Green
} else {
    Write-Host "  ❌ URL 格式错误，请检查是否有特殊字符" -ForegroundColor Red
}

# 订阅内容获取测试
Write-Host "`n[2] 订阅内容获取..." -ForegroundColor Yellow
try {
    $headers = @{
        "User-Agent" = "ClashForWindows/0.20.39"
    }
    $resp = Invoke-WebRequest -Uri $SubscriptionUrl -Headers $headers -TimeoutSec 20 -UseBasicParsing
    
    Write-Host "  状态码: $($resp.StatusCode)"
    Write-Host "  内容长度: $($resp.Content.Length) 字节"
    Write-Host "  内容类型: $($resp.Headers['Content-Type'])"
    
    if ($resp.Content.Length -gt 100) {
        # 判断是否是 base64 内容
        if ($resp.Content -match "^[A-Za-z0-9+/=]{100,}={0,2}$") {
            Write-Host "  ✅ 订阅格式正确（Base64 编码）" -ForegroundColor Green
        } else {
            Write-Host "  ⚠️ 订阅内容不是标准 Base64，可能解析失败" -ForegroundColor Yellow
        }
        
        # 解码测试
        try {
            $decoded = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($resp.Content))
            $yaml = $decoded.Substring(0, [Math]::Min(200, $decoded.Length))
            Write-Host "`n  订阅内容预览:" -ForegroundColor Cyan
            Write-Host "  $yaml" -ForegroundColor Gray
        } catch {
            Write-Host "  ❌ Base64 解码失败，内容可能已损坏" -ForegroundColor Red
        }
    }
} catch {
    Write-Host "  ❌ 获取失败: $($_.Exception.Message)" -ForegroundColor Red
    
    if ($_.Exception.Message -match "403|Forbidden") {
        Write-Host "  💡 提示: 订阅链接可能已被 GitHub 缓存拦截"
        Write-Host "  💡 解决: 访问 https://clashvip.net 重新获取原始订阅链接"
    }
}

# 常见订阅格式错误
Write-Host "`n[3] 常见错误检查..." -ForegroundColor Yellow
$errors = @()
if ($SubscriptionUrl -match "github.com") {
    $errors += "⚠️ 使用了 GitHub Raw 链接，建议使用机场原始订阅链接"
}
if ($SubscriptionUrl -match "raw\.") {
    $errors += "⚠️ Raw 链接可能被缓存，内容不是最新"
}
if ($SubscriptionUrl.Length -gt 500) {
    $errors += "⚠️ URL 过长，可能被截断导致解析失败"
}
if ($errors.Count -eq 0) {
    Write-Host "  ✅ 未发现明显格式问题" -ForegroundColor Green
} else {
    $errors | ForEach-Object { Write-Host "  $_" -ForegroundColor Yellow }
}

Write-Host "`n========== 推荐订阅源 ==========" -ForegroundColor Cyan
Write-Host "ClashVIP 官方订阅: https://clashvip.net (实时更新，节点质量最优)" -ForegroundColor Green
Write-Host "机场导航: https://nav.clashvip.net (汇集多家订阅)" -ForegroundColor Cyan
```

---

### 故障 7：频繁断线

#### 原因分析

| 可能原因 | 诊断信号 | 解决方案 |
|---------|---------|---------|
| 网络不稳定 | 本地宽带频繁掉线 | 检查本地网络 |
| 节点服务器不稳定 | 特定节点断线 | 更换节点 |
| 订阅过期 | 断线后节点列表为空 | 更新订阅 |
| Clash 版本 BUG | 特定操作后断线 | 更新到最新版本 |
| TUN 模式冲突 | 开启 TUN 后断线 | 关闭 TUN 或重装驱动 |
| 心跳保活超时 | 长时间空闲后断线 | 配置 `keep-alive` |

#### 诊断脚本

```powershell
# connection-monitor.ps1 - 持续监控连接状态，记录断线事件
param(
    [string]$ProxyHost = "127.0.0.1",
    [int]$ProxyPort = 9090,
    [int]$IntervalSec = 10,
    [int]$DurationMin = 60
)

Write-Host "========== 连接监控器 ==========" -ForegroundColor Cyan
Write-Host "监控时长: ${DurationMin}min | 间隔: ${IntervalSec}s"
Write-Host "按 Ctrl+C 停止监控`n"

$startTime = Get-Date
$disconnectCount = 0
$lastStatus = $null
$log = @()

while ((Get-Date) -lt $startTime.AddMinutes($DurationMin)) {
    $timestamp = Get-Date -Format 'HH:mm:ss'
    
    try {
        $api = Invoke-RestMethod -Uri "http://${ProxyHost}:${ProxyPort}/proxies/GLOBAL" -TimeoutSec 5
        $currentNode = $api.now
        $upload = $api.upload
        $download = $api.download
        
        if ($lastStatus -eq "disconnected") {
            $disconnectCount++
            $log += "[$timestamp] 🔄 重新连接: $currentNode"
            Write-Host "[$timestamp] 🔄 重新连接: $currentNode" -ForegroundColor Green
        }
        
        Write-Host "[$timestamp] ✅ 连接正常 | 节点: $currentNode | 上传: $upload | 下载: $download" -ForegroundColor Gray
        $lastStatus = "connected"
        
    } catch {
        if ($lastStatus -ne "disconnected") {
            $log += "[$timestamp] ❌ 连接断开"
            Write-Host "[$timestamp] ❌ 连接断开！" -ForegroundColor Red
        }
        $lastStatus = "disconnected"
    }
    
    Start-Sleep -Seconds $IntervalSec
}

Write-Host "`n========== 监控报告 ==========" -ForegroundColor Cyan
Write-Host "总监控时间: $DurationMin 分钟"
Write-Host "断线次数: $disconnectCount"
Write-Host "断线频率: $([math]::Round($disconnectCount / $DurationMin, 2)) 次/小时"

if ($disconnectCount -gt 3) {
    Write-Host "`n⚠️ 断线过于频繁！" -ForegroundColor Red
    Write-Host "建议:"
    Write-Host "  1. 检查网络稳定性"
    Write-Host "  2. 更新 Clash 到最新版本"
    Write-Host "  3. 更换节点或更新订阅"
    Write-Host "  4. 检查是否开启了 TUN 模式"
}
```

---

## 📊 日志分析方法

### Clash 日志关键词速查

| 关键词 | 含义 | 行动 |
|--------|------|------|
| `parse error` | YAML 配置文件语法错误 | 检查配置文件语法 |
| `dial error` | 节点连接失败 | 更换节点 |
| `connection refused` | 端口被拒绝 | 检查端口或更换节点 |
| `timeout` | 连接超时 | 检查网络或节点状态 |
| `DNS poison` | DNS 污染 | 更换 DNS 服务器 |
| `tls: handshake failure` | TLS 握手失败 | 更换节点或开启 TLS 伪装 |
| `unsupported protocol` | 不支持该协议 | 确认节点协议类型 |
| `subscription` | 订阅相关 | 检查订阅是否过期 |
| `rule` | 规则匹配 | 查看规则优先级 |

### PowerShell 日志分析

```powershell
# analyze-logs.ps1 - 自动分析 Clash 日志中的错误和警告
param(
    [string]$LogPath = "$env:APPDATA\Clash for Windows\logs"
)

if (-not (Test-Path $LogPath)) {
    Write-Host "未找到日志目录: $LogPath" -ForegroundColor Red
    $LogPath = Read-Host "请输入 Clash 日志目录路径"
}

Write-Host "========== Clash 日志分析 ==========" -ForegroundColor Cyan
Write-Host "日志目录: $LogPath`n"

$logFiles = Get-ChildItem $LogPath -Filter "*.log" -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending

if (-not $logFiles) {
    Write-Host "未找到日志文件" -ForegroundColor Yellow
    exit
}

# 读取最近3个日志文件
$allContent = ""
foreach ($file in $logFiles | Select-Object -First 3) {
    $allContent += Get-Content $file.FullName -Tail 500 | Out-String
}

# 关键词分析
$keywords = @{
    "YAML/配置错误" = @("parse error", "yaml error", "invalid config", "decode error")
    "连接错误" = @("dial error", "connection refused", "timeout", "connect error")
    "DNS问题" = @("dns poison", "dns error", "lookup failed")
    "TLS错误" = @("tls:", "handshake", "certificate")
    "订阅问题" = @("subscription", "fetch error", "fetch failed")
    "规则问题" = @("rule:", "match", "direct")
}

foreach ($category in $keywords.Keys) {
    $kws = $keywords[$category]
    $matches = $allContent -split "`n" | Where-Object { 
        $line = $_
        $kws | ForEach-Object { if ($line -match $_) { return $true } }
        $false
    }
    
    if ($matches) {
        Write-Host "❌ $category ($(($matches | Measure-Object).Count) 条)" -ForegroundColor Red
        $matches | Select-Object -First 3 | ForEach-Object { 
            Write-Host "    → $_" -ForegroundColor Gray 
        }
        if (($matches | Measure-Object).Count -gt 3) {
            Write-Host "    → ... 等 $(($matches | Measure-Object).Count - 3) 条更多" -ForegroundColor Gray
        }
    } else {
        Write-Host "✅ $category - 无异常" -ForegroundColor Green
    }
}

Write-Host "`n💡 如发现持续错误，建议:"
Write-Host "  1. 更新 Clash 到最新版本"
Write-Host "  2. 删除旧配置文件重新导入"
Write-Host "  3. 联系 https://clashvip.net 客服"
```

---

## 🛡️ 预防措施体系

### 日常维护清单

```powershell
# maintenance-checklist.ps1 - 日常维护清单
Write-Host "======================================" -ForegroundColor Cyan
Write-Host "   Clash 日常维护清单 (建议每周执行)" -ForegroundColor Cyan
Write-Host "======================================" -ForegroundColor Cyan

$checks = @()

# 1. 订阅检查
Write-Host "`n[1] 订阅有效性..." -NoNewline
try {
    $url = "https://clashvip.net"
    $r = Invoke-WebRequest -Uri $url -TimeoutSec 10 -UseBasicParsing
    Write-Host "✅ 官网正常访问" -ForegroundColor Green
} catch {
    Write-Host "❌ 官网无法访问，请检查网络" -ForegroundColor Red
}

# 2. Clash 版本检查
Write-Host "[2] Clash 版本..." -NoNewline
$currentVer = (Get-Item "C:\Program Files\Clash for Windows\Clash for Windows.exe" -ErrorAction SilentlyContinue).VersionInfo.FileVersion
if ($currentVer) {
    Write-Host "当前版本: $currentVer" -ForegroundColor Cyan
    Write-Host "    💡 建议定期检查 https://github.com/Fndroid/clash_for_windows_pkg/releases" -ForegroundColor Gray
} else {
    Write-Host "⚠️ 未找到 Clash for Windows 安装路径" -ForegroundColor Yellow
}

# 3. 节点数量检查
Write-Host "[3] 节点数量..." -NoNewline
try {
    $api = Invoke-RestMethod -Uri "http://127.0.0.1:9090/proxies/GLOBAL" -TimeoutSec 5
    $count = ($api.all | Where-Object { $_ -notmatch "DIRECT|REJECT|GLOBAL" }).Count
    Write-Host "共 $($count) 个节点" -ForegroundColor Cyan
    
    if ($count -lt 10) {
        Write-Host "    ⚠️ 节点数量较少，建议更新订阅" -ForegroundColor Yellow
    }
} catch {
    Write-Host "⚠️ 无法获取节点数量（Clash API 未开启）" -ForegroundColor Yellow
}

# 4. 端口检查
Write-Host "[4] 核心端口监听..." -NoNewline
$ports = @(7890, 7891, 9090)
$activePorts = $ports | Where-Object { 
    Get-NetTCPConnection -LocalPort $_ -ErrorAction SilentlyContinue 
}
if ($activePorts) {
    Write-Host "$($activePorts -join ', ') 端口正常" -ForegroundColor Green
} else {
    Write-Host "❌ 所有核心端口均未监听，Clash 可能未运行" -ForegroundColor Red
}

# 5. DNS 缓存清理
Write-Host "[5] DNS 缓存清理..." -NoNewline
ipconfig /flushdns | Out-Null
Write-Host "已完成" -ForegroundColor Green

Write-Host "`n======================================" -ForegroundColor Cyan
Write-Host "维护检查完成。如有问题，请访问:" -ForegroundColor Cyan
Write-Host "  机场官网: https://clashvip.net" -ForegroundColor Green
Write-Host "  机场导航: https://nav.clashvip.net" -ForegroundColor Cyan
Write-Host "  教程社区: https://clashhub.net" -ForegroundColor Cyan
Write-Host "  用户论坛: https://bbs.clashhub.net" -ForegroundColor Cyan
```

---

## ❓ 常见问题 FAQ

### Q: Clash 启动后没有任何反应怎么办？

A: 检查是否有多余的 Clash 实例在后台运行（`taskkill /IM "Clash for Windows.exe" /F`），然后重新启动。

### Q: 订阅更新后节点列表是空的怎么办？

A: 很可能是 Base64 解码失败。用 `base64 -d` 命令手动解码订阅内容，确认是否完整 YAML。

### Q: 开了系统代理但浏览器还是直连怎么办？

A: 部分浏览器（Firefox/Edge）有自己的代理设置，需单独配置或安装 SwitchyOmega 插件。

### Q: TUN 模式开启后无法上网怎么办？

A: 这是最常见的 TUN 问题。解决方案：① 更新 Clash 到最新版；② 重装 TUN 驱动；③ 尝试不同的 TUN 模式（enhanced/generic）。

### Q: Clash 占用了大量内存怎么办？

A: 可能是规则文件过大（>1MB）。检查 `rule-providers` 中的规则文件大小，精简不需要的规则集。

### Q: 如何彻底卸载 Clash？

A: 停止 Clash → 删除安装目录 → 删除 `%APPDATA%\Clash*` 和 `%LOCALAPPDATA%\Clash*` → 删除系统代理设置 → 重启电脑。

---

## 🔗 推荐资源

- **ClashVIP 官网**: https://clashvip.net — 100+ 优质节点，7×24 客服
- **机场导航**: https://nav.clashvip.net — 汇集优质订阅
- **Clash 教程**: https://clashhub.net — 配置教程与故障排查
- **用户论坛**: https://bbs.clashhub.net — 真实用户经验分享
- **客户端下载**: https://clash-for-windows.net — 最新版下载
- **Clash Wiki**: https://github.com/Dreamacro/clash/wiki — 官方文档

---

**更新日期**: 2026-08-20  
**版本**: v2.0  
**状态**: 已深度扩展，含完整 PowerShell/Python 诊断脚本
