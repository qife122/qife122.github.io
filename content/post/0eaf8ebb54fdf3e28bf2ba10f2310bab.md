---
date: 2025-08-05T00:59:01+08:00
title: Azure Front Door WAF IP限制绕过漏洞分析
tags: [网络安全, Azure, WAF绕过, 云安全]
authors: qife
description: 本文披露Azure Front Door WAF的IP限制功能存在严重设计缺陷，攻击者可通过X-Forwarded-For标头轻易绕过IP白名单保护，该问题已被微软确认为预期行为而非漏洞。
---

# Azure's Front Door WAF WTF: IP限制绕过

## 背景
Azure提供两种WAF服务：全局部署的Front Door WAF和区域部署的Application Gateway WAF。本文披露的漏洞存在于Front Door WAF的IP限制功能中。

当管理员配置IP限制规则时，默认匹配变量为"RemoteAddr"，该变量会优先检查X-Forwarded-For HTTP头而非真实客户端IP。而另一个选项"SocketAddr"才真正验证连接IP地址。


## 漏洞验证
1. **测试环境搭建**
   - 部署测试站点并配置Front Door WAF
   - 创建自定义规则：仅允许IP 123.45.67.89访问（使用RemoteAddr变量）
   - 启用WAF防护模式

2. **绕过验证**
   ```bash
   curl -H "X-Forwarded-For: 123.45.67.89" https://victim.site
   ```
   添加恶意X-Forwarded-For头即可绕过IP限制

## 技术细节
- **变量差异**：
  | 服务            | 变量名    | 行为                  |
  |----------------|----------|----------------------|
  | Front Door WAF | RemoteAddr | 信任X-Forwarded-For头 |
  | Front Door WAF | SocketAddr | 验证真实连接IP        |
  | App Gateway WAF| RemoteAddr | 验证真实连接IP        |

- **攻击影响**：
  - 使用Burp Intruder可在40分钟内暴力破解/16网段
  - 匹配自定义规则后会跳过所有OWASP防护检测

## 检测与修复
**检测脚本**：
```powershell
./azure_frontdoor_waf_wtf.ps1
```

**修复方案**：
1. 将规则变量改为SocketAddr
2. 如需代理支持，应组合规则：
   ```plaintext
   IF RemoteAddr IN [allowed_ips] 
   AND SocketAddr = [proxy_ip]
   ```

## 结论
微软将该行为标记为预期功能而非漏洞，但存在严重设计缺陷：
1. 相同变量名在不同产品中行为不一致
2. HTTP头验证被错误标记为"IP限制"
3. 默认选项存在安全风险且无明确警告

管理员应立即检查现有规则配置，避免依赖不安全的RemoteAddr变量。

> 特别感谢Aaron James @TrustedSec的技术反馈