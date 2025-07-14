---
date: '2025-07-14T11:52:45-07:00'
draft: false
title: 'Week 4: libcups and Dependencies'
---
I want to start by saying that porting libcups is significantly more complex than porting zlib or PDFio, and I was definitely wrong in saying that I could port it quickly. Although setting up the build scripts and environment is relatively simple, like with zlib and PDFio, I also have to make significant additions and changes to the source code in multiple libraries.

# Mbed TLS

The first challenge comes in the form of Zephyr’s TLS library, Mbed TLS. Since the design methodologies of OpenSSL, GNU TLS, and Mbed TLS differ, I first had to modify Mbed TLS to expose previously static functions (mainly for `cupsAreCredentialsValidForName`). Since the contributors didn’t want users to have access to functions like `x509_crt_verify_name` that are practically needed to implement some of the libcups functions (maybe due to security concerns), this sort of modification is unofficial and can be quite murky.

Additionally, I am still in the process of refactoring the GNU TLS implementation of libcups functions to Mbed TLS, which is not trivial since there are thousands of lines to sift through and there are a whole swath of differences between the two libraries.

# mDNS Responder

After finishing up the Mbed TLS implementation, I still have to worry about mDNS Responder since, although it is supported by libcups, the Zephyr fork is missing many different types in the original version that libcups uses. I do not know the exact reason as to why mDNS Responder is missing a large amount of code here, but I will try first to implement the required types and functions for libcups.

# Other Notes

There are also some types normally defined in libc headers that are not defined in newlib, picolibc, or the Zephyr core library, so I will also try to modify my libc after dealing with the above two dependencies. I will maybe also have to come back to libcups if I ever also import libpng or libpam. USB printing also needs to be supported, so I will then use Zephyr’s USB libraries in place of libusb as well.

In terms of the timeline, I definitely will be pushing the other ports back due to the amount of source code that I need to go through for libcups. I estimate that I may need another couple of weeks, which means that I should at least have a mostly functional version of libcups running by the midterm evaluation, and hopefully, some of PAPPL should be done since it has no new dependencies.

You can check the documents below for a recap and some further details:  
Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)