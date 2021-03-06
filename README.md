# 戴尔9020M黑苹果EFI配置
## 本EFI基于[li3p](https://github.com/li3p/dell-optiplex-9020-hackintosh-opencore)的基础上对以下关键配置进行了适配。

当你看到这段文字时，下面提到的问题均已在EFI内解决。

***
+ CPU:i7-4750HQ
+ WIFI/蓝牙:DW1830
***
### 目前已经完成的功能
+ 睡眠（支持蓝牙唤醒，在此模式下整机功耗5-10W，苹果桌面电脑默认优先为此模式）
+ 休眠（不支持蓝牙唤醒，在此模式下整机功耗0.8W（与关机一致），需要手动点击苹果logo-Sleep来进入此模式，此模式下唤醒后隔空投送会失效，如果长时间不用建议关机。）
+ Airdrop与Handoff
+ 正常变频
+ 开机音效
+ 正确核显适配
+ USB正确映射
***
关于核心显卡性能的一些更新：
> 如果你需要使用4K显示器，请在DeviceProperties-PciRoot(0x0)/Pci(0x2,0x0)内添加:framebuffer-unifiedmem 00000080。

> 稳定性与性能问题：
~~目前使用的ig-platform-id为0300220D，即桌面处理器的framebuffer，Geekbench跑分在3400-3600左右.
如果更改为0600260A，即笔记本的framebuffer，geekbench跑分可以达到4000-4400左右.~~

如果出现核心显卡黑屏，闪烁，性能结果浮动过大（正常基本在4450左右浮动），优先检查供电的问题。我这一台使用卖家提供的适配器就会出现这些问题，使用可调电源测试后问题得到一定的改善。

目前p95单烤CPU功耗在37W左右，整机58W，单烤cpu没有出现稳定性问题。heaven单烤GPU功耗峰值会去到31W，此时电脑可能会出现黑屏，闪屏的情况，最后核心稳定在25W左右，整机42-44W。

> 更改为笔记本framebuffer出现的问题:
~~开机时屏幕会短时间花屏.
运行heaven进行压力测试时 可以看到核心显卡频率短暂达到最高调度：即1.15Ghz,但是屏幕会黑屏闪烁.~~

好像找到了一个性能和稳定性并存的配置方法
具体操作:
- 在DeviceProperties-PciRoot(0x0)/Pci(0x2,0x0)，修改Device-id为12040000,AAPL,ig-platform-id不需要更改，仍待保持0300220D.
***
## 相同配置使用本EFI之前要求的操作
+ 更新到最新的BIOS版本
+ 进入BIOS恢复默认配置后，设置为UEFI引导，打开Legacy启动兼容，关闭网卡的PXE引导
+ 使用modGRUBShell更改隐藏的BIOS配置
#### !!!每个电脑的bios变量可能会不一致，下面的配置只针对9020M，请不要轻易在其他系统上使用相同命令!!!
> 关闭CFG LOCK ：setup_var 0xD9F 0x0

> 设置DVMT为64M ：setup_var 0x263 0x2

> 启动EHCI hand-off：setup_var 0x2 0x1

> 设置XHCI模式：setup_var 0x144 0x1

> 重新配置EHCx接口到XHCI：setup_var 0x15A 0x2，setup_var 0x146 0x0，setup_var 0x147 0x0

+ 完成后输入reboot重启，正常安装macos后,替换为本EFI，配置platformInfo。机型为Macmini7,1
***
### 目前发现的问题
> 如果你使用的是黑果小兵的安装引导，可能会出现无法进入安装界面的情况，可以将Opencore更新到最新版本。并将此EFI内的ACPI文件替换到引导盘里。

> 一些USB键盘会将电脑从休眠唤醒

> 如果是使用其他型号的CPU，你需要重新配置SSDT-EC.aml和SSDT-PLUG.aml，CPUFriendDataProvider，来获得最佳的变频性能。
>>相关教程https://dortania.github.io/OpenCore-Post-Install/universal/pm.html#enabling-x86platformplugin
>>如果不进行变频配置 睡眠可能会出现问题。

> 从休眠唤醒后， “隔空投送”频道会被限制到6，此时隔空投送失效，可以通过重新开关WIFI和蓝牙来暂时解决。

>~~（偶尔）开机时会出现输入设备反应迟钝的问题。本来以为是USBmap的问题，配置之后问题依旧，但是出现次数并不多，而且不是一定出现。~~
已经解决。启动项内加入agdpmod=vit9696后开机输入设备卡顿的问题消失。
