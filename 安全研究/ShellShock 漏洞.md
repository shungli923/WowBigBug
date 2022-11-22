# ShellShock 漏洞

Shellshock是一个威胁性较高的漏洞，它能够给予了攻击者更高的特权，借此攻击者甚至可以随意破坏系统。该漏洞第一次爆出是 CVE-2014-6271，此漏洞在2018年又进行了更新：CVE-2014-7169。

Shellshock是Bash Shell(GNU Bash最高版本为4.3）中的一个安全漏洞，该漏洞源自于Bash从环境变量可以执行任意高权限Bash命令。利用此漏洞的威胁参与者可以在目标主机上远程发出命令。

当Web服务器接收到对页面的请求时，该请求的三个部分都可能会受到Shellshock攻击：请求URL，与URL一起发送的标头 以及 所谓的“参数”（当您在网站上输入您的姓名和地址，通常会将其作为参数发送给请求）。

例如，以下是检索example主页的实际HTTP请求：

![](https://raw.githubusercontent.com/shungli923/PicGoImg/master/WLAQ202102006_05200.jpg)

在此例中，URL是网址，标头为Accept-Encoding，AcceptLanguage等。这些标头向Web服务器提供有关用户Web浏览器功能，首选语言，使用的浏览器等信息。

服务器会将它们转换为Web服务器内部的变量以便于处理。例如：Web服务器想知道用户首选的语言是什么，以便于决定如何回应用户。

服务器在响应用户对于exmaple主页的请求时，可以通过逐个字符复制请求标头来定义以下变量：

![](https://raw.githubusercontent.com/shungli923/PicGoImg/master/111.jpg)

当将变量传递到称为 bash 的 shell 中时，会发生 ShellShock 。Bash 是 Linux 系统上使用的通用 shell 。Web 服务器经常需要运行其他程序来响应请求，并且通常将这些变量传递到 bash 或另一个 shell 中。

当攻击者修改原始 HTTP 请求以包含神奇的`(){:;};`时，假设攻击者把HTTP头部中的User-Agent内容改为：`(){:;}; /bin/eject`，就会在服务器端创建如下变量：`HTTP＿USER＿AGENT=(){:;}; /bin/eject`。

**在Bash运行时遇到 `(){:;};` 类似格式就将其当成环境变量赋值，而Bash脚本在解析“函数环境变量”的时并没有以函数结尾 } 结束,而是一直执行后面的 shell 命令,也就是向 bash 注入一段命令，因而导致漏洞产生。**从 bash1.14 至 u4.3 都存在这些漏洞。

这是因为 bash 对于处理以`(){:;};`开头的变量具有特殊的规则。bash 不会将变量 HTTP＿USER＿AGENT 视为没有特殊含义的字符序列，而是将其解释为需要执行的命令。

回到上述例子，因为 HTTP＿USER＿AGENT 的变量值来自于 User-Agent 标头，攻击者可以控制此标头，因为它是通过HTTP请求进入Web服务器的，从而攻击者可以使易受攻击的服务器运行所需的任何命令。

大多数 Shellshock 命令是使用 HTTP User-Agent 和 Referer 标头注入的，但也有攻击者使用 GET 和 POST 参数以及其他随机 HTTP 标头。

为了提取私人信息，攻击者使用了两种技术。最简单的提取攻击的形式为：`(){:;}; /bin/cat/etc/passwd`。

以上命令将读取密码文件  /etc/passwd，并将其添加到 Web 服务器的响应中。因此，通过 Shellshock 漏洞注入此代码的攻击者将看到密码文件作为返回的网页的一部分显示在他们的屏幕上。
