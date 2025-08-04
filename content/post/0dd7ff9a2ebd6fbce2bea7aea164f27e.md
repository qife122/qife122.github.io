---
date: 2025-08-04T14:40:46+08:00
title: Palo Alto Networks 关键安全更新：CVE-2024-5910 和 CVE-2024-3596 漏洞分析
tags: [网络安全, 漏洞挖掘, 系统安全]
authors: qife
description: 本文详细分析了Palo Alto Networks发布的两大高危漏洞（CVE-2024-5910和CVE-2024-3596），涉及Expedition迁移工具配置泄露和RADIUS协议权限提升风险，并提供完整的修复方案和防护建议。
---

### 摘要  
Palo Alto Networks发布关键安全更新，修复了包括Expedition迁移工具高危漏洞（CVE-2024-5910，CVSS评分9.3）在内的多个安全问题。该工具用于配置迁移与优化，但漏洞会导致导入的配置密钥、凭证等敏感数据暴露，攻击者可借此劫持管理员账户。  
另一漏洞（CVE-2024-3596）存在于RADIUS协议中，通过中间人攻击可实现权限提升至"超级用户"，影响PAN-OS防火墙与RADIUS服务器的认证过程（当使用CHAP或PAP时）。

### 受影响系统  
**CVE-2024-5910**  
- Expedition（1.2.92之前版本）  

**CVE-2024-3596**  
- PAN-OS（版本<11.1.3、11.0.4-h4、10.2.10、10.1.14、9.1.19）  
- Prisma Access（预计7月30日前修复）  

### 紧急措施  
1. **升级软件**：Expedition升级至1.2.92+，PAN-OS升级至指定版本。  
2. **访问限制**：严格控制Expedition工具的网络访问权限。  
3. **RADIUS配置**：避免在未加密通道中使用CHAP/PAP协议。  

### 长期建议  
- 及时应用所有安全补丁  
- 定期审查认证协议与网络访问策略  

官方公告详见：[Palo Alto Networks安全通告](https://security.paloaltonetworks.com/CVE-2024-5910)  

---