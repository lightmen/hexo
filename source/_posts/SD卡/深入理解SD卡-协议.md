---
title: 深入理解SD卡:协议
categories: SD卡
tags:
- SD卡协议
- Linux
- kernel

---

# Overview
**深入理解SD卡**系列文章将介绍SD卡，涉及SD卡的协议及驱动代码。我们学习SD卡目的是为了理解SD卡的驱动代码，修改它，最终解决工作中遇到的SD卡相关的问题。本系列文章的目标是理解SD，包括协议和驱动代码。在学习任何设备驱动时，有个东西我们是无法绕过的，那就是_协议_，本文讲的就是SD卡的协议。

学习SD卡协议，可以让我们更好的了解SD卡的运作机制。在最开始学习SD卡的时候，我们只需要对SD卡的协议有个大概了解，能基本满足我们看懂SD卡驱动代码就行。如果之后在阅读SD卡驱动代码有不理解的地方，可以回过头来翻翻SD协议文档。建议在读SD驱动源码和学习SD卡协议之间交替进行，互相验证。

关于SD卡，有个叫SD卡协会的组织，这个组织规定了各种涉及SD卡的协议，并发布协议文档。这些SD卡协议文档，最重要的有两种文档：SD Specifications Part 1 Physical Layer Simplified Specification 和SD Specifications Part A2 SD Host Controller Simplified Specification。

个人理解，定协议的目的就是为了使某个事物标准化，标准化后，可以方便大家协作，简化工作量，提高效率，避免重复工作导致的浪费。Physical Layer Simplified Specification（以下简称：卡协议）规定了SD卡的物理规格和SD卡使用的命令协议，像Sandisk、Kingston这类SD卡制造商必须遵守该协议。假设Sandisk开发了一款SD卡没有遵循该协议，而是自己内部新搞了一套协议，这样市面上就没有设备能使用该款SD卡。除非有人专门开发一个驱动去适配该款SD卡，但是这样很浪费人力。如果每个SD卡厂商都使用自己的协议，那么每支持一款SD卡，都需要重新写一套代码去适配它，那这工作量就很恐怖了。

类似的，SD Host Controller Simplified Specification（以下简称：主机协议）用来标准化SD主机控制器，针对的是SD卡主机控制器厂商。这个协议不是强制的，在我们阅读SD驱动代码的时候，如果涉及到SD卡主机控制的代码，我们可能需要翻一下这篇文档，或者查阅SD卡主机控制器厂商提供给我们的文档（一般都是各大cpu芯片厂商提供给我们开发者文档）。

本文的讲解的是卡协议，接下来，会有一些的英文夹杂在中文里面，因为有些名词，还是原汁原味的好，我翻译出来，没了那种韵味，水平有限，望大家谅解。

# System Features
本大章节讲解SD的一些基本特征，包括SD卡的物理规格、容量、速度等方面。

##Form-factor
目前市面上按物理规格来看，常见的SD卡有三种：

- 标准的SD卡，这种卡比较大，在有些相机或者PC电脑上会使用；
- 第二种是miniSD，这种卡我没怎么使用，不作详述；
- 最后一种是叫TF卡，也称mirco SD，这种卡比较小，是我们最常接触的，像我们的手机里面使用的就是这种卡。很多人基本上都管我们手机使用的那种卡叫SD卡，这样的叫法实际上不够准确，更准确应该是叫TF卡，但是不管怎样，都没人会去计较，能理解就行。

本文中，如果我说SD卡，都是泛指这三类SD卡，除非特意说明。并且如果特指，我会使用标准SD卡或者TF卡等名称代替。

## Capacity of Memory
SD卡按容量(Capacity)分类，可以分为标准容量卡、高容量卡，扩展容量卡，详细如下：
> 1. Standard Capacity SD Memory Card (SDSC): 这种卡容量小于等于2GB 
> 2. High Capacity SD Memory Card (SDHC): 这种卡容量大于2GB，小于等于32GB
> 3. Extended Capacity SD Memory Card (SDXC)：这种卡容量大于32GB， 小于等于2TB

如果你买了一张16G或者32G的SD卡，你会发现SD卡上面印有"HC"字样，代表该卡是SDHC卡，同理，64G的SD卡上面印着"XC"，表示SDXC卡。

## Voltage range
SD卡按供电范围划分，分两种：
> 1. High Voltage SD Memory Card: 操作的电压范围在2.7-3.6V
> 2. UHS-II SD Memory Card： 操作的电压范围VDD1: 2.7-3.6V, VDD2: 1.70-1.95V

UHS-II类型的卡参考协议文档： SD Specifications Part 1 UHS-II Simplified Addendum
 
## Bus Speed Mode (using 4 parallel data lines)
SD卡按总线速度模式来分，有下面几种：
> 1. Default Speed mode: 3.3V供电模式，频率上限25MHz，速度上限 12.5MB/sec
> 2. High Speed mode: 3.3V供电模式，频率上限50MHz，速度上限 25MB/sec
> 3. SDR12： UHS-I卡， 1.8V供电模式，频率上限25MHz，速度上限 12.5MB/sec
> 4. SDR25： UHS-I卡， 1.8V供电模式，频率上限50MHz，速度上限 25MB/sec
> 5. SDR50： UHS-I卡， 1.8V供电模式，频率上限100MHz，速度上限 50MB/sec
> 6. SDR104： UHS-I卡， 1.8V供电模式，频率上限208MHz，速度上限 104MB/sec
> 8. DDR50： UHS-I卡， 1.8V供电模式，频率上限50MHz，性能上限 50MB/sec
> 9. UHS156： UHS-II RCLK Frequency Range 26MHz - 52MHz, up to 1.56Gbps per lane.

SDR(Single Date Rate), 一个周期只能采集一次数据，即一个bit，由于SD卡是4条数据线并行传输，所以一个周期能传输4bit，如果频率是50MHz（即1秒传输次数为50 000 000），那么1秒能传输的数据量为25MB（这里1MB为1 000 000 Byte)。所以这就是为什么各种SDR模式里面，频率上限是速度上限的两倍。而对于DDR(Double Data Rate)，在时钟上升沿和下降沿都可以采集数据，也就是单一周期内可读取或写入2次，因此4条并行数据线在一个周期内能传输8bit。

## Speed Class
SD卡按照读写性能划分，有5种规格，每种规格后面的数字象征最小的读写速度：

- Class 0 - 这种卡没有性能要求
- Class 2 - 要求在 Default Speed mode 下，性能至少要达到（大于等于） 2MB/sec
- Class 4 - 要求在 Default Speed mode 下，性能至少要达到 4MB/sec
- Class 6 - 要求在 Default Speed mode 下，性能至少要达到 6MB/sec
- Class 10 - 要求在 High Speed mode 下，性能至少要达到 10MB/sec

厂商卖的SD卡上面基本上都会印着一个用圆圈包围起来的数字10，表示该卡是Class 10 类型的卡。

#  Bus Protocol
在SD Bus上，有三种transaction：

- **Command**: 一个命令代表着将开始一个操作。命令通过CMD线传输，方向从host到card。
- **Response**: 响应是card对前一次host发送的命令的执行情况的反馈。也是通过CMD线传输，方向从card到host。
- **Data**: 数据是通过4条data线传输的，方向可以从card到host，也可以从host到card。

不管Command，还是Response或者Data，都开始于一个start bit （bit值0），结束于一个end bit（bit值1）。

关于这块的内容不做过多解释了，详情自行阅读"Physical Layer Simplified Specification Version 4.10"文档 "3.6 Bus Protocl" 章节的内容。 

# Registers 
下图是SD卡的体系架构，可以看到内部包含了一系列的寄存器：

<div style="text-align:center" markdown="1">
![SD Memory Card Architecture](http://on61oh42c.bkt.clouddn.com/SD-Memory-Card-Architecture.png)
</div>

各个寄存器的详细信息如下：

![SD Memory Card Registers](http://on61oh42c.bkt.clouddn.com/SD-Memory-Card-Registers.png)

## OCR register
OCR寄存器保存着SD卡的工作电压范围。如果OCR寄存器的某位为1，表示卡支持该位对应的电压。最后一位表示卡上电后的状态(是否处于”忙状态”)，如果该位为0，表示忙，如果为1，表示处于空闲状态。

![OCR Register Definition](http://on61oh42c.bkt.clouddn.com/ocr.png)

## CID register
CID是一个128 bits的寄存器，该寄存器包含一个卡的标识信息。

![The CID Fields](http://on61oh42c.bkt.clouddn.com/cid.png)

## CSD Register
卡的描述数据寄存器（CSD）包含了访问该卡数据时的必要配置信息，比如the data format, error correction type, maximum data access time, device size 等等。

![The CSD Register Fields (CSD Version 2.0)](http://on61oh42c.bkt.clouddn.com/csd.png)

## RCA register
卡的相对地址,该16位卡地址寄存器保存了卡在识别过程中发布的地址。该地址用于在主机识别卡后，利用该地址与卡进行通信。该寄存器只有在SD模式下才有效。

##  SCR register
SD配置寄存器提供SD卡的特殊特性信息，其大小为64位。该寄存器由厂商编程，主机不能对它进行编程。

![The SCR Fields](http://on61oh42c.bkt.clouddn.com/scr.png)

# UHS
UHS（Ultra High Speed）是与SDXC同时推出的SD卡总线标准。此标准适用于SDHC和SDXC。

UHS-I最高传输速度（理论值）为104MB/s。英文字母I代表该设备（SD卡或读卡器）支持UHS-I接口。英文字母U，包含数目字1，代表该设备读写速度达U1。

UHS-II最高传输速度达312MB/s，是UHS-I的三倍。

设备（如智能手机）必须支持UHS，才能保证达到U1或U3最低写入速度。

下面介绍UHS-I初始化的命令序列流程。

![Command Sequence to Use UHS-I](http://on61oh42c.bkt.clouddn.com/Command-Sequence-to-Use-UHS-I.png)

- 上电后，卡会处于3.3V signaling模式下。第一个CMD0命令会选择bus模式：SD模式或者SPI模式。只有在SD模式下，才能进入1.8V signaling模式。一旦卡进入1.8V signal模式，卡不能切换到SPI模式或者3.3V signal模式，除非重新上电。
- 收到CMD0命令后，卡将进入空闲状态（Idle state），但是仍然工作在SDR12时序下。UHS-I只提供了SD模式，没有提供SPI模式。
- 由于更高的总线速度需要低水平的signaling，对SDR50、DDR50和SDR104模式，UHS-I提供的signaling为1.8V。host会给卡提供3.3V的电压，并且提供1.8V signaling水平的电压给SDCLK、CMD和DAT[3:0]线，这几个都是从3.3V的电源线转换过来的。为了避免主机与卡之间的电压不匹配，signaling水平在初始化时的电压转换序列中就已经被改变了。主机和卡通过ACMD41命令来确认双方是否支持1.8V signaling模式。如果主机和卡都支持1.8V signaling模式，这就意味着UHS-I卡可用。
- UHS-I只能使用4-bit的bus模式，CMD42是个例外。如果卡被锁住了，就需要通过发送CMD42命令（1-bit模式）解锁，然后发送ACMD6命令将bus模式切换到4-bit。
- 在卡解锁的情况下，CMD19命令执行在1.8V signaling的传输状态。其他情况，CMD19都会被当做非法命令。


# SD Memory Card Functional Description
对SD卡与主机(host)来说，有两种操作模式：

- Card identification mode: 对卡reset重置后，主机进入卡识别模式，对卡来说，在reset后，除非收到CMD3命令，否则卡一直处于该模式下。
- Data transfer mode: 当卡第一次发布它的RCA后，该卡将处于数据传输模式。而对主机来说，在它识别了bus线上的所有卡后，进入该模式。

## Card identification mode
### Operating Condition Validation
SD卡识别模式流程图如下：

![SD Memory Card State Diagram (card identification mode)](http://on61oh42c.bkt.clouddn.com/SD-Memory-Card-State-Diagram_card-identification-mode.png)


1. 在主机与卡通信之前，主机不清楚卡支持的电压范围，并且卡也不知道是否支持主机提供的供电电压。主机会以默认电压发送一个reset指令(CMD0),并且主机默认卡能支持该命令。然后，为了确认电压，主机接下来会发送一个CMD8命令。

2. 为了验证SD卡接口的操作条件，主机通过发送SEND\_IF\_COND (CMD8)命令，去获取SD卡支持的工作电压范围。SD卡通过检测CMD8的参数部分来检查主机使用的工作电压，主机通过分析卡CMD8的response参数来确认SD卡是否可以在所给电压下工作，如果SD卡可以在指定电压下工作，则它的response里面会包含cmd8参数里面提供的电压 。如果不支持所给电压，则SD卡不会给出任何响应信息，并继续处于IDLE状态。如果要初始化SDHC和SDXC，在第一次发送ACMD41命令前，必须先发送CMD8。

3. SD\_SEND\_OP\_COND (ACMD41)命令来识别或者拒绝不匹配host主机供电电压范围的卡。如果SD卡在主机规定的电压范围内不能实现数据传输，卡将放弃下一步的总线操作而进入不活动状态(Inactive State)。

4. 主机发送ACMD41命令时，可以通过将该命令所带的OCR参数设置为0，用来查询卡支持的工作电压范围。当ACMD41被用于查询时，卡将忽略掉ACMD41里面的HCS参数。主机在查询到卡的工作电压后，也许会将该电压作为接下来发送的ACMD41命令的参数。

5. 在整个初始化过程中，主机不允许改变正在操作的电压范围。

### Card Initialization and Identification Process


![Card Initialization and Identification Flow (SD mode)](http://on61oh42c.bkt.clouddn.com/Card-Initialization-and-Identification-Flow.png)

1. 当总线被激合后，主机就开始处理卡的初始化和识别。在主机发送SD_SEND_OP_COND(ACMD41)命令开始处理SD卡初始化时，主机会在ACMD41的参数中设置它的操作条件和设置OCR中的HCS位。HCS位被设置为1表示主机支持SDHC或者SDXC。HCS被设置为0表示主机不支持SDHC和SDXC。

2. 卡利用OCR里面的busy位来通知主机ACMD41的初始化已经完成。如果busy位为0，表示卡还在初始化，如果busy位为1，说明初始化已经完成。主机会在1s的时间内，重复不断地发送ACMD41命令，直到busy位被置1为止。卡只有在第一次收到设置电压的ACMD41命令时，才会去检查操作条件和OCR中的HCS位。并且在重复发送ACMD41命令的这段时间里，主机不应该发送任何命令，除了CMD0。

3. 如果卡能正确响应CMD8，之后，卡对ACMD41命令的响应会包含一个CCS字段，CCS在卡返回ready时（busy位置1）有效。CCS=0表示卡是SDSC，CCS=1表示卡是SDHC或者SDXC。

4. 在ACMD41之后，主机会发送 ALL\_SEND\_CID  (CMD2)，获取卡的CID。在卡发送它的CID之后，卡进入识别状态（Identification State）。

5. 接着，主机发送CMD3 (SEND\_RELATIVE\_ADDR)，请求卡发布卡的RCA。RCA是一个比CID短的，并且将来在数据传输模式中使用的地址。

## Data Transfer Mode

因为一些卡在识别模式（Identification Mode）下，对操作频率有限制，所以在识别模式结束前，主机的频率需要一直保持在 fOD。在数据传输模式（Data Transfer Mode），主机频率在fpp范围内是可执行的。

主机必须发送SEND\_CSD(CMD9)来获得卡规格数据寄存器(CSD)内容，获取像块大小、卡容量这类信息。

SET\_DSR(CMD4)广播命令配置所有识别到的卡的驱动阶段。它对DSR寄存器进行编程以适应应用的总线布局（长度）、总线上卡的数目和数据传输频率。clock rate也是在这个时候从fOD切到fpp。对卡和主机来说，SET\_DSR(CMD4)命令是个可选。

CMD7用于选择卡，并且将卡带入传输状态(Transfer State)。在同一个时间内，只有一张卡能进入传输状态。当发送的CMD7的RCA地址参数为"0x0000"，所有卡将跳回到准备状态（Stand-by State ）。

SD卡数据传输模式的流程图如下：

![ SD Memory Card State Diagram (data transfer mode)](http://on61oh42c.bkt.clouddn.com/SD-Memory-Card-State-Diagram_data-transfer-mode.png)

对已经拥有RCA的卡来说，对它发送identification commands（比如ACMD41、CMD2），它将不会有任何回应。在数据传输模式下，主机与被选中的卡（使用定向命令）之间的数据传输都是点对点的。通过cmd线，所有定向命令（addressed commands）都会收到一个用于确认的response。

下面是数据传输模式下关于数据传输的一些总结：

* 在任何时候，所有的读命令集在执行过程中都可以被stop command (CMD12)打断。cmd12命令将会使数据传输终止，并且使卡退回到传输状态（Transfer State）。读命令集包括：block read(CMD17), multiple  block read(CMD18), send write protect(CMD30), send SCR(ACMD51) 和 general command in read mode (CMD56)。
* 在任何时候，所有的写命令集在执行过程中都可以被stop command (CMD12)打断。写命令集包括： block write(CMD24 and CMD25), program CSD(CMD27), lock/unlock command(CMD42)和general command in write mode(CMD56)。
* 一旦数据传输完成，卡就会退出数据写状态，并且进入正在编程状态（Programming State）（传输成功），或者进入传输状态（传输失败）。
* 如果一个块写操作被打断，但是最后一个block的块长度和CRC有效的话，这块数据也将会被编程到卡里。
* 卡也许会对块写操作提供缓存，这意味着，在一个block还在被编程的情况下，下一个block可以被发送这个卡里面。如果所有的写缓存都已经满了的话，只要卡还在正在编程状态，DAT0线就会一直保持在拉低状态。
* 对写CSD、写保护和擦除操作来说，卡不会提供缓存。这意味着，在卡正忙于处理这其中任何一个命令时，卡不会接收任何发送到卡的数据。只要卡还在忙，DAT0线就会拉低，并且处于正在编程状态（Programming State）。
* 当卡正在编程时，不允许任何一个参数设置命令集（Parameter set commands）。参数设置命令集包括： set block length(CMD16), erase block start(CMD32)和erase block end(CMD33)。
* 当卡正在编程时，不允许任何一个读命令集。
* 当将其他的卡从准备状态（Stand-by）切换到传输状态（使用CMD7），不会中断当前卡的擦除或者编程操作。当前卡将会切换到断开状态（Disconnect State），并且释放数据线。
* 当卡正在编程或者待编程时，对其重置（发送CMD0或者CMD15），将会导致操作终止，并且可能会导致卡内的数据内容被破坏。因此主机有责任去禁止这样的操作。


至此，本文关于SD卡协议的内容就介绍到这里。通过本文，可以对SD卡有个大概的了解，尤其是关于SD卡初始化这段内容。就SD卡协议这方面来说，了解一些基本的东西就行。在我们今后在遇到SD问题时需要时，可以翻出来看一下。水平有限，有些地方可能会出现错误，望各位能指出来，希望能和各位共同探讨技术方面的内容。

