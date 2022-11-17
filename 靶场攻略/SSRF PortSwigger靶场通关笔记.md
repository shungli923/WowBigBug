# SSRF PortSwigger靶场通关笔记

> 本靶场通关笔记只记录通关过程中思考和实现的过程，不包含相关原理的讲解，要是有不明白相应步骤为何这样操作的师傅们可以移步 SSRF原理篇 结合着一起学习。

## Lab: Basic SSRF against the local server

看实验要求，需要找到一个访问内部系统的URL然后通过这个删除一个用户。进去看看。

![image-20221113194950250](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113194950250.png)

马上定位到目标URL，发到repeater模块准备一顿操作

![image-20221113195750946](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113195750946.png)

发送payload如下,表面上什么都没有，好像被拦截了，不过咱返回了咱们这么多信息，好歹看完，看看有没有返回一些其他的接口来着。

```http
http://localhost/admin
```

![image-20221113195824204](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113195824204.png)

果然发现了一个接口：

![image-20221113195959987](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113195959987.png)

把这个接口重新作为payload发送，看响应包是一个302跳转。好像是成功了。

![image-20221113200050491](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113200050491.png)

重新返回admin目录看看还有没有这个东西，结果是没有了

![image-20221113200220307](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113200220307.png)

**通关成功**

## Lab: Basic SSRF against another back-end system

看实验要求简单来说就是要我们进行一个内网存活探测，找到存活页面后对该页面进行操作。

![image-20221113202222401](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113202222401.png)

定位到目标页面后，发到intruder模块进行探测

![image-20221113202344431](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113202344431.png)

设置爆破点，然后设置数字字典1到255，进行攻击！简单的等待之后，显然发现了不一样的地方

![image-20221113203021920](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113203021920.png)

把这个页面发给repeater模块，看响应包发现有两个删除用户接口

![image-20221113203124062](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113203124062.png)

任意访问一个看看能不能实现删除，选择删除carlos，到了一个302的跳转页面

![image-20221113203215662](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113203215662.png)

在此访问admin目录，看显然用户 carlos 被删除成功

![image-20221113203310992](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221113203310992.png)

这里就成功利用SSRF 进行了内网探测和用户删除的操作。

**通关成功**

## Lab: SSRF with blacklist-based input filter

看实验介绍，说是用SSRF删一个用户，然后说有两个防御机制需要绕过。

![image-20221114201853851](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114201853851.png)

直接定位到目标地点，然后尝试SSRF，可以看到存在防御

![image-20221114202135257](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114202135257.png)

访问http://127.0.0.1，可以看见依然被绕过。

![image-20221114202557772](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114202557772.png)

访问http://127.1，返回html页面

![image-20221114202649243](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114202649243.png)

访问http://127.0.1/admin，发现被阻挡

![image-20221114202741427](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114202741427.png)

对admin进行url编码，访问127.1/%61%64%6d%69%6e

![image-20221114202906624](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114202906624.png)

对admin进行二次url编码，访问http://127.1/%61%64%6d%69%%36%65，返回html页面

![image-20221114203037680](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114203037680.png)

发现两个接口，访问接口进行删除，直接访问依然被拦截。

![image-20221114203230119](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114203230119.png)

尝试二次url编码绕过，发现302跳转

![image-20221114203406989](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114203406989.png)

再次访问admin页面，carlos用户已经被删除

![image-20221114203446522](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114203446522.png)

**通关成功！**

## Lab: SSRF with whitelist-based input filter

看要求，需要我们通过SSRF删除某用户，然后善意提示我们一下有防护需要进行一定绕过。

![image-20221114205659662](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114205659662.png)

简单进行了尝试，说是必须要有 stock.weliketoshop.net 很明显的白名单绕过

![image-20221114213920787](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114213920787.png)

网上查了一下，SSRF白名单绕过要么用 `@` 要么用 `#` 把攻击的域名拼接上去，于是进行尝试。

在高光地点尝试了 `@` 和 `@#` ，出现如下页面，显示服务器内部错误，依然无法利用。

![image-20221114214243436](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114214243436.png)

在高光地点尝试了 `#` 和 `#@` ，出现如下页面，依然无法利用。

![image-20221114214433031](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114214433031.png)

之后尝试在这个地方进行各种编码（对 `@` 一次到三次URL编码，对 `#` 进行一次到三次编码，对 `@#` 进行一次到三次URL编码，对 `#@` 进行一次到三次URL编码），都依然没有进展，响应包的标志要么是400，要么是500。

![image-20221114215509887](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114215509887.png)

于是把127.0.0.1提到前面去，在红框中尝试 `@` 、 `#` 和 `@#` 、`#@` 以及相关进行一次二次三次编码之后响应包和上面的差不多。

