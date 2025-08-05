---
date: '2025-08-04T14:08:05-07:00'
draft: false
title: 'Week 7: Midterm Update'
---
Now that midterm evaluations have come out, it is time to reflect on my experiences and the progress that I’ve made. Overall, I am proud of the progress that I have made, but I know that there is a lot more work left to do. For this week, I have been able to get a test suite running for libcups. However, there is a lot more testing to be done since libcups is a lot more complex and less self-contained than other libraries like zlib or pdfio.

# Current State of libcups

You can see in the `CMakeLists.txt` of my libcups fork that there are some files not currently being compiled and some missing functionality. This is mainly due to two factors that would each take a while to account for: file locking in Zephyr and mDNS Responder. The former is mostly out of scope for this project, but the latter is something that I will look into more after testing the other parts of libcups, since DNSSD is pretty important for things like advertising the print server. However, this will take quite a bit of effort since these unimplemented parts of mDNS Responder usually rely on daemons, which seems to be why I do not see the entirety of mDNS Responder ported to Zephyr or other embedded OSes. 

Additionally, some tests are also not compiled currently since they use syscalls for pipes and process forking interfaced by the POSIX API, which needs to be replaced with Zephyr APIs. However, I have not actually been able to find a Zephyr equivalent for `fork`, so that will need to be accounted for.

# Testing

Unfortunately, the progress with testing has not been the best since I have spent a lot of time moving into a new apartment. Additionally, testing will likely take a while since my summer classes have just begun, and it looks like I will also have to learn OpenOCD, unlike the last time that I mentioned it. This is because the very first test in `testarray.c` calls the function `cupsArrayNew` that gives the following error on my ESP32:  
```  
cupsArrayNew: E:  \*\* FATAL EXCEPTION  
E:  \*\* CPU 0 EXCCAUSE 28 (load prohibited)  
E:  \*\*  PC 0x4000be94 VADDR (nil)  
E:  \*\*  PS 0x60a20  
E:  \*\*    (INTLEVEL:0 EXCM: 0 UM:1 RING:0 WOE:1 OWB:10 CALLINC:2)  
E:  \*\*  A0 0x800593ac  SP 0x3ffd3920  A2 0xe  A3 0xa  
E:  \*\*  A4 0x3f405906  A5 0x3ffd3940  A6 0x3ffd3920  A7 0x4  
E:  \*\*  A8 (nil)  A9 0x3ffd32b0 A10 0xe A11 0x3ffb01c0  
E:  \*\* A12 0xe A13 0x3ffd3940 A14 0x3ffd3920 A15 0x3ffd36b0  
E:  \*\* LBEG 0x400014fd LEND 0x4000150d LCOUNT 0xfffffffd  
E:  \*\* SAR 0x1f  
E:  \*\*  THREADPTR 0x5316bb7

Backtrace:0x4000be91:0x3ffd3920 0x400593a9:0x3ffd3940 0x400fec4e:0x3ffd3960 0x400fec75:0x3ffd39b0 0x400d1657:0x3ffd39d0 

E: \>\>\> ZEPHYR FATAL ERROR 0: CPU exception on CPU 0  
E: Current thread: 0x3ffd5f50 (unknown)  
E: Halting system  
```
However, I do not even know the extent to which OpenOCD would help in a case like this, since it is a problem on the architectural level caused by loading from a page where loading is prohibited.

# Next Steps

I will fill out the midterm evaluation and then get back to working on libcups testing. In terms of the timeline, I think that this week is going to have to be dedicated to testing libcups, and PAPPL will have to be pushed back until libcups is functioning to a satisfactory level. I do not want to make any hard deadlines or promises since I know that testing and implementing these libraries and programs often take a lot more time than I initially think, but I think that I should hopefully be able to get libcups and PAPPL mostly functioning (possibly as well as some part of ipp-usb-bridge) by the time CUPS 3.0b1 comes around in September/October so that I will be ready to implement CUPS. Otherwise, the timeline and design diagram (which will both be updated) are listed below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)