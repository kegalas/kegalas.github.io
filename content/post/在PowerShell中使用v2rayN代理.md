---
title: "在PowerShell中使用v2rayN代理"
date: 2022-08-06T16:33:44+08:00
draft: false
tags: [Windows,代理,PowerShell,v2rayN]
categories: 其他
---

通常，v2rayN设置中的ip是127.0.0.1，http协议的端口是10809，在PowerShell中临时使用代理，需要输入如下命令

```
$env:HTTP_PROXY="http://127.0.0.1:10809"
$env:HTTPS_PROXY="http://127.0.0.1:10809"
```

之后输入

```
curl www.google.com
```

来检验是否成功。如果v2rayN本身是能够连接到谷歌的，那么得到的结果应该类似于下方：

```
StatusCode        : 200
StatusDescription : OK
Content           : <!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="en"><head><meta conten
                    t="Search the world's information, including webpages, images, videos and more. Google has many spe
                    ci...
RawContent        : HTTP/1.1 200 OK
......
```

另外不要使用ping命令去测试，虽然不明原因但是确实无法ping到谷歌。

