---
layout: post
title: "How to Connect to a Cisco Router Configuration Interface From a (Modern) Macintosh"
date: 2011-10-12 18:14
comments: true
categories: [Cisco, Network management, Apple, Mac OS, Lion, Homebrew]
---

Cisco routers configuration interface is generally available using the well known RJ45-to-serial blue cable, that you naturally plug to your computer serial port. On modern systems, especially Macintosh computers, the serial port has been long forgotten, leaving you with only USB, FireWire, sometimes eSata and now Thunderbolt ports.

However this can be easily solved with a simple adapter, a little time and some additional software.

<!--more-->

# USB-to-Serial adaptor

So the first step is to buy a USB-to-Serial adaptor that you will connect to serial end the Cisco blue management cable. I recommend that you opt for an adapter which uses the **FTDI** or the **Prolific** driver. Both are well maintained and have kernel extensions compatible with latest Mac OS X versions.

I myself bought the [TrendNet TU-S9](http://www.amazon.com/TRENDnet-Serial-Converter-TU-S9-Blue/dp/B0007T27H8/ref=sr_1_1?s=electronics&ie=UTF8&qid=1318438432&sr=1-1) as it is cheap, from a well known brand... and blue! Unfortunately, the installation media did not contain the latest version of the driver, required to perform with Mac OS X Lion. Nevertheless, it can be easily downloaded [here](http://www.prolific.com.tw/support/files//IO%20Cable/PL-2303/Drivers%20-%20Generic/MacOS/MacOS%2010.x/md_PL2303_MacOSX10.6_dmg_v1.4.0.zip). This
version is working with 10.6 (Snow Leopard) as well as 10.7 (Lion).

Simply extract the archive, mount the disk image and install the contained package. When this is done restart your computer.

Fire up a **Terminal.app** and plug the adapter to one of your USB ports. Check that the driver is well loaded by observing the output of the `dmesg` command:

```
Guillaume@friis:~> sudo dmesg | grep "PL-2303"
PL-2303/X V1.3.0 start, Prolific
```

Replace `PL_2303` by `FTDI` if your adapter uses the **FTDI** driver.

We will also need to know the name of the associated `tty` interface, in my case, this is `/dev/tty.usbserial`:

```
Guillaume@friis:~> ll /dev/tty.*
crw-rw-rw-  1 root  wheel   18,   2 Oct 10 07:54 /dev/tty.Bluetooth-Modem
crw-rw-rw-  1 root  wheel   18,   0 Oct 10 07:54 /dev/tty.Bluetooth-PDA-Sync
crw-rw-rw-  1 root  wheel   18,   4 Oct 10 07:54 /dev/tty.GuillaumesiPhone-Wirele
crw-rw-rw-  1 root  wheel   18,   6 Oct 12 19:24 /dev/tty.usbserial
```

# Minicom

Now that the adapter is working, we need to install extra software to access the routers console through the serial configuration interface. The easiest to install and to use is **Minicom**.

## Installation

If you use [Homebrew](http://mxcl.github.com/homebrew/), this is simply a command away:

```
Guillaume@friis:~> brew install minicom
```

## Configuration

Once this is done, you need to launch **Minicom** for configuration:

```
Guillaume@friis:~> minicom -s
```

In the `configuration` screen, chose `Serial port setup`, press `A` to update the `Serial device` parameter to point to the USB-to-serial device. Then press `E` to set the transfer rate to 9600 bauds. The settings should look as follows:

```
+-----------------------------------------------------------------------+  
| A -    Serial Device      : /dev/tty.usbserial                        |  
| B - Lockfile Location     : /usr/local/Cellar/minicom/2.4/var         |  
| C -   Callin Program      :                                           |  
| D -  Callout Program      :                                           |  
| E -    Bps/Par/Bits       : 9600 8N1                                  |  
| F - Hardware Flow Control : Yes                                       |  
| G - Software Flow Control : No                                        |  
|                                                                       |  
|    Change which setting?                                              |  
+-----------------------------------------------------------------------+  
```

Press `Enter` or `Esc` to return to the `configuration` screen.


In the `configuration` screen, chose `Modem and dialing`, press `A` to empty
the Init string. The settings should look as follows:

``` 
+--------------------[Modem and dialing parameter setup]---------------------+
|                                                                            |
| A - Init string .........                                                  |
| B - Reset string ........ ^M~ATZ^M~                                        |
| C - Dialing prefix #1.... ATDT                                             |
| D - Dialing suffix #1.... ^M                                               |
| E - Dialing prefix #2.... ATDP                                             |
| F - Dialing suffix #2.... ^M                                               |
| G - Dialing prefix #3.... ATX1DT                                           |
| H - Dialing suffix #3.... ;X4D^M                                           |
| I - Connect string ...... CONNECT                                          |
| J - No connect strings .. NO CARRIER            BUSY                       |
|                           NO DIALTONE           VOICE                      |
| K - Hang-up string ...... ~~+++~~ATH^M                                     |
| L - Dial cancel string .. ^M                                               |
|                                                                            |
| M - Dial time ........... 45      Q - Auto bps detect ..... No             |
| N - Delay before redial . 2       R - Modem has DCD line .. Yes            |
| O - Number of tries ..... 10      S - Status line shows ... DTE speed      |
| P - DTR drop time (0=no). 1       T - Multi-line untag .... No             |
|                                                                            |
| Change which setting?       (Return or Esc to exit)                        |
+----------------------------------------------------------------------------+
```
Press `Enter` or `Esc` to return to the `configuration` screen.

Select `Save setup as dfl` and then `Exit from Minicom`.

## Usage

Next connect your USB-to-Serial adapter to the Cisco management cable and the
management cable to your router. Launch **Minicom** and you should connect to the management console:

```
Guillaume@friis:~> minicom
```

By default, when **Minicom** refers to the `META` key, it means `ESC` on your Macintosh keyboard. For instance, use:

* `ESC` + `Z` for help
* `ESC` + `X` to cleanly exit **Minicom**