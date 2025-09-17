---
date: '2025-09-16T22:16:36-07:00'
draft: false
title: 'Week 13: Short ippeveprinter Update'
---
As I alluded to in my previous post, I did not have much time to work on ippeveprinter this week, but I did manage to figure out what was wrong before. Now, mostly DNS-SD and networking work remains to be done.

As for the previous issue I faced, I figured out that it was caused by a lack of available stack memory for `ippeve_main` and that I actually had to `malloc` the stack myself since the default `pthread_attr` allocation method conflicts with the net management allocations for some reason (if I allocate one, the other will fail to allocate unless I set the stack size low enough).

Otherwise, the only issues right now that are preventing my ESP32s3 from simulating a printer are related to DNS-SD. Namely, DNS-SD services of the type `_ipp._tcp` seem to be more difficult to advertise than a custom service type like `_zephyr._tcp` or a more standard service type like `_http._tcp`. Using the `service` function taken from the Zephyr mDNS Responder sample ([source code](https://github.com/HubertYGuan/zephyr-example/blob/main/tests/libcups/main.c#L212)), I am able to advertise with those two service types, but not with IPP (the service is not detected by `avahi-browse` or CUPS). I am thinking this may be because the IPP service type has a flagship type “printer,” and the multiple printer services need to be registered in the way that ippeveprinter does it (with a `_printer._tcp` service on port 0 registered first).

As for next steps, besides dealing with DNS-SD service registration, I will also have to debug the client processing functionality before moving on to PAPPL. Hopefully, this should not take too long (but this functionality does rely on the libcups HTTP API, so I will have to see about that). As always, you can follow my progress on the project proposal and design diagram linked below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)