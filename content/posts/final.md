---
date: '2025-11-07T02:29:12-08:00'
draft: false
title: 'GSoC 2025 Final Report: Porting CUPS to Zephyr'
---
# Project Overview

The main objective of this project is to bring OpenPrinting’s open source printing stack to resource-constrained systems using the Zephyr RTOS. This is motivated by the current lack of embedded, real-time print server solutions, which could open the door to highly efficient and closely coupled print server and printer hardware development. Furthermore, the boundaries for power and resource consumption in printing would be pushed further with the elimination of the overhead of operating systems like embedded Linux. For this project, only the topic of running a print server on Zephyr is explored, but future work in hardware integration is ripe for exploration.

In general, porting OpenPrinting’s printing stack means porting and replacing existing dependencies with Zephyr equivalents, porting the main library (llbcups) that allows for building basic print servers that process raw data (due to resource constraints), and porting/testing a main print server framework/application (PAPPL). On Zephyr, this means that each ported component is a module that can be enabled and configured by users via Kconfig. This also means the creation of test applications that actually run and use the modules and their functions. As for hardware support, I can only guarantee the results I have gathered on the ESP32-S3 microcontroller since that is the only platform I have tested on (besides an older ESP32).

# Progress and Leftover Work

(Note, all the links to my repositories and commits are found at the final section)

## Dependencies

The first part of the stack I completed porting and testing was the compression library zlib. This is a compression library with significantly different functionality compared to Zephyr’s lz4. Due to the differing functionality, porting was necessary to avoid extensive refactoring with lz4. Zlib is also fairly simple and easy to test with its provided tests, so I am confident in its performance on Zephyr.

The second dependency I ported was the PDFio library for reading and writing PDF files. However, I realize that this was not entirely necessary for the goal of running a print server on Zephyr since it does require quite a lot of resources to deal with PDF files compared to streaming raw data. However, the library is now a Zephyr module and I have done some basic tests to confirm at least some functionality, but not any tasks that require larger amounts of memory (more than there exists in the DRAM of an average microcontroller).

## Libcups

The vast majority of this project was spent on porting the server-side part of libcups that is concerned with raw data. This is a subset of libcups that does not support features that are moreso used in client side functionality such as code in `dest.c` or DNS-SD advertising.

Here is a quick run-down of everything that had to be changed or left out in this version of libcups on Zephyr:

* `dest.c` and related dest files are removed.  
* Mbedtls replaces gnutls and openssl. However, the mbedtls functionality has not been tested much as it has not been a main priority for the print server.  
* DNS-SD functionality is restricted strictly to Zephyr’s supported advertising capability. Furthermore, functions such as `cupsDNSSDServiceAdd` do not add DNS-SD services since these services must be added at compile time according to Zephyr’s API.  
* Use of the file system for config files and similar things is mostly unsupported, but the file i/o operations should still function relatively fine. There are some issues, however, with `ippFileRead` on large files.
* All memory allocation is moved to the external heap region defined in the Zephyr shared multi-heap when Espressif PSRAM support or similar external heap config options are enabled.

Overall, libcups is operational in terms of the main tests of array, clock, hash, language, HTTP, and IPP.

As for what remains to be done, further testing still needs to be done in all tests besides array, clock, hash, language, HTTP, and IPP. Special attention must be paid to the mbedtls functionality since it is a large chunk of dense code that was written entirely by myself and undoubtedly has errors strewn across different places. Further investigation should also be done in integrating Zephyr’s DNS-SD API with libcups.

## Zephyr Kernel

I have also had to make some changes to the Zephyr kernel in order to properly run applications using libcups and PAPPL. This mainly includes changing the default mbedtls repository to my own fork. This fork changes some config options and exposes a function necessary to libcups called `x509_crt_verify_name`. Additionally, I have added an option for enabling basic semaphores in the `sys_heap` functions and an option for supporting external heap memory for pthread stacks.

## PAPPL and Print Server Applications

The actual print server frameworks/applications like PAPPL are currently very much a work-in-progress. I have created a Zephyr module and test-application for running the PAPPL system that do compile. However, there is a lot more testing that needs to be done to address errors in PAPPL.

Another avenue for print server applications to go down is ippeveprinter, which emulates a print server and is mainly used to test communication over IPP. I have ran this and had success with DNS-SD advertisement but processing an IPP message still remains to be a challenge. I have also recently (as of the time of writing) done some more testing with ippeveprinter and struggled with processing `test.conf`, but succeeded with smaller config files. 

# Upstreaming

I have not been able to get the Zephyr Project organizers to host libcups and the other Zehpyr modules as official modules on the zephyrproject-rtos GitHub account. Therefore, it still remains a priority to move these modules to the official Zephyr account to make sure as many people as possible can have easy access to the code. As for right now, I will also begin the process of merging my version of the Zephyr kernel with the main Zephyr tree so that pthread stacks on the external heap and similar functionality is possible.

# Future Work

The main project that must be ironed out in terms of bugs and errors is PAPPL, since it is the main framework for running print servers. After that, the project should move toward real hardware starting with porting `ippusb_bridge`, a library for IPP over USB communication between USB printers and the print server. Then, hardware restrictions and constraints must be tested to discover what hardware is necessary for real printers running this software. Lastly, the eventual goal of integrating the print server directly with the rest of a printer’s embedded architecture on one board or SoC can be explored.

As for my future contributions to the project, I cannot promise very much in terms of code or debugging contributions due to schooling and research, but I will always make an effort to respond to and answer the questions of any open source contributors who are interested in this project. I will also aim to keep up on this project if it gets picked up by others in the open source community and continue to make some bits of contribution based on my experience with the project.

# Key Takeaways
Quickly, in terms of key takeaways of this project, I have first learned that software development environment matters a lot for ease of debugging and the general software development workflow. It has been exceptionally difficult to debug on Zephyr with a regular microcontroller and OpenOCD + GDB due to the bugginess with stepping (stepping one line often skips many), backtraces failing after exceptions, and the lack of hardware debugging (with the address bus) like in Vivado. It is also difficult to apply traditional software development and operating systems knowledge nor embedded systems knowledge since the level of abstraction is relatively far away from the hardware and low-level bare metal-like software, but Zephyr is still an RTOS designed for embedded devices and lacks many of the regular OS features. Lastly, I have also learned that managing complex systems like libcups by using these individual modules and tests is paramount for making progress. For example, when I was trying to get ippeveprinter to work on its own before doing more libcups testing, I would get enigmatic errors that would originate from all sorts of places and would be difficult to diagnose.

# Links:

You can find my project proposal, various commits, and repositories below. You can also look at the rest of my blog to look further into what I have done each week.

* [Zephyr Testing Application](https://github.com/HubertYGuan/zephyr-example) (My own repository)  
* [zlib](https://github.com/madler/zlib/compare/develop...HubertYGuan:zlib:zephyr)  
* [pdfio](https://github.com/michaelrsweet/pdfio/compare/master...HubertYGuan:pdfio:zephyr)  
* [mbedtls](https://github.com/zephyrproject-rtos/mbedtls/compare/zephyr...HubertYGuan:mbedtls:zephyr) (minor changes)  
* [libcups](https://github.com/OpenPrinting/libcups/compare/master...HubertYGuan:libcups:zephyr)  
* [PAPPL](https://github.com/michaelrsweet/pappl/compare/master...HubertYGuan:pappl:zephyr)  
* [Project Proposal](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

## Contact

The best way to contact me is via email ([hubertyguan@gmail.com](mailto:hubertyguan@gmail.com)). You can also see the rest of my GitHub account [here](https://github.com/HubertYGuan). 