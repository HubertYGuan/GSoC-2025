---
date: '2025-10-10T00:26:46-07:00'
draft: false
title: 'Week 16: ippeveprinter Woes and Updated Plans'
---
From what I could gather this week when working on ippeveprinter, it is likely not going to be running on Zephyr any time soon. The reality is that ippeveprinter is 8k lines of code without some kind of modular testbench and that it is clearly not written for embedded systems (besides embedded Linux devices, which have traits of traditional computer systems). It has also become a debugging nightmare to get through since logs are not clear as to what exactly is going wrong and there is not much comparison with regular ippeveprinter on Linux since most of the logs disappear if nothing goes wrong. Furthermore, step-through debugging is not the best when testing a real-time connection HTTP connection that can timeout. Lastly, it takes a lot of overhead time just to reset my testing setup since `ippfind` can often have trouble finding ippeveprinter.

For reference, the main errors I can see are indicated in the following lines:
```
I: 4_cupsSetError: last_error=server-error-internal-error, last_status_message="Success"
I: 4_cupsSetError: last_error=server-error-internal-error, last_status_message="No request 
```
and
```
E: net_context_get(): -2
E: Cannot allocate a new TCP connection
```
I also get the following backtrace when exploring the `_cupsSetError` calls:
```
Breakpoint 2, _cupsSetError (status=1070494016, message=0x4208c3a0 <_cupsSetError> "6A", localize=true) at /home/hubert/zlibtest/modules/lib/libcups/cups/request.c:862
862     {
(gdb) back
#0  _cupsSetError (status=1070494016, message=0x4208c3a0 <_cupsSetError> "6A", localize=true)
    at /home/hubert/zlibtest/modules/lib/libcups/cups/request.c:862
#1  0x4208b284 in cupsLangLoadStrings (lang=0x3fce7140, filename=<optimized out>, strings=<optimized out>)
    at /home/hubert/zlibtest/modules/lib/libcups/cups/language.c:515
#2  0x4208b428 in cups_lang_new (language=0x3c3711e8 "en_US") at /home/hubert/zlibtest/modules/lib/libcups/cups/language.c:604
#3  0x4208b550 in cupsLangFind (language=<optimized out>) at /home/hubert/zlibtest/modules/lib/libcups/cups/language.c:125
#4  0x4208aaf9 in cupsLangDefault () at /home/hubert/zlibtest/modules/lib/libcups/cups/langprintf.c:517
#5  0x4208c435 in _cupsSetError (status=<optimized out>, message=0x3c110210 "No request URI.", localize=<optimized out>)
    at /home/hubert/zlibtest/modules/lib/libcups/cups/request.c:889
--Type <RET> for more, q to quit, c to continue without paging--
#6  0x420809a2 in httpReadRequest (http=0x3c35afd0, uri=0x3c3722f8 "", urilen=1024) at /home/hubert/zlibtest/modules/lib/libcups/cups/http.c:2020
#7  0x4200e510 in process_http (client=0x3fce56b8) at /home/hubert/zlibtest/module-app/tests/libcups/ippeveprinter.h:4677
#8  0x4200eeec in process_client (client=0x3fce56b8) at /home/hubert/zlibtest/module-app/tests/libcups/ippeveprinter.h:4636
#9  0x42024995 in zephyr_thread_wrapper (arg1=0x3fce56b8, arg2=0x4200ee48 <process_client>, arg3=<optimized out>)
    at /home/hubert/zlibtest/zephyr/lib/posix/options/pthread.c:537
#10 0x42010d18 in z_thread_entry (entry=0x42024984 <zephyr_thread_wrapper>, p1=0x3fce56b8, p2=0x4200ee48 <process_client>, p3=0x0)
    at /home/hubert/zlibtest/zephyr/lib/os/thread_entry.c:48
(gdb) c
Continuing.
Don't have the number of threads in FreeRTOS!
[esp32s3.cpu0] Target halted, PC=0x4208C3A0, debug_reason=00000001
Set GDB target to 'esp32s3.cpu0'
[esp32s3.cpu1] Target halted, PC=0x40043A40, debug_reason=00000000

Breakpoint 2, _cupsSetError (status=1107870624, message=0x3c369858 "", localize=9) at /home/hubert/zlibtest/modules/lib/libcups/cups/request.c:862
862     {
(gdb) back
#0  _cupsSetError (status=1107870624, message=0x3c369858 "", localize=9) at /home/hubert/zlibtest/modules/lib/libcups/cups/request.c:862
#1  0x420809a2 in httpReadRequest (http=0x3c384528, uri=0x3c369858 "", urilen=1024) at /home/hubert/zlibtest/modules/lib/libcups/cups/http.c:2020
#2  0x4200e510 in process_http (client=0x3fce5ff8) at /home/hubert/zlibtest/module-app/tests/libcups/ippeveprinter.h:4677
#3  0x4200eeec in process_client (client=0x3fce5ff8) at /home/hubert/zlibtest/module-app/tests/libcups/ippeveprinter.h:4636
#4  0x42024995 in zephyr_thread_wrapper (arg1=0x3fce5ff8, arg2=0x4200ee48 <process_client>, arg3=<optimized out>)
    at /home/hubert/zlibtest/zephyr/lib/posix/options/pthread.c:537
#5  0x42010d18 in z_thread_entry (entry=0x42024984 <zephyr_thread_wrapper>, p1=0x3fce5ff8, p2=0x4200ee48 <process_client>, p3=0x0)
    at /home/hubert/zlibtest/zephyr/lib/os/thread_entry.c:48
```
It seems like there are some weird memory-related issues here (and the TCP connection allocation issue keeps popping up even if I put `CONFIG_NET_MAX_CONN=128` and `CONFIG_NET_TCP_WORKQ_STACK_SIZE=32768` and I doubt that I could increase it very much more).

# Updated Project Steps

Since ippeveprinter has proven to be much too difficult to debug, I will go back to testing more library functionality. This will be mainly HTTP for libcups, and then I will see what I can do with PAPPL’s functions as I implement them. Luckily, it seems that PAPPL’s architecture is much more modular and there are testbenches for testing different aspects of the print server (or mainloop). I also want to take another look at the TLS functionality since I am getting a load-prohibited exception in `tls-mbedtls.c`. 

Then, I can of-course get working on something a bit different, which is `ippusb_bridge` in Rust. This should hopefully be at least a bit more understandable since the source code files are not very large.

As always, you can find the design diagram and project proposals linked below.  
Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)