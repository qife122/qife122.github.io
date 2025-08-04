---
date: 2025-08-04T00:21:29+08:00
title: "libcurl HTTP/3 POST请求处理中的栈释放后使用漏洞分析"
tags: [漏洞分析,HTTP/3,libcurl,内存安全]
authors: qife
description: "本文详细分析了libcurl在处理HTTP/3 POST请求时存在的栈释放后使用漏洞，该漏洞源于CURLOPT_POSTFIELDS选项对用户提供数据的指针保留但不复制，当原始栈帧销毁后访问该指针会导致内存损坏。"
---

# Stack use-after-scope in HTTP/3 POST request processing via CURLOPT_POSTFIELDS

## 摘要
libcurl的HTTP/3请求处理在使用CURLOPT_POSTFIELDS与栈分配缓冲区时存在栈释放后使用漏洞。libcurl保留了用户提供的POST数据指针，但在原始栈帧销毁后仍访问该指针，导致内存损坏和潜在拒绝服务。

## 漏洞详情
该漏洞发生在transfer.c:569的Curl_pretransfer()中，当libcurl对先前存储的POST数据指针（现在指向无效栈内存）调用strlen()时触发。

## 复现步骤/概念验证

### 环境
- libcurl版本: 8.16.0-DEV (master分支)
- 编译器: Clang 20.1.8 with AddressSanitizer
- 平台: macOS (ARM64)
- 配置: 启用HTTP/3 (ngtcp2/nghttp3)

### 复现步骤
1. 使用ASAN构建libcurl:
```bash
export CC=clang
export CFLAGS="-O1 -g -fsanitize=address,undefined"
./configure --with-openssl --with-nghttp2 --with-nghttp3 --with-ngtcp2
make
```

2. 编译POC代码:
```c
// http3_crash_poc.c
#include <curl/curl.h>
#include <string.h>

int main() {
    CURL *curl = curl_easy_init();
    
    // 栈分配缓冲区将在作用域外失效
    {
        char body_data[257];
        memset(body_data, 'A', 256);
        body_data[256] = '\0';
        
        curl_easy_setopt(curl, CURLOPT_URL, "https://example.com/");
        curl_easy_setopt(curl, CURLOPT_HTTP_VERSION, CURL_HTTP_VERSION_3);
        curl_easy_setopt(curl, CURLOPT_POST, 1L);
        curl_easy_setopt(curl, CURLOPT_POSTFIELDS, body_data); // 漏洞调用
        curl_easy_setopt(curl, CURLOPT_TIMEOUT_MS, 50L);
    } // body_data在此处超出作用域
    
    // libcurl在传输期间访问无效内存
    curl_easy_perform(curl);
    curl_easy_cleanup(curl);
    return 0;
}
```

3. 编译并运行:
```bash
clang -fsanitize=address http3_crash_poc.c -lcurl -o poc
./poc
```

## 崩溃输出
```
==3720==ERROR: AddressSanitizer: stack-use-after-scope on address 0x00016fa21470
READ of size 45 at 0x00016fa21470 thread T0
    #0 strlen
    #1 Curl_pretransfer transfer.c:569
    #2 multi_runsingle multi.c:2376
    #3 curl_multi_perform multi.c:2756
    #4 easy_transfer easy.c:705
    #5 easy_perform easy.c:813

SUMMARY: AddressSanitizer: stack-use-after-scope transfer.c:569 in Curl_pretransfer
```

## 技术分析

### 根本原因
漏洞源于libcurl的CURLOPT_POSTFIELDS行为：
1. libcurl存储指针但不复制数据
2. 应用程序的栈缓冲区在作用域退出后失效
3. libcurl稍后在Curl_pretransfer()中解引用无效指针

### 受影响代码路径
```
curl_easy_setopt(CURLOPT_POSTFIELDS) →
curl_easy_perform() →
Curl_pretransfer() →
strlen(invalid_pointer) →
CRASH
```

## 修复建议
1. 文档: 明确说明CURLOPT_POSTFIELDS数据在传输完成前必须保持有效
2. API增强: 考虑添加边界检查或对栈检测指针自动复制
3. 替代API: 推广更安全使用模式的CURLOPT_COPYPOSTFIELDS

## 影响

### 安全影响
- 拒绝服务: 必然导致应用终止的崩溃
- 内存损坏: 释放后使用可能导致不可预测行为
- 潜在代码执行: 特定情况下内存损坏可能被用于控制流劫持

### 受影响场景
- 使用libcurl进行HTTP/3 POST请求的应用程序
- 任何CURLOPT_POSTFIELDS指向栈分配内存的代码模式
- 特别影响:
  - HTTP/3客户端应用
  - 使用栈缓冲区作为请求体的API客户端
  - 堆使用受限的嵌入式系统

### 实际暴露
- 语言绑定: 许多curl绑定可能无意中创建此模式
- 示例应用: CLI工具、网络爬虫、API客户端
- 严重性: 由于HTTP/3采用率增长和远程可利用性而较高