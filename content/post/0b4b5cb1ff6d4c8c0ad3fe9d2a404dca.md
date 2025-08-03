---
date: 2025-08-03T04:53:11+08:00
title: 在Linux上运行原版Intercepter-NG网络嗅探工具
tags: [网络安全, Linux, Wine, 数据包嗅探]
authors: qife
description: 本文详细介绍了在Kali Linux系统上通过Wine运行Intercepter-NG 0.9.9网络嗅探工具的具体步骤，包括WinPcap封装库安装、32位兼容环境配置以及常见错误解决方法。
---

# 在Linux上运行原版Intercepter-NG

## 操作指南（已更新！）

[1] 下载Wine用的WinPcap封装库及libpcap-dev
```bash
wget http://sniff.su/wine_pcap_dlls.tar.gz
apt-get install libpcap-dev
```
如果是i386版Kali系统，直接跳转至步骤[3]。

---

[2] Kali x64系统需执行以下命令：
```bash
dpkg --add-architecture i386
apt-get update
apt-get install wine-bin:i386
apt-get install tcpdump:i386
```

---

[3] 将dll文件复制到wine库目录
```bash
cp wpcap.dll.so /usr/lib/i386-linux-gnu/wine
cp packet.dll.so /usr/lib/i386-linux-gnu/wine
```

[4] 安装winetricks并配置
```bash
apt-get install winetricks
winetricks cc580
ethtool --offload eth0 rx off tx off
```

[5] 下载Intercepter-NG 0.9.9并移除冲突dll
```bash
rm wpcap.dll
rm packet.dll
wine Intercepter-NG.exe
```

## 常见问题解答

1. **文件缺失错误**  
确保从压缩包完整提取所有文件，包括`packet.dll.so`

2. **配置文件报错**  
必须以root用户身份（非sudo）在程序所在目录直接运行

3. **Kali 2.0兼容性问题**  
建议使用Wine 1.4版本运行程序

4. **64位系统支持**  
需完整安装32位兼容库：
```bash
dpkg --add-architecture i386
apt-get install ia32-libs -y
```

> 注意：程序开发者不提供Linux系统或第三方应用的配置支持，本教程仅适用于Kali 1.x/2.0系统环境。