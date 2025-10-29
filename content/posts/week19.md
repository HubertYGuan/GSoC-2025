---
date: '2025-10-29T02:10:59-07:00'
draft: false
title: 'Week 19: Wrapping up libcups Testing For Now'
---
This is a short status update on testing for libcups and PAPPL for the week (I can’t write that much since it is currently midterm season for me). Firstly, I have implemented the semaphores for `sys_heap` functions that I mentioned earlier. However, I have not extensively tested the new multithreaded `sys_heap` behavior with these semaphores. It seems like there are no current conflicts with the WiFi thread running in the external heap and the main thread interacting with the external heap at least.

Secondly, I have recently had the array and IPP tests pass. I was struggling for quite a while trying to figure out why the heap was behaving weirdly and it turned out it was because I set the free function of all CUPS arrays by default to use the external heap free, which caused double frees.

As for next steps, I have created a fork of the PAPPL repository [here](https://github.com/HubertYGuan/pappl/tree/zephyr) where you can keep track of my progress. I will focus on porting the core server-side functionality to try to get those parts of the API up-and-running, then focus on testing the provided print server application itself. Luckily, it seems like PAPPL is much more platform-agnostic and relies mostly on libcups, so I should not need to make much changes to the source code of PAPPL itself. I will try to get a final working product out as soon as possible since I do not have much time left, especially considering the midterms.

As always, you can find the design diagram and project proposals linked below.  
Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)