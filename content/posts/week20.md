---
date: '2025-11-04T01:37:14-08:00'
draft: false
title: 'Week 20: Setting Up PAPPL and Wrapping Up'
---
This is another short status update on PAPPL and the project in general. I have mainly spent this week modifying PAPPL source code to fit the current use case on Zephyr. The project I have does compile (although many of the allocated stacks/pools are shrunk due to the increased memory requirements). Currently, I am dealing with some issues related to thread local storage/stack pointer stuff as seen below:

```
D: (): zsock_getaddrinfo: host: , service: 8000, hints: 0x3c5f31f0
D: (): socket: ctx=0x3fccd3f8, fd=8
I: Listening for connections on ':1012871920'.
E:  ** FATAL EXCEPTION
E:  ** CPU 0 EXCCAUSE 28 (load prohibited)
E:  **  PC 0x420a50ae VADDR 0xc
E:  **  PS 0x60c20
E:  **    (INTLEVEL:0 EXCM: 0 UM:1 RING:0 WOE:1 OWB:12 CALLINC:2)
E:  **  A0 0x820b74e8  SP 0x3c5f3510  A2 0x3fcd3c90  A3 (nil)
E:  **  A4 0x3c5f34e0  A5 0x3fcd4890  A6 (nil)  A7 0x4207f3e0
E:  **  A8 0x820b7460  A9 0x3c5f30c0 A10 0x3fcd3c90 A11 0x3c5f34e0
E:  ** A12 0x1 A13 0x1 A14 0xa0 A15 0xc
E:  ** LBEG 0x400556d5 LEND 0x400556e5 LCOUNT 0xfffffffc
E:  ** SAR 0x1e
E:  **  THREADPTR (nil)
E: >>> ZEPHYR FATAL ERROR 0: CPU exception on CPU 0
E: Current thread: 0x3fcc22d0 (unknown)
E: Halting system
```

The main things that I have had to remove or will need to remove in the future are listed here:

- Network device-related functionality  
- Libusb-related code, which will have to be replaced with Zephyr’s USB API and a future port of ippusb\_bridge  
- Executing external scripts or using pipes (I’d ideally want to keep the number of threads to a minimum)  
- Stuff related to users/user groups  
- DNSSD browsing and querying (advertising will also have to be replaced with Zephyr’s API)

Unfortunately, there is definitely not enough time left to replace all of the USB printer code to simulate a USB printer, so that is something that will have to be explored in the future. Otherwise, I will get the server aspect of PAPPL running as best as I can despite the lack of the ability to connect a printer. I will also explore ippeveprinter once again as a proof-of-concept for printing on Zephyr.

You can see my version of PAPPL [here](https://github.com/HubertYGuan/pappl/tree/zephyr) and, as always, you can keep track of my final progress on the design diagram and project proposal below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)