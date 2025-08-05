---
date: 2025-08-05T19:59:42+08:00
title: SharePoint跨站脚本(XSS)漏洞安全通告发布
tags: [安全通告, SharePoint, XSS漏洞, 权限提升]
authors: qife
description: 微软发布安全通告983438，披露SharePoint Server 2007和SharePoint Services 3.0中存在的跨站脚本漏洞，可能导致站点内权限提升问题，文章详细说明了缓解措施及IE8过滤器的防护作用。
---

微软今日发布安全通告983438，针对SharePoint Server 2007和SharePoint Services 3.0中存在的跨站脚本(XSS)漏洞。该漏洞可能导致攻击者在SharePoint站点内实现权限提升(EoP)。使用Internet Explorer 8客户端的服务器风险较低，因为IE8的XSS过滤器能在互联网区域缓解此问题。目前尚未发现活跃攻击。

运行SharePoint Server 2007或SharePoint Services 3.0的客户应审阅并实施安全通告中讨论的缓解措施和临时解决方案，包括限制对SharePoint help.aspx XML文件的访问，以及在内部网区域启用IE8 XSS过滤器。

微软正通过Microsoft Active Protections Program(MAPP)与合作伙伴积极协作，提供可扩展客户保护的解决方案信息。微软始终致力于与安全研究人员合作解决软件漏洞，确保用户在漏洞被恶意利用前获得全面高质量的安全更新。

受影响用户可访问http://support.microsoft.com获取支持，并建议联系所在国家的执法机构。我们将通过博客和Twitter(@msftsecresponse)持续更新进展。

*本文档按"原样"提供，不附任何担保，亦不授予任何权利。*