![image-20221114214754003](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114214754003.png)

直到尝试了这样一个组合之后发现了转机，`%25%32%33@` —》就是对 `#` 进行二次URL编码，然后拼接上 `@` 符号。（md服了，试半天才试出来）

![image-20221114220411214](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114220411214.png)

然后访问admin目录，找到删除用户接口

![image-20221114220800370](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114220800370.png)

之后就是一系列熟悉的操作了，调用接口。

![image-20221114220901306](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114220901306.png)

![image-20221114220931778](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221114220931778.png)

**通关成功！**



## Lab: SSRF with filter bypass via open redirection vulnerability

看实验要求是说需要我们去到http://192.168.0.12:8080这个机器的admin目录页然后删除一个用户。开干！

![image-20221115212532458](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115212532458.png)

在数据包堆堆里一顿梭哈只发现这个数据包可能跟SSRF有点关系，开始黑盒测试。

![image-20221115213133004](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115213133004.png)

对着两个参数一顿fuzz，结果啥也没有，根本没有反应，于是又去找其他数据包看看，发现了一个包返回的MIMETYPE是HTML，并且有完整的返回内容，仔细分析其返回包，发现了了不得的东西，他好像还有一个参数叫做 path，可以通过这个参数访问其他路径

![image-20221115214215696](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115214215696.png)

很巧的是我们最初定位的数据包就包含了这个路径应用，于是进行操作，看看能不能重定向到实验要求的其他主机的admin目录下。在无尽的fuzz之后，遇到无数次的红框内容之后我迷茫了。

![image-20221115215611682](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115215611682.png)

不理解为啥不行，于是又去对比数据包看看啥原因。

![image-20221115215804110](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115215804110.png)

![image-20221115215852692](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115215852692.png)

好像能理解了，因为我们是要用重定向来绕过防御达到SSRF，而在先前的尝试参数根本没有访问链接的能力，只有nextProduct下面的path参数才可以有这个功能，而开始的check下面根本没有path这个参数。草了，应该是这个原因。一尝试，果然如此。终于构造出来了。

![image-20221115221033928](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115221033928.png)

接下来就是访问删除链接进行用户删除了，

![image-20221115221212379](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221115221212379.png)

**通关完成！**

## Lab: Blind SSRF with out-of-band detection

来到实验要求环节，说是内部监测机器会监测Referer头里的内容，尝试能否通过这个地点达到SSRF

![image-20221116143706751](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116143706751.png)

一顿操作之后在两个地方发现了referer头，一个是所有商品陈列加载时（第一个图），一个是某一个商品详细信息加载时（第二个图）。

![image-20221116143940902](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116143940902.png)

![image-20221116144015365](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116144015365.png)

在burp里弄各个监听器看看能不能被请求到，经过一番尝试，最终在商品详细信息加载页面在将相应的Referer头换成burp生成的可控链接后，成功收到了相应的请求。

![image-20221116145844466](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116145844466.png)

**通关成功！**

## Lab: Blind SSRF with Shellshock exploitation

这一个和上一个差不多，故事都是从referer头展开，只是还得进行内网探测，然后找到存活的机器，并且发送指令得到以机器当前用户信息。好难啊，给我愁坏了不知道从哪里开始。

![image-20221116153534599](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116153534599.png)

做了很久没辙，最后看了官方答案，需要使用到一个叫 `Collaborator Everywhere` 的插件。安装插件：

![image-20221116155101315](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116155101315.png)

然后把实验靶场的域名添加到burp监测里面。

![image-20221116155141096](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116155141096.png)

设置好了之后就是随意的访问网站，可以看到下图，当访问到某一个product的具体信息时，插件监测的信息得到了回复，明显看到 `Referer` 和 `UA` 俩个地方都是可以进行与外界进行交互的。

![image-20221116163223388](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116163223388.png)

由于题目要求是需要获得存活主机的用户名，而此刻不知道具体存活的机器，因此将该数据包发往intruder模块，其中参数设置如下，通过1-255进行内网探测，点击start Attack之后。

注：UA头那里那一串看不懂的东西就叫 `shellshock` ，不懂或者不明白的可以在 SSRF 漏洞原理讲解篇查看。

![image-20221116163902046](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116163902046.png)

等待进攻完成，这是已经得到回显，成功通过DNS获取到某主机用户名

![image-20221116171245367](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221116171245367.png)

本质上来说已经是通关成功了，如果我们需要知道具体是哪一个ip的主机的用户名被我们知道了，怎么定位到具体的机器呢？有一个笨方法。

在这个地方可以生成255个colaborator地址，然后在intruder模块中设置一个ip对应一个地址，这样在结果中根据DNS解析的colaborator地址即可锁定具体的机器。

**通关成功！**
