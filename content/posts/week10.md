---
date: '2025-08-25T16:42:28-07:00'
draft: false
title: 'Week 10: DNS-SD and ippeveprinter'
---
This week, I put the JTAG debugging and the rest of the libcups tests on hold in favor of getting ippeveprinter (a minimal print server example) up and running on Zephyr. Generally, what I have been seeing myself doing through this process and stage of development is removing all of the parts that would not be feasible to support right now and focusing on the actual application rather than all of the tests for libraries. This is because it’s become clear to me that time is going to be scarce over the next weeks as I have finals for my summer classes and then a significantly more difficult Fall term after that.

# DNS-SD and mDNS Responder

Despite having to work on midterms for most of this week, I was able to investigate further into Zephyr’s mDNS Responder implementation and implement libcups’ DNS-DS API using it (although only with service registration). However, Zephyr’s DNS-SD and mDNS Responder systems leave a lot to be desired, especially when the program needs to register more complex services at run time. Currently, the services use the `dns_sd_rec` struct and have to be declared as `static const` and stored as iterable sections in the ROM when registered. This is difficult to manage since I cannot declare services dynamically on the heap like in Apple’s mDNS Responder, and I am stuck with whatever input values I put in (hostname, domain, service type, etc., besides the port) when I declare them as global variables. I also do not think that Zephyr supports multiple service types for a single service (which is something ippeveprinter would normally use).

If some of this functionality turns out to be needed for PAPPL, or as I am testing ippeveprinter, I think that the most straightforward way of dealing with this is to move the `dns_sd_rec` iterable section to RAM and use an interrupt or other system to alert mDNS Responder every time data is changed. However, this would require changing the actual Zephyr tree, and I am sure that there are reasons as to why the Zephyr developers do not want these `dns_sd_rec` structs to be changed dynamically. Therefore, it could possibly be implemented as an experimental Kconfig option. Otherwise, I do not think that Apple’s mDNS Responder would run well under heavy resource constraints, and making another module would not be ideal compared to building on what is already there in base Zephyr.

# Other Notes and Next Steps

Building the new DNS-SD API was not too difficult, but building a version of ippeveprinter has been more difficult due to its reliance on syscalls like `pipe`, `dup`, and `execve`. Therefore, a lot of the time I will spend trying to get this print server up will likely be using Zephyr’s thread API. I have also had no luck with JTAG debugging, and I feel like it would definitely be useful for running ippeveprinter. Therefore, I will look into getting a new ESP-32 kit that should be better supported by Zephyr and OpenOCD.

To summarize, I will continue working on ippeveprinter until I can get a satisfactory result, and then continue with tackling the more complex PAPPL. As always, you can keep up with my progress via the timeline and project proposal linked below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)
