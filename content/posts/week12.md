---
date: '2025-09-08T23:41:54-07:00'
draft: false
title: 'Week 12: More Debugging'
---
Although I was not able to work very much this week, I was able to delve further into running ippeveprinter, and I also have an update on JTAG debugging. I was able to resolve the previous issues about pthreads for now, as well as most of my previous issues with debugging, but there is now a whole new suite of issues for me to deal with.

# ippeveprinter

After running ippeveprinter in a pthread, I noticed that I would either get stuck somewhere spinning indefinitely or get the following store-prohibited error:  

```
[00:00:05.373,000] <inf> libcups: Starting loop
[00:00:05.373,000] <inf> libcups: Polling
[00:00:05.373,000] <err> os:  ** FATAL EXCEPTION
[00:00:05.373,000] <err> os:  ** CPU 0 EXCCAUSE 29 (store prohibited)
[00:00:05.373,000] <err> os:  **  PC 0x420137de VADDR (nil)
[00:00:05.373,000] <err> os:  **  PS 0x60620
[00:00:05.373,000] <err> os:  **    (INTLEVEL:0 EXCM: 0 UM:1 RING:0 WOE:1 OWB:6 CALLINC:2)
[00:00:05.373,000] <err> os:  **  A0 0x820138d4  SP 0x3fcba4a0  A2 (nil)  A3 0x3fcafaa8
[00:00:05.373,000] <err> os:  **  A4 (nil)  A5 0x60620  A6 (nil)  A7 0x51feaa
[00:00:05.373,000] <err> os:  **  A8 0x820137b6  A9 0x51feaa A10 (nil) A11 (nil)
[00:00:05.373,000] <err> os:  ** A12 0x3fcbecac A13 0x3fca8600 A14 0xffffffff A15 0xffffffff
[00:00:05.373,000] <err> os:  ** LBEG 0x40056f5c LEND 0x40056f72 LCOUNT 0xffffffff
[00:00:05.373,000] <err> os:  ** SAR 0xa
[00:00:05.373,000] <err> os:  **  THREADPTR 0x3fcba470
[00:00:05.373,000] <err> os: >>> ZEPHYR FATAL ERROR 0: CPU exception on CPU 0
[00:00:05.373,000] <err> os: Current thread: 0x3fcba7a8 (esp_timer)
[00:00:05.602,000] <err> os: Halting system
```

From the logs, I can see that the exception happens somewhere in [this loop](https://github.com/HubertYGuan/zephyr-example/blob/main/tests/libcups/ippeveprinter.h#L5592) due to `esp_timer`. I even get the same error from running the program without the code commented out in the loop. However, if I don’t use the `k_sleep` or I use `pthread_detach` instead of `pthread_join` in my `main`, the CPU will get stuck and spin indefinitely (which is indicated by how `avahi-browse` is not able to find the service the application registers despite it being able to do so in my other testing). This also just happens most times I use `west espressif monitor` for the first time after flashing.

I am thinking that there is either something weird going on with using `k_sleep`, `poll`, and other functions that use a timer within pthreads, or there is some other pthread-related issue at play here. Therefore, I will have to do some more testing of my own with pthreads to figure out what is exactly going on.

# JTAG Debugging

I have also been experimenting with JTAG debugging with my ESP32s3, and it has been quite  a journey just to get it to a point where I can actually step through and set breakpoints. Notably, the app image offset has to be set using Espressif’s OpenOCD commands and there has been inconsistent behaviour where sometimes the program will halt despite normally proceeding past a point when using the Espressif monitor. However, I have been able to use JTAG debugging to get the following details about the issue I mentioned in the previous section:  

```
Breakpoint 3, run_printer (printer=0x3fcca708) at (user)/zlibtest/module-app/tests/libcups/ippeveprinter.h:5594
5594        LOG_INF("Polling");
(gdb) break ippeveprinter.h:5602
Breakpoint 4 at 0x4200a000: ippeveprinter.h:5602. (2 locations)
(gdb) c
Continuing.
[esp32s3.cpu0] Target halted, PC=0x4038CCF6, debug_reason=00000001
Don't have the number of threads in FreeRTOS!
[esp32s3.cpu0] Target halted, PC=0x4038CCF6, debug_reason=00000001
^C[esp32s3.cpu0] Target halted, PC=0x4037D6CE, debug_reason=00000000
Set GDB target to 'esp32s3.cpu0'
[esp32s3.cpu1] Target halted, PC=0x40043A40, debug_reason=00000000

Program received signal SIGINT, Interrupt.
arch_system_halt (reason=0) at (user)/zlibtest/zephyr/kernel/fatal.c:30
30              for (;;) {
(gdb) backtrace
#0  arch_system_halt (reason=0) at (user)/zlibtest/zephyr/kernel/fatal.c:30
#1  0x4037a33d in k_sys_fatal_error_handler (reason=0, esf=0x3fcba420 <timer_task_stack+3872>) at (user)/zlibtest/zephyr/kernel/fatal.c:44
#2  0x4037a520 in z_fatal_error (reason=0, esf=0x3fcba420 <timer_task_stack+3872>) at (user)/zlibtest/zephyr/kernel/fatal.c:119
#3  0x40378f78 in xtensa_fatal_error (reason=0, esf=0x3fcba420 <timer_task_stack+3872>) at (user)/zlibtest/zephyr/arch/xtensa/core/fatal.c:109
#4  0x4037a1ef in xtensa_excint1_c (esf=0x3fcba420 <timer_task_stack+3872>) at (user)/zlibtest/zephyr/arch/xtensa/core/vector_handlers.c:687
#5  0x4037915e in _xstack_call0_6 () at (user)/zlibtest/zephyr/arch/xtensa/core/xtensa_asm2_util.S:364
#6  0x40379148 in _do_call_4 () at (user)/zlibtest/zephyr/arch/xtensa/core/xtensa_asm2_util.S:364
#7  0x420138d4 in timer_task (arg=0x0) at (user)/zlibtest/modules/hal/espressif/components/esp_timer/src/esp_timer.c:468
#8  0x4200c6f8 in z_thread_entry (entry=0x420138bc <timer_task>, p1=0x0, p2=0x0, p3=0x0) at (user)/zlibtest/zephyr/lib/os/thread_entry.c:48
```

This confirms that there is some sort of error with using timers, but I will have to look closer into the source code and do some separate testing to figure out how to use these timers without causing this load prohibited exception.

# Next Steps

Although I have final exams coming up for this week, I still should be able to carry out some of the aforementioned tests with timers. Otherwise, I will also look into testing the actual http and ipp capabilities of ippeveprinter on Zephyr in order to get a printer that can be interacted with up and running. As always, you can also follow my progress on the project proposal and design diagram linked below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)