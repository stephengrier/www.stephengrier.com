---
layout: post
title: Finding the right hardware for an OpenWrt Wi-Fi router
categories: [OpenWrt, Wifi6]
---

I recently had my old VDSL home internet connection upgraded to shiny new
fibre-to-the-premises (FTTP). As part of the upgrade my ISP sent me a new [Eero
6 Wi-fi router](https://eero.com/shop/eero-6) to replace my old [Huawei
DG8041W](https://rhyslolly.com/2020/12/12/talktalk-wifi-hub-black-huawei-dg8041w-review-test-and-tear-down/).
On paper the Eero 6 is a nice device, with Wi-fi6, a 1.2GHz quad core CPU and
512MB of RAM - quite impressive specs for a home router. The Eero is a wireless
mesh device, providing the possibility of adding extra Eero nodes to provide
blanket wi-fi coverage for your home.

Despite the decent specs I was left very unsatisfied with the device. The Eeros are
clearly designed to be easy for novices to install and manage through a mobile
app available for Android and iOS smartphones. But for power users the Eero 6
just does not provide enough configurability; for example you can name your
SSIDs to whatever you want but there's no way to give your 2.4GHz and 5GHz
radios different SSIDs if you wanted to. There is no SSH or even a web
interface, everything is configured from the mobile app, which I found massively
frustrating. And it just didn't give me the ability to see what was going on at
the heart of my home network.

I decided it was time to look for a new wi-fi router that met my needs. I knew
that I wanted:

* the ability to see what was happening in my home network; to be able to see
  what connections were being made, perform packet dumps, view system logs
* firmware that was relatively new and not full of security holes; preferably
  based on open source software that I could keep up-to-date myself
* something relatively powerful, preferably a recent dual or quad core ARM-based
  CPU with a decent clock speed and at least 512MB of RAM
* low power - drawing no more than 5-7 Watts when idle - both to keep the cost
  of running it low and to make sure it didn't run too hot and burn by house
  down
* something cheap, preferably under £100
* the ability to route packets at up to gigabit. Although I haven't gone
  for a 1Gb internet connection yet, I'd still like a router that is capable of
  routing at that speed if I choose to upgrade at some point in the future
* preferably capable of offloading routing and switching to hardware to keep the
  CPU free for other tasks and to allow me to scale the CPU down to save power
  and keep the system cooler
* Wi-Fi6 would be nice but certainly not essential

Essentially I wanted a bit of a project to keep me occupied over the Christmas
break. Having watched a load of YouTube videos and blog posts I decided I wanted
to build a router that I could run OpenWrt on.

# Choosing the hardware

I knew I wanted something cheap and low power. [OpenWrt](https://openwrt.org/)
is a Linux distribution designed to run on low-powered embedded devices. It will
run on as little as 32MB of RAM and just 4MB of flash storage and has support
for [a plethora of architectures](https://openwrt.org/docs/techref/targets).
This gave me a lot of hardware options.

My first thought was whether I could install OpenWrt on my [Huawei
DG8041W](https://rhyslolly.com/2020/12/12/talktalk-wifi-hub-black-huawei-dg8041w-review-test-and-tear-down/).
The Huawei has excellent Wi-fi coverage with 7 antenna - 3x3 for the 2.4GHz band
and 3x3 for the 5GHz band, so I thought it might be a cheap option for an
OpenWrt router. This plan was dashed very quickly when I found that the Huawei
is powered by a Triductor chipset, which [is not supported by OpenWrt and is
very unlikely to ever
be](https://forum.openwrt.org/t/adding-openwrt-support-for-huawei-dg8041w-2-t5-new-2020-talktalk-wifi-hub-black/91364).

Next, I watched Wolfgang's great [YouTube video about building a home router
from a Fujitsu S920 thin client](https://www.youtube.com/watch?v=uAxe2pAUY50). Although
they're quite old now, the S920s have pretty decent hardware with a quad core
amd64 processor and 2 or 4GB of DDR3 RAM. This is easily capable of running an
OpenWrt or pfSense router and would make a nice system. However, it would need
a separate Wi-fi adaptor to make it an access point and the lack of antennas
would limit its usefulness here. I also couldn't find one with a UK power
adaptor for much under £70 quid on ebay and there were other potentially better
options at that price point.

<iframe width="560" height="315" src="https://www.youtube.com/embed/uAxe2pAUY50"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
allowfullscreen></iframe>

I considered the [NanoPi R6S from
FriendlyElec](https://www.friendlyelec.com/index.php?route=product/product&product_id=289).
The R6S is tiny single board computer, sporting the Rockchip RK3588S SoC, two
ARM based Cortex quad-core CPUs, 8GB of RAM and three ethernet ports, including
two 2.5GbE ports and a third 1GbE port, all for about $120. One of these would
make a really nice little router. However, there were a couple of things that
put me off. Firstly, the R6S does not come with a Wi-fi controller so I'd either
need to buy a USB Wi-Fi adapter and use one of the device's USB3 ports, or buy
a separate access point. A USB Wi-Fi adapter would not make a great access point
as it wouldn't have enough antennas to provide the coverage I needed. Buying a
separate access point would complicate my setup and was not something I wanted
to do at this point.

<img src="https://www.friendlyelec.com/image/cache/catalog/details/R6S_01-900x630.jpg" />

The bigger problem, though, is that the NanoPi R6S is not supported by
mainstream OpenWrt. Although it claims to come with OpenWrt installed it
actually runs a forked version of OpenWrt called
[FriendlyWrt](https://github.com/friendlyarm/friendlywrt). For me this makes the
NanoPi interesting as a development device but I wouldn't trust running one of
these as an internet-facing router at the heart of my home network.

I then came across [GL.iNet](https://www.gl-inet.com/). GL.iNet are a Hong Kong
based manufacturer specialising in small form factor travel routers running a
forked version of OpenWrt. In particular, their
[Flint](https://www.gl-inet.com/products/gl-ax1800/) device, which is a handsome
looking home wireless router supporting Wi-Fi 6, a quad-core Qualcomm IPQ6000
CPU, 512MB of DDR3 RAM, a USB3 port and 4 external wireless antennas. Despite
being a lovely bit of kit I eventually discounted it for the same reason as the
NanoPi R6S - it doesn't run stock OpenWrt but a heavily modified fork. There is
[lots of discussion on the OpenWrt
forums](https://forum.openwrt.org/t/gl-inet-ax1800-new-router-openwrt-support/105163)
about getting OpenWrt running on the Flint but until it is fully supported it's
not an option for me.

<iframe width="560" height="315" src="https://www.youtube.com/embed/fcdujvltleU"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
allowfullscreen></iframe>

I considered building my own router from a [Raspberry
Pi](https://www.raspberrypi.com/). I watched [Dev Odyssey's excellent video
about building an OpenWrt router from a Raspberry Pi Compute Module 4 and a
DFRobot Carrier Board](https://www.youtube.com/watch?v=o2NTvaRv4Yg). I must say
this really appeals to me and I'd love to make one of these, possibly just as
an extra Wi-Fi access point. However, there's one massive problem with the
Raspberry Pi - as of December 2022 you can't get one for love-nor-money. The firm
[has faced dire supply chain issues and huge
demand](https://www.raspberrypi.com/news/production-and-supply-chain-update/)
and the result is they're not currently available from any of their approved
resellers anywhere. In [a recent
announcement](https://www.raspberrypi.com/news/supply-chain-update-its-good-news/)
it appears availability may improve throughout 2023 and if that is the case this
might yet make a cool future project.

<iframe width="560" height="315" src="https://www.youtube.com/embed/o2NTvaRv4Yg"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
allowfullscreen></iframe>

At this point it became apparent to me that if I wanted a reliable OpenWrt
device I would need to start by checking the OpenWrt forums for recommended
devices. When you read the OpenWrt forums a number of devices are clearly very
popular and come up often.

One such device is [the TP-Link Archer
C7](https://openwrt.org/toh/tp-link/archer_c7). The Archer C7 is said to be
rock-solid when running OpenWrt due to it's Qualcomm Atheros QCA9558 and QCA9880
wireless chipsets, which are well supported in OpenWrt. You can pick up a second
hand C7 on ebay for £30-40, so this would be a low risk option. However, the
hardware is quite old now - it was released in 2013, which makes it nearly 10
years old. It is also a little under-powered for my liking - with a single core
720MHz CPU, 128MB of RAM and just 8MB of flash storage, enough to comfortably
run OpenWrt but a little under-powered for me nonetheless and it won't route
packets at gigabit speeds. But the C7 is still a great option as a cheap OpenWrt
access point if that's what you're looking for.

Having watched [OneMarcFifty's excellent video about OpenWrt hardware
choices](https://www.youtube.com/watch?v=wP1ZcQBLL1k) I then considered the
[D-Link
DIR-2660](https://www.dlink.com/en/products/dir-2660-exo-ac2600-smart-mesh-wi-fi-router).
The DIR-2660 is well supported by OpenWrt due to its use of MediaTek SoC and
wireless chipsets. It is also a relatively powerful device for a Wi-Fi router
with a dual core MediaTek MT7621AT 880MHz CPU and 256MB of RAM. And the MT7621
SoC supports hardware flow offloading, handing off the work of routing packets
to its network processing unit (NPU) to keep its CPU free for other tasks.
However, there are [reports of stability issues with OpenWrt and the MT7615
wireless
chipset](https://forum.openwrt.org/t/openwrt-on-mt7621-mt7615n-devices-with-5ghz-problems/107392).
Not everyone with the chipset seems to be experiencing the reported problems,
but I really don't want to be struggling with stability issues with my home
Wi-Fi router, so I decided to steer clear of any devices with the MT7621/MT7615
combination.

<img src="https://www.dlink.com/-/media/global-product-images/consumer/home-networking/routers/dir-2660/product-preview/dir-2660-4.png" />

Another hardware option that gets a lot of mentions on the OpenWrt forums is the
[Netgear R7800 (aka Nighthawk X4S)](https://openwrt.org/toh/netgear/r7800). The
R7800 is a very powerful Wi-Fi router with a dual core ARM Cortex-A15 1.7GHz CPU
and 512MB of RAM, 2 USB3 ports, and 4 detachable antennas. The device uses the
Qualcomm Atheros IPQ8065 SoC and the Qualcomm Atheros QCA9984 wireless chipset,
which, while not the best in terms of open source driver support, does have
excellent, stable support in OpenWrt.

It is possible to pick up an R7800 second hand on ebay for £70-80, so would make
an affordable and relatively powerful OpenWrt home router. This ended up being a
serious contender for me and I almost decided to get one of these over the
Belkin RT3200, but in the end there were two reasons why I did not go this
route. Firstly the IPQ8065 does not support hardware flow offloading (at least
not in OpenWrt), and although the dual core CPU is more than capable of routing
packets up to gigabit speeds the lack of flow offloading will increase the power
consumption and the running temperature, both of which are an issue for me.
Secondly, the R7800's power consumption [is reported to be ~ 7-10 watts at
idle](https://www.missingremote.com/review/2016/03/netgear-nighthawk-x4s-r7800-ac2600-wireless-router),
good but almost double the 5 watts of the Belkin RT3200.

# The Belkin RT3200

The [Belkin RT3200](https://openwrt.org/toh/linksys/e8450) is another device
that is getting a lot of mentions on the OpenWrt forums. It is pretty much the
only Wi-fi 6 capable device that is currently well supported by OpenWrt and as a
result is getting a lot of attention from OpenWrt enthusiasts. One example of
this is [OneMarcFifty's excellent YouTube video on OpenWrt and the
RT3200](https://www.youtube.com/watch?v=YyGVudeiEmI).

<iframe width="560" height="315" src="https://www.youtube.com/embed/YyGVudeiEmI"
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay;
clipboard-write; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen></iframe>

The RT3200, which is also available as the Linksys E8450, has excellent support
from OpenWrt due to its use of the MediaTek MT7622BV SoC and the MediaTek
MT7915E Wi-Fi 6 chipset. Although the Mediatek chipsets have proprietary
firmware on them, Mediatek do [actively contribute to the development of open
source drivers for their
devices](https://patchwork.kernel.org/project/linux-mediatek/list/?submitter=171503),
so open source support for them is generally good. There is also [fairly active
development of the mt76 drivers](https://github.com/openwrt/mt76/commits),
particularly around newer chipsets like the mt7915.

OpenWrt is [reported to be stable on the
RT3200](https://forum.openwrt.org/t/belkin-rt3200-linksys-e8450-wifi-ax-discussion/94302)
and there is good [OpenWrt documentation for the
device](https://openwrt.org/toh/linksys/e8450), so lots of folks have trodden
this path already and there's good guidance on how to get the most out of the
device under OpenWrt.

The RT3200 is a relatively powerful device for a home router, with a dual-core
ARMv8 CPU clocked at 1.35GHz, 512MB of DDR3 RAM, 5 Gigabit ethernet ports and
128MB of NAND flash storage. There's plenty of power here to meet my needs and
enough to run ad blocking, openvpn or QoS should I ever want any of those.

The mt7622/mt7916 hardware combination is fairly new and therefore power
efficient. The Cortex-A53 CPU can be frequency scaled down to make the system
even more power efficient when under lower loads, as well as running cooler. The
RT3200 is [reported to consume just 5 watts at
idle](https://forum.openwrt.org/t/belkin-rt3200-linksys-e8450-wifi-ax-discussion/94302/20),
which is great for a device that will be running 24/7.

The RT3200 will happily route/NAT and switch packets at Gigabit speeds. Not only
that, the mt7622 SoC supports hardware flow offloading, so it will route and
switch packets at Gigabit speeds with close to zero CPU usage! This is a great
feature as it leaves the CPU free for other purposes and allows it to be scaled
down to save power and keep the system cooler. Awesome! At the time of writing
[the mt762X devices are the only devices with support for this in
OpenWrt](https://forum.openwrt.org/t/how-to-judge-whether-the-device-supports-hardware-flow-offloading/121930/2).

The RT3200 is an 802.11ax capable Wi-Fi device, which means it supports the
latest Wi-Fi 6 standard. This was not initially a bit deal for me as I only have
one AX capable client device and a good AC capable device would have met my
needs. However, the RT3200 is still capable of supporting 802.11ac on the 5GHz
band and b/g/n on the 2.4GHz band, so 802.11ax is a nice added bonus.

The Belkin RT3200 is also very affordable. Although I initially intended to pick
up a second hand device to use as an OpenWrt router, the RT3200 is available
brand new for £75, less than older models like the Netgear R7800 or the D-Link
DIR-2660 go for second hand on Ebay.

![](/images/{{ page.id }}/rt3200-in-box.jpg)

## Cons

As with everything there are downsides with the Belkin RT3200. It is not a
perfect system.

Firstly, the RT3200 only supports 802.11ax on the 5GHz radio; the 2.4GHz radio
only supports 802.11b/g/n. Not a big deal for me, but the 802.11ax standard does
apply to both wavebands so may be something to bare in mind if you are someone
who wants the latest Wi-Fi standard.

Secondly, the RT3200 only has a single USB-2 port, no USB-3 port here. Again,
this is not a big deal for me, but this might be an issue if you want to use the
device as, say, a NAS file server.

Third, the RT3200 has only internal antennas. Other devices like the Netgear
R7800 have external antennas for maximum Wi-Fi range, and the lack of these on
the RT3200 may limit the coverage a bit. However, I intend to use the Eero 6 my
ISP gave me in bridge mode as a Wi-Fi access point upstairs, so I won't need the
RT3200 to reach all corners of my home.

# Final thoughts

Having considered all the factors above, the RT3200 ticked all the boxes for me.
It is a very affordable, power efficient, powerful system supporting 802.11ax
that is known to run OpenWrt with good stability.

I've taken the plunge and ordered one. I intend to flash the latest version of
OpenWrt on it and do some testing. All being well I'll be posting again with an
update on how it went.

