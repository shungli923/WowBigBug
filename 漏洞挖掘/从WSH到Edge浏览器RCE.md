# 组合攻击通过Edge浏览器进行RCE

> 本文来自微软安全研究员（Abdullahman Alqabandi）
> 原文链接：https://leucosite.com/Microsoft-Edge-RCE/

## 应用内启动外部程序

当我们在浏览器 URL 栏里输入mailto:test@test.test 时，屏幕会弹出一个提示框，询问是否要切换应用，一旦用户同意，则切换的应用程序将会直接运行。

![img](https://leucosite.com/qimg/Art17-sub0.png)

在这个例子里，因为 Outlook 是系统默认的邮件服务应用程序，从下面的图可以看到，Outlook 被启动的原因是因为某些参数被发送到了 Outlook 的可执行文件里：

![](https://raw.githubusercontent.com/shungli923/PicGoImg/master/20221104145832.png)

所以，如果我们输入不正常的参数，很明显将为导致异常出现。那么问题来了，除了邮件客户端，还有那些应用程序可以通过这样的方式启动呢？

## 一个很方便的协议

在注册表里，我们可以找到所有的正在使用的自定义协议，在 `Computer\HKEY_CLASSES_ROOT\` 目录里，我发现这个目录下的很多文件夹里都含有 `shell\open\command` 这条路径。例如 `ms-word` 目录里便存在这样一条路径：

![](https://raw.githubusercontent.com/shungli923/PicGoImg/master/20221104145856.png)

以 `ms-word` 为例，可以发现这样一条数据 `C:\Program Files (x86)\Microsoft Office\Root\Office16\protocolhandler.exe "%1"` 。和上述邮件服务器相同，当这个语句被触发时，相应给参数将会传递给 `ms-word` 的可执行文件。这时出现了下面的的情况：

![img](https://leucosite.com/qimg/Art17-sub2.png)

简单尝试了一下是否其他参数可以让 `ms-word` 可执行文件接受，无果。便准备查寻找一些更轻松的目标

![img](https://leucosite.com/qimg/Art17-sub4.png)

终于找到了一个叫 `WScript.exe` 可以直接将恶意参数成功传递。在这之前，如果你不知道什么是 `Windows 脚本宿主文件`（ Windows Script Host，简写为 WSH），您可以阅读一下这篇文章：https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wscript 。链接是一个英语网页，简单来说就是 `WSH` 通过各种不同对象模型的运转来模拟支持脚本语言环境，这样用户可以在` WSH `中进行执行各种不同的脚本语言。现在咱们看一下当在 Edge浏览器的 URL 输入栏输入 `wshfile:test` 的时候会发生什么吧。

首先，我们可以看到一个提示框是否选择默认程序运行，从注册表中可以看到，是通过 Windows Script Host (WScript.exe) 导致这个询问框的：

![img](https://leucosite.com/qimg/Art17-sub5.png)

点击 `OK` 键出现下面的东东

![img](https://leucosite.com/qimg/Art17-sub6.png)

可以看到，`WScript.exe` 尝试运行用户传递的路径出的文件，它尝试定位到路径 `C:\WINDOWS\system32\wshfile:test` 处的文件，但显然文件并不存在。那我们又能做什么呢？是否可以放一个名叫 `wshfile:test` 的文件，不行。那可以怎么做？

## 利用

从图片可以看出，第一个测试成果很明显：目录遍历。在浏览器 URL 栏里输入 `wshfile:test/../../foo.vbs` 后，点击回车键后出现如下提示框

![img](https://leucosite.com/qimg/Art17-sub7.png)

牛啊，现在我们可以定位任何目录里的任何文件，只要我们可以将文件放在一个可预测的地方，那我们就可以进行 RCE ！但说起来容易做起来难。看起来好像大多数的 Edge 浏览器缓存文件都进入了 Salted 目录。换句话说，我们可以把放置文件，但是不能知道文件被放置的具体位置。

这时我想起马特·尼尔森写的一篇很棒的文章【文章地址：https://enigma0x3.net/2017/08/03/wsh-injection-a-case-study/】，文章依旧是全英文的， 这篇文章里说 Windows 在 `C:\Windows\System32\Printing_Admin_Scripts\en-US\pubprn.vbs` 位置有 vbs 后缀的文件会受到 WSH注入 的影响，这个的 VBS 文件可以接受两个参数，通过对参数构造从而欺骗 VBS 脚本编译器 进而造成任意代码执行。遗憾的是这个漏洞早已被修复，目前只对未进行系统升级的电脑有效。所以这并不足够，因此文章里还提到了在这里面还存在许多的存在 WSH 注入的点，所以我开始了 WSH 注入的寻找之旅。



我开始寻找目录里的每一个能找到的 VBS 文件，并测试他们是否可以接受参数。最终发现了这个路径：

`C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.17134.48_none_c60426fea249fc02\SyncAppvPublishingServer.vbs`

这个脚本接受一些参数并将参数直接传递至 `powershell.exe` ，参数的传递过程没有经过任何过滤处理！这样我们可以执行任意命令了。至于更深处的原因可以看到该脚本的第 36 行：

`psCmd = "powershell.exe -NonInteractive -WindowStyle Hidden -ExecutionPolicy RemoteSigned -Command &{" & syncCmd & "}"`

可以看到，我们可以直接修改 `syncCmd` ，不仅如此，Edge 浏览器并未对引号进行处理，所以我们可以传递任何想传递的参数给 `WScript.exe`。同时，更方便的是，这个 `power shell` 将会隐藏运行，这使得整个攻击显得更加完美！

然而也有一个问题，就是上面这个特殊的 VBS 文件所在的文件夹的命名取决于用户使用的 Windows 的版本。在我的win10 - 17134版本的电脑上，这个文件夹的命名包括了 `10.0.17134` 。如果你的操作系统不一样，这里的这个文件夹的命名也将不同。至于其他部分的命名规律，几乎没有文档进行相关的阐述。



我在报告中解释，我们所需要的只是Edge中的一个允许我们检测本地文件（而不是读取它们）的漏洞，我找不到这样的漏洞，但假设它随时都会弹出。更重要的是，我们不必逐个字符猜测整个文件夹的名称。在 Windows 中，文件夹带有一个名为`DOS PATH`的速记版本，因此很可能猜测文件夹位置的 DOS 路径版本。

因此相比于尝试猜测如下路径：

`C:\Windows\WinSxS\amd64_microsoft-windows-a..nagement-appvclient_31bf3856ad364e35_10.0.17134.48_none_c60426fea249fc02\SyncAppvPublishingServer.vbs`

我们不如这样去猜测：

`C:\Windows\WinSxS\AMD921~1.48_\SyncAppvPublishingServer.vbs`

因为这两个不同的路径代表的是同一个文件，因此这让我的在报告中的解释更加掷地有声。



最后，再思考一下那个讨厌的提示框怎么办？没有用户会傻到点击 OK 键然后让电脑运行 Windows Script Host。幸运的是，因为提示框出来时默认焦点就在 OK 键上，这意味着用户只需按住 Enter 键，就可以让程序运行起来。

## 最后

最终的 Poc 如下所示：

`<a id="q" href='wshfile:test/../../WinSxS/AMD921~1.48_/SyncAppvPublishingServer.vbs" test test;calc;"'>test</a>`

`<script>
window.onkeydown=e=>{
	window.onkeydown=z={};
	q.click()
}
</script>`
