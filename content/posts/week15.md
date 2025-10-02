---
date: '2025-10-02T00:47:49-07:00'
draft: false
title: 'Week 15: More pthread and Memory Adventures in ippeveprinter'
---
This week has definitely had many ups and downs in terms of this project and my work now that I am officially starting my Fall quarter. Surprisingly (or unsurprisingly due to reasons I will get into), I have not had that much time to work on this project despite classes being basically nonexistent in terms of assignments (with the exception of the embedded systems course ECE 153a that makes me go into Vivado even before the first lecture). The main thing that has been taking up my time is working on a new initiative with the ACM student chapter at UCSB, which involves managing communications with much of the faculty from two departments and many meetings for event planning.

Otherwise, I was able to explore further into HTTP communications with the use of PSRAM or SPI RAM on the ESP32s3 to alleviate memory bottlenecks. In terms of good news, I can use 8MB of external memory, meaning that running out of memory should hopefully not be an issue anytime soon (but it does mean the print server will only work on boards with around this amount of external memory on them). On the other hand, memory allocated from PSRAM is not the same as memory from the system heap, leading to memory management issues.

Here is an example of an error that prevents me from proceeding with using PSRAM (at least for pthread stacks):

```
#0  arch_system_halt (reason=0) at /home/(user)/zlibtest/zephyr/kernel/fatal.c:30
#1  0x40379602 in k_sys_fatal_error_handler (reason=0, esf=0x3fcb20b0 <sys_work_q_stack+512>) at /home/(user)/zlibtest/zephyr/kernel/fatal.c:44
#2  0x4037963b in z_fatal_error (reason=0, esf=0x3fcb20b0 <sys_work_q_stack+512>) at /home/(user)/zlibtest/zephyr/kernel/fatal.c:119
#3  0x40378e6c in xtensa_fatal_error (reason=0, esf=0x3fcb20b0 <sys_work_q_stack+512>) at /home/(user)/zlibtest/zephyr/arch/xtensa/core/fatal.c:109
#4  0x40379467 in xtensa_excint1_c (esf=0x3fcb20b0 <sys_work_q_stack+512>) at /home/(user)/zlibtest/zephyr/arch/xtensa/core/vector_handlers.c:693
#5  0x40379052 in _xstack_call0_6 () at /home/(user)/zlibtest/zephyr/arch/xtensa/core/xtensa_asm2_util.S:364
#6  0x4037903c in _do_call_4 () at /home/(user)/zlibtest/zephyr/arch/xtensa/core/xtensa_asm2_util.S:364
#7  0x4037af6c in z_unpend_thread (thread=0x3c316f3c <_net_buf_tx_bufs+140>) at /home/(user)/zlibtest/zephyr/kernel/sched.c:659
#8  0x4037b41b in z_unpend_all (wait_q=0x3c317d1c <_k_mem_slab_buf_tx_pkts+12>) at /home/(user)/zlibtest/zephyr/kernel/sched.c:945
--Type <RET> for more, q to quit, c to continue without paging--
#9  0x4037994c in k_heap_free (heap=0x3c317d10 <_k_mem_slab_buf_tx_pkts>, mem=<optimized out>) at /home/(user)/zlibtest/zephyr/kernel/kheap.c:213
#10 0x4037c692 in k_free (ptr=0x3fcb7d4c <kheap.system_heap+22812>) at /home/(user)/zlibtest/zephyr/kernel/mempool.c:69
#11 0x4037c77a in z_impl_k_thread_stack_free (stack=0x3fcb7d50 <kheap.system_heap+22816>) at /home/(user)/zlibtest/zephyr/kernel/dynamic.c:151
#12 0x420249fa in k_thread_stack_free (stack=<optimized out>) at /home/(user)/zlibtest/build/zephyr/include/generated/zephyr/syscalls/kernel.h:52
#13 pthread_attr_destroy (_attr=0x3fcbacc8 <posix_thread_pool+664>) at /home/(user)/zlibtest/zephyr/lib/posix/options/pthread.c:1425
#14 0x42024b42 in posix_thread_recycle () at /home/(user)/zlibtest/zephyr/lib/posix/options/pthread.c:568
#15 0x42024bae in posix_thread_recycle_work_handler (work=0x3fc95e50 <posix_thread_recycle_work>)
    at /home/(user)/zlibtest/zephyr/lib/posix/options/pthread.c:462
#16 0x4037a4b0 in work_queue_main (workq_ptr=0x3fcbd5e8 <k_sys_work_q>, p2=<optimized out>, p3=<optimized out>)
--Type <RET> for more, q to quit, c to continue without paging--
    at /home/(user)/zlibtest/zephyr/kernel/work.c:737
#17 0x42010ae0 in z_thread_entry (entry=0x4037a454 <work_queue_main>, p1=0x3fcbd5e8 <k_sys_work_q>, p2=0x0, p3=0x0)
    at /home/(user)/zlibtest/zephyr/lib/os/thread_entry.c:48
```

This happens because `k_free` will cause a fatal error when used on memory not from the system heap (such as from PSRAM). However, I do not think it makes sense to just try to use regular system heap memory for all of these threads, since I’ve counted that I need about 25 threads in total, and each of these threads will most likely take up a considerable amount of stack memory (such as 32768 bytes). Although some threads terminate before others begin, the \~150 KB of remaining system heap would not be enough for these threads and other allocations (especially not with PAPPL).

This leaves one main option, which is allowing pthreads (or rather `pthread_attr_t`s) to keep track of which stacks are from the PSRAM. Luckily, adding one bit to `pthread_attr_t` if PSRAM is enabled should do the trick (but that does mean modifying the kernel and trying to get the changes upstreamed).

In terms of issues later in the future, I realize that having upwards of 25 threads running simultaneously is not the easiest to manage, and there will likely be multithreading issues that pop up. Furthermore, I have not tested the HTTP API due to all the issues I’ve been having with memory allocation and multithreading, so that could also turn out to be a major issue.

Although the sort of program I have with ippeveprinter is seemingly removed from the classical “grand loop” architecture that is being stressed in my embedded systems class (it’s not like the 25 threads are all constrained to finish within one "cycle" that fits strict time requirements in `run_printer`, but at the same time, a print server doesn’t really have hard deadlines like an aircraft), I will still take the advice of my professor to change just a few lines at a time before testing and continuing consistently throughout the days with some gaps between to allow my mind to think clearly on the test results I get. Hopefully, this will mean I can transfer this work over to PAPPL somewhat soon (and then scramble to learn how to port a Rust library for ippusb\_bridge). As always, you can keep up with my progress on the design diagram and project timeline below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)