---
date: '2025-08-18T18:35:30-07:00'
draft: false
title: 'Week 9: Debugging and mDNS Responder'
---
This week has definitely been less straightforward than the previous weeks as I continue testing libcups and figuring out a clear path forward for what needs to be ported and what can be left out. I have continued with getting through the other tests for libcups (at least the ones relevant to the print server application), exploring mDNS Responder use in libcups, and I have also been working on getting OpenOCD debugging with my debugger board.

# Tests and mDNS Responder

I have been able to run and debug a few more tests for libcups. In general, I’m seeing a few main roadblocks to getting these features into the port. The first one should not be too difficult, which is file i/o. It seems like all the file i/o should be rewritten using Zephyr’s FS API instead of the POSIX API, since the latter seems to cause load-prohibited exceptions. The second main roadblock is networking and HTTP. Since Zephyr’s preferred networking APIs work a bit differently from what libcups uses, it is very possible that the whole networking stack would need to be migrated to Zephyr’s APIs. If you want to see details on the individual tests I have run, you can take a look at the project proposal.

In terms of mDNS Responder and DNS-SD in general, it seems like Zephyr’s built-in mDNS Responder capabilities should be adequate for just letting the print server advertise itself as a service and handle queries. However, libcups is made for both clients trying to send print jobs and the print servers themselves, so the DNS-SD functionality with browsing will have to be removed. There is no clear divide in the DNS-SD code between the sharing and local sides, so I am thinking of just implementing the functionality that an example program like `ippeveprinter` uses and removing the rest for now.

# Debugging

I recently received my ESP-Prog JTAG debugger probe and have been spending much of my time ever since trying to get debugging with OpenOCD and Zephyr running with this. On the bright side, I have been able to debug programs on my ESP32-WROOM-32D when they are using the Arduino or the ESP-IDF frameworks. I’m able to set break points, jump, set variables (most of the time), etc. However, no matter what I have tried, I cannot get a Zephyr program to pass the initial examination in OpenOCD, and I get the error below.   
``` 
$ openocd \-f board/esp32-wrover-kit-3.3v.cfg \-c "adapter speed 5000"  
Open On-Chip Debugger 0.12.0-01018-gb89d626c6 (2025-08-18-16:03)  
Licensed under GNU GPL v2  
For bug reports, read  
        http://openocd.org/doc/doxygen/bugs.html  
force hard breakpoints  
adapter speed: 20000 kHz

adapter speed: 5000 kHz

Info : Listening on port 6666 for tcl connections  
Info : Listening on port 4444 for telnet connections  
Info : clock speed 5000 kHz  
Info : JTAG tap: esp32.cpu0 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)  
Info : JTAG tap: esp32.cpu1 tap/device found: 0x120034e5 (mfg: 0x272 (Tensilica), part: 0x2003, ver: 0x1)  
Error: Unexpected OCD\_ID \= 00000000  
Warn : target esp32.cpu1 examination failed  
Info : starting gdb server for esp32.cpu0 on 3333  
Info : Listening on port 3333 for gdb connections  
Error: Unexpected OCD\_ID \= 00000000  
Error: Unexpected OCD\_ID \= 00000000  
Error: Unexpected OCD\_ID \= 00000000  
Error: Unexpected OCD\_ID \= 00000000  
Error: Unexpected OCD\_ID \= 00000000  
``` 
The only mention of this `Unexpected OCD_ID = 00000000` error I have seen is related to the ESP-32S3 when trying to use the JTAG pins without burning the eFuse to use them properly. However, it is clear that my board should not have this issue since JTAG debugging works fine with a different RTOS. I have tried with OpenOCD from Espressif, the one included in the Zephyr SDK 0.17.2 under `sysroots`, and the Zephyr fork on GitHub, all to no avail.

In terms of possible solutions, I have seen videos showing that the ESP32S3 boards with built-in JTAG debugging work with Zephyr, and the ESP32-WROOM-32D is not recommended for new designs, meaning that a change in hardware is always an option if I can’t figure something else out. Otherwise, please let me know of any other things you think could help with this.

# Next Steps

JTAG debugging is a roadblock right now, so I will spend some time trying some suggestions to see if I can get that running. Otherwise, I will focus mainly on implementing the DNS-SD functionality libcups needs and also further investigating libcups’ networking stack. As always, you can view the current project proposal and design diagram below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)