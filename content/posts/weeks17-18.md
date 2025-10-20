---
date: '2025-10-20T01:35:03-07:00'
draft: false
title: 'Week 17/18: Testing with the New Memory Management Scheme'
---
I had to combine these two weeks into one report since I generally do not have much time to work during the first half of the week since that’s when I have to spend the most time on schoolwork (now that the academic term has fully begun). I was able to focus on testing during the last few days and have discovered some important information, and I should be able to progress to testing PAPPL relatively soon.

# libcups Testing

Now that I have more networking, external memory, etc. infrastructure in place, I was able to pass the libcups HTTP and Language tests as well as most of the IPP tests. The main issues I am encountering have to do with the memory allocation mechanisms that the `sys_heap`	functions and therefore the shared multi-heap use. Since the networking thread stacks can get quite large, I originally moved them over to external memory. However, `sys_heap` functions are not atomic and do not make use of semaphores, allowing for all kinds of allocation bugs when I am conducting tests. This is especially true since I have decided to move all of the CUPS heap allocation functionality over to external memory (mainly so I wouldn’t have to always keep track of which free function to use and since sometimes the regular heap would overflow). Since I also have already made some changes to the Zephyr kernel, I will go forward with making a new Kconfig option in the kernel tree to turn on atomic `sys_heap` functions (since this seems more straightforward than adding semaphores to every call or making a bunch of wrapper functions).

# Next Steps

Otherwise, I will continue with finishing the rest of the relevant libcups tests, including redoing the array test since it fails with the new memory scheme. This will mainly include the IPP fuzzing, threads, TLS check, creds, and file i/o. After finishing that, I will make a fork of PAPPL to continue with testing PAPPL functionality on the print server side.

As I am also often working on labs for my embedded systems class, I will also make sure to make use of more of the knowledge of lower-level functionality of embedded systems as I continue with testing. For example, I have learned some computer microarchitecture before, but I have also been exposed to ISAs on embedded devices. Therefore, I will try to make use of looking at registers, stack pointers, etc. in the exception reports as well as in the debugger (since it seems like regular line-by-line stepping skips over a lot of stuff, so I will see if instruction-by-instruction stepping is any better). I’ll also try to keep the changes between test runs to a minimum and also keep track of my changes (which is something I have not always been the best at).

As always, you can find the design diagram and project proposals linked below.  
Project Proposal (also contains a bunch of logs):  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)