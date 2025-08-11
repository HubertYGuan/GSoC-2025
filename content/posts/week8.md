---
date: '2025-08-11T12:35:23-07:00'
draft: false
title: 'Week 8: libcups Updates'
---
As I alluded to in my previous post, I am now back in Santa Barbara (or rather Goleta). This means I have some additional resources from my lab and professors, but also a lot of additional responsibilities from my classes, dealing with my apartment, etc. This means that I will have a smaller amount of time to work on GSoC, but I will make sure to work on the project as much as I can and continue to keep you all updated.

# libcups Testing

Thankfully, libcups seems to finally be compiling and linking fine (besides the parts with mDNS Responder) after changing some Mbed TLS configs. I have also shifted the testing framework over from using stdio to Zephyr logging since I was receiving load-prohibited exceptions (like the one I mentioned in the previous post) from these functions. After modifying the tests to account for the filesystem, I was able to get `testarray` and `testclock` to pass, but now I am trying to tackle `testcreds`.

Currently, whenever I try to run `testcreds`, I get no output from my program’s logging, both with and without `CONFIG_LOG_MODE_IMMEDIATE` on. The last time I saw this was when I was testing PDFio using the `md2pdf` program, which likely caused a heap overflow or some memory overflow. I think that `testcreds` also uses quite a lot of stack memory as seen in the 30KB+ buffers for some of the TLS functions on top of the heap allocations, so I am suspecting a lack of memory here, despite any concrete evidence like an exception that would indicate a stack overflow. Therefore I will also work on reducing the memory footprint of these functions and the library as a whole. Lastly, I should also mention that there is a possibility that file i/o may be causing some issues here since file i/o is more difficult compared to on Linux and I do not have the CA certificate files on my board that a Linux system would have.

# Other Thoughts and Next Steps

I currently do not have a board for JTAG debugging with my ESP32, but I should be getting one delivered relatively soon, which should make debugging easier (especially for testcreds). Otherwise, I will look toward tackling some of the other tests and seeing how I could implement mDNS Responder. The latter combined with my test suite’s already high memory usage makes me especially worried about memory constraints in the future, however. Since I do not have very much experience with memory optimization on the embedded level (CS classes only really seem to care about memory complexity in Leetcode), I will have to discuss this issue further throughout the porting process. Otherwise, you can find the current design diagram and proposal/timeline below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)