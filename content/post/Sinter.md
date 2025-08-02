---
title: "Sinter：macOS全新用户态安全执行框架解析"
tags: [macOS安全, 端点防护, Swift编程, 用户态安全]
authors: qife
description: "Sinter是首个完全用Swift编写的开源macOS端点防护代理，基于苹果EndpointSecurity API实现实时事件授权，解决了内核态到用户态迁移中的关键技术挑战。"
---

# Sinter：macOS全新用户态安全执行框架

## 简单、开源且基于Swift

Sinter是我们用Swift编写的开源macOS端点安全执行代理（支持10.15及以上系统）。它完全基于用户态构建，利用新的EndpointSecurity API接收macOS内核发出的安全事件回调。通过简单规则即可控制事件允许/拒绝，无需传统杀毒软件的全盘扫描或特征检测。

## 纯用户态安全代理的探索

实现端点安全解决方案需要实时拦截和授权OS级事件。过去这意味着使用内核态回调API或直接挂钩内核代码。苹果在2019年末宣布将弃用所有第三方内核扩展，引入EndpointSecurity API作为替代方案。

## EndpointSecurity API解析

该API实现了macOS内核的实时回调机制，支持NOTIFY（通知）和AUTH（授权）两种事件类型。它取代了原有的Kauth KPI等内核态方案，成为macOS 11 Big Sur后唯一合法的实时监控接口。

## Sinter开发中的关键技术挑战

1. **实时决策不影响系统性能**  
   采用异步处理架构，通过`es_copy_message`解耦消息处理，建立双优先级队列（常规程序和大程序分离），确保系统响应能力。

2. **防范TOCTOU竞态条件**  
   针对"检查时-使用时"时间差攻击，实现文件事件监控机制。当检测到执行文件被篡改时立即清除缓存（已向苹果提交改进建议FB8352031）。

3. **应用包代码签名验证**  
   macOS可执行文件存在于应用包（.app）中，需验证整个包的签名。Sinter创新性地实现双缓存机制：EndpointSecurity原生缓存+自定义应用包验证缓存。

4. **系统扩展安装优势**  
   作为System Extension部署可获得系统级保护（包括SIP扩展防护），防止root用户卸载。未来版本将迁移到此模式。

5. **证书签名与公证流程**  
   建立自动化CMake工作流处理苹果严格的代码签名、公证和权限申请（EndpointSecurity权限需6周人工审核）。

## 未来展望

Sinter将持续增强规则引擎灵活性，整合文件完整性监控、内存注入防护（通过mmap事件分析）及NetworkExtension网络流量控制。我们欢迎社区通过GitHub或Empire Hacking Slack频道参与贡献。

> 随着内核扩展的弃用，苹果为端点防护建立了统一用户态API标准，这将提升系统稳定性并减少攻击面，而Swift语言的选择确保了长期兼容性优势。