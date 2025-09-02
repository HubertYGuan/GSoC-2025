---
date: '2025-09-01T18:56:48-07:00'
draft: false
title: 'Week 11: ippeveprinter Continued'
---
This week was mainly spent trying to figure out all of the small issues in trying to translate ippeveprinter and libcups functionality over from Linux. I’ve learned more about how Zephyr handles concepts like pthreads and how they can be a lot more difficult to deal with than on Linux. I also recently received my ESP32s3 board, so I have also been testing the application there as well.

# ippeveprinter Update

Luckily, it seems that using the previously mentioned syscalls like `pipe`, `dup`, and `execve` are not entirely necessary for the functioning of ippeveprinter since those are only used to execute user-defined bash commands for incoming print jobs. However, something else has become the main problem with implementing the rest of ippeveprinter: pthreads. Since libcups uses pthread keys to manage its globals, I’ve had issues every time I’ve tried to initiate and store globals. Mainly, `pthread_setspecific` will return `EINVAL` since `to_posix_thread(pthread_self())` will fail. Inside of `pthread_self`:  
```
pthread_t pthread_self(void)
{
   size_t bit;
   struct posix_thread *t;


   t = (struct posix_thread *)CONTAINER_OF(k_current_get(), struct posix_thread, thread);
   bit = posix_thread_to_offset(t);
   LOG_DBG("bit (%u)", bit);
   return mark_pthread_obj_initialized(bit);
}

```
the value `t` will be outside of the `posix_thread_pool` that `posix_thread_to_offset` gets the offset based on. Therefore, I get the debug message that the bit offset is `296204322` or `11a7b822`, which is far outside of the bounds of `posix_thread_pool`. I also tried running a test with just `pthread_key_create` and `pthread_set_specific` running in its own Zephyr thread, but I still get the same kind of result.

Edit: I found [this issue](https://github.com/zephyrproject-rtos/zephyr/issues/60769) that details this behaviour and how I actually have to use a thread through `pthread_create` to be able to run this code. (Also, it’s marked as closed, but I was not able to find documentation regarding this issue) Otherwise, I will attempt to run ippeveprinter in a pthread due to this.

In other news, I have been able to get wifi up and running, so ippeveprinter can connect to the local network. However, there are more concerning issues when it comes to networking. Firstly, the libcups DNS-SD API has to be changed significantly in terms of design, since registration of services seems to be much better done by the user because of the const iterable sections that store DNS-SD service data. Therefore, I will add Kconfig options for the ippeveprinter application that allow setting DNS-SD service details (since you can’t do that at run time). Otherwise, I have some future concerns in regards to the libcups HTTP API, since it is quite different from the Zephyr HTTP client and related APIs, and it was written mainly for Unix-like OSes (but time will tell if I will have to switch over to the Zephyr APIs).

# Miscellaneous and Next Steps

I mentioned earlier that I just received my ESP32s3 Devkitc, which should be a sizeable upgrade for development. That’s because I was running low on DRAM space for my older ESP32, but the new ESP32s3 has quite a bit more buffer room in terms of memory. Additionally, I will make sure to leverage OpenOCD JTAG debugging (hopefully) so that I won’t have to keep rebuilding/reflashing to add log statements and `k_sleep`s in the code to make sure the log messages get through before an exception is thrown. Otherwise, after dealing with pthreads and setting up the Kconfig options for DNS-SD, I should hopefully be pretty close to getting a version of ippeveprinter running on Zephyr that can be recognized as a printer on my network. As always, you can check my progress through the links below in the design diagram and project timeline.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)