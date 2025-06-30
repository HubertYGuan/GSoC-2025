---
date: '2025-06-29T21:40:07-07:00'
draft: false
title: 'Week 2: PDFio and zlib Continued'
---
This week was mainly a continuation of my work during week 1. I continued work on zlib by conducting more rigorous testing, covering the relevant tests within the default CMakeLists. Overall, I think that zlib is mostly functional at the current state and should provide general compression and decompression functionality.

# Notes on zlib

There are still some things to keep in mind on the current state of zlib; first of which is building the test application, especially since I have to use a file system like littlefs. In `gzlib.c` and related source files for “gz” prefixed functions, file i/o functionality is required, and I figured that littlefs would be a good fit since it is designed for use in embedded applications. However, it seems that some platforms, such as my `native_sim` and `qemu_riscv64/32`, struggle with building the application with littlefs or with interacting with the filesystem. I have found that real hardware (i.e., the ESP32) works the best for building and running the application. However, the application still fails two coverage tests (`cover_wrap` and `cover_back`) in the provided CMake tests. Therefore, I will have to look into OpenOCD so I can get a debugger running. On the other hand, I’m able to pass all the tests not involving file i/o on native\_sim, likely since there are differences between the C libraries on `native_sim` and other platforms (e.g., I think I am able to include header files on my system but not in the Zephyr libc). Another factor to keep in mind is that I also decreased the parameters for max LZ77 window bits and default memory level to avoid a heap overflow on my ESP32. Lastly, I have a few remarks on the Kconfig option `CONFIG_POSIX_API`. Although this [page](https://docs.zephyrproject.org/latest/services/portability/posix/overview/index.html) states that the option should not be used for new Zephyr applications, the truth is that `unistd.h` still uses a `#ifdef CONFIG_POSIX_API` block around some important functions like `read()`, `write()`, etc. Therefore, I have set the option as a dependency for zlib.

# Notes on PDFio

The main issue with porting PDFio right now is that Zephyr does not currently implement the `strdup()` function. There is still an open [issue](https://github.com/zephyrproject-rtos/zephyr/issues/22464) on this, so I think it is about time to implement this for the sake of PDFio. Otherwise, PDFio seems to compile fine on `qemu_riscv64/32`, and I will begin testing shortly after implementing strdup.

# Next Steps

After I finish debugging the zlib and PDFio tests, I will do the same with libcups and then PAPPL. The latter two will be a step up in terms of complexity since networking is involved, and I have to pay attention to memory usage since zlib alone was already causing heap overflows with the default parameters. Otherwise, you can view my GitHub and the design documents to see the status of the project.

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)