# USBMap
<br>Py脚本，用于映射USB端口并创建自定义SSDT或injector kext(WIP).

***

# Installation

<br/>要安装，请在终端中一次执行以下一行操作:

    git clone https://github.com/corpnewt/USBMap
    cd USBMap
    chmod +x USBMap.command
    
<br/>然后使用 `./USBMap.command` 或双击 *USBMap.command*

***

# PreRequisites

<br/>控制器必须命名如下:

* _EHC1 -> EH01_
* _EHC2 -> EH02_
* _XHCI/XHC1 -> XHC__

- 要开始端口检测，您需要有USBInjectAll.kext（可能还有XHCI_unsupported.kext），或者为您的操作系统版本使用端口限制增加补丁，或者需要在扫描中检测端口（首先禁用所有  SSxx  端口，然后禁用所有  HSxx  端口）。<br/>后者是如何使该自述文件集中，因为它允许在不存在端口限制补丁的OS版本上进行映射。

***

# Some Practices To Consider

- 除非我们使用端口限制补丁，否则每个控制器只能有15个USB端口。<br/>当检测到由脚本填充的端口时，它们将根据其控制器和显示在窗口底部的计数进行计数。务实地对待这一点很重要。一些逻辑解决方案包括:

* 如果将键盘和鼠标插入USB3.0端口，则它们可能只使用与其关联的USB2.0端口，因此可以省略每个端口的USB3.0变体。<br/>
* 将设备插入每个端口实现跟踪可用的端口,并且只启用所需的设备。

在我的 Maximus X Code 代码中，我有 HS01 -> HS14, USR1 -> USR2, 和 SS01 -> SS10 这是26个总端口。<br/>
我通常只将鼠标、键盘和DAC插入机器背面的端口，然后在机箱正面的4个端口上进行热交换。<br/>
为此，我禁用了主板背面的所有 _all_ USB 3.0端口（因为我的键盘、鼠标和DAC都是USB2.0），并禁用了所有不支持的usb2.0端口。<br/>
这使我的总端口数保持在11个左右，远远低于15个端口的限制.

***

# Using The Script

在满足上述先决条件后，端口检测的第一步是禁用 _SSxx_ 端口，以确保我们不超过每个控制器15个端口的限制。<br/>
此脚本将搜索 _EH01_, _EH02_, _HUB1_, _HUB2_, 和 _XHC_ 以查找有效端口，并尝试相应地映射它们。<br/>
这都是通过 scraping/parsing ioreg中和system_profiler并确定哪些端口可用并已填充来完成的。<br/>
所有发现的端口都附加到 USBMap.command 所在目录下的usb.plist中。<br/>
这使我们能编译设置，重启，编译更多设置，以及重启插拔更多的设备。<br/>

要禁用_SSxx ports_，我们在脚本的main主菜单中选择 `S. Exclude SSxx Ports` ，这将在 nvram引导的 `boot-args` 添加 `-uia_exclude_ss`（这不会覆盖nvram引导的任何 non-uia 启动参数，也不会影响 config.plist 中的任何启动参数）。<br/>

在设置了那个参数之后，我们重新启动。再次进入桌面时，打开脚本，选择 `D. Discover Ports` - 这将启动5秒的检测循环。<br/>
在循环的每次迭代中，脚本都会检查所有新设备的可见端口。<br/>
在这段时间内，您将需要一个USB2.0设备，并将其插入您计划与2.0端口一起使用的每个USB端口 - 然后等待下一个检测循环反映更改，然后继续。<br/>
当你插入所有你想使用的端口后，你就会映射出你的USB2.0端口。按 `q` ，然后按「enter」回车，脚本将提示您退出检查模式。<br/>

下一步是禁用HSxx端口，并使用USB 3.0设备重复此过程。<br/>
我们通过从主菜单中选择 `H. Exclude HSxx Ports` 来完成此操作。<br/>
这将提示我们包括我们的鼠标和键盘，因为它们通常是USB 2.0 - 如果他们的端口被排除在外，将是不可用的。<br/>
在切换端口以便只选择您的键盘和鼠标后，您可以选择 `C. Confirm` 以删除nvram中以前的任何uia引导参数，<br/>并添加 `-uia_exclude-hs uia_include=HSxx,HSxx` ，其中两个 _HSxx_ 值对应于鼠标和键盘端口。



设置之后，我们再次重新启动，并像对上面的 _HSxx_ 端口所做的那样完成发现过程。<br/>
当您已映射出要使用的所有端口后，可以按 `q` 并按回车键以离开发现。<br/>
然后，我们要选择 `P. Edit Plist & Create SSDT/Kext` ，这将带我们进入一个菜单，<br/>在其中我们可以 enable/disable 端口（基于上面的发现），并更改端口类型。<br/>
可用端口类型如下：


```
0: Type A connector
1: Mini-AB connector
2: ExpressCard
3: USB 3 Standard-A connector
4: USB 3 Standard-B connector
5: USB 3 Micro-B connector
6: USB 3 Micro-AB connector
7: USB 3 Power-B connector
8: Type C connector - USB2-only
9: Type C connector - USB2 and SS with Switch
10: Type C connector - USB2 and SS without Switch
11 - 254: Reserved
255: Proprietary connector
```
With common types falling into 3 categories:
```
0: USB 2.0
3: USB 3.0
255: Internal Header
```


在列表的底部是一个计数器，如果您超过15端口的限制，它会让您知道 - 这不需要多个控制器，所以请考虑这一点。
当按照需要设置列表时，可以选择 `K. Build USBMap.kext` 或 `S. Build SSDT-UIAC` 。<br/>
前者将试图建立一个 _USBMap.kext_ 注入（它不需要 _USBInjectAll.kext_ ）<br/>

- 这将 _only 添加不显示 _USBInjectAll.kext_ 的 ports_，<br/> 但是如果你的系统像我的华硕 Maximus X 代码（它已经检测到了 _HS01-HS14_ 和 _SS01-SS10_ ，而不是 _USBInjectAll.kext_ ），<br/>
你就已经在15以上了。端口限制和添加更多端口将没有好处。<br/>
创建SSDT通常是大多数用户想要做的，因为它只选择希望系统看到的端口。<br/>
这意味着它可以根据需要添加和删除端口。<br/>
不过，SSDT _does_ 要求存在 _USBInjectAll.kext_ .<br/>
