---
date: '2025-07-28T21:10:09-07:00'
draft: false
title: 'Week 6: More Progress on libcups'
---
Although libcups has been the most challenging project to port and has taken a lot longer than I initially thought, I feel like I am finally a lot closer to getting the library operational. Even though I am not quite done with getting everything prepared for compilation unlike what I said I would do earlier, I have gone through and added code for all of the parts where Mbed TLS is needed.

# More on Mbed TLS

Switching to Mbed TLS has been the major challenge to overcome throughout the whole process of porting libcups, which is why I think getting even a first draft of all the code out to replace GNU TLS and OPENSSL is a significant milestone. However, I am definitely not done yet, since I will still have to go through intensive testing. As for the earlier issue of Mbed TLS’s extended key usage storing capabilities for CSRs, I have only been able to find `cupsSignCredentialsRequest` used in tests and the `cups-x509.c` tool. Since I think that the CLI programs of CUPS are not as essential right now (compared to getting the applications for the CUPS local and sharing servers running), I am fine with changing the API for now and changing the tool code later.

Another topic regarding Mbed TLS to pay attention to is the longevity of the code that I have written using Mbed TLS for libcups. Mbed TLS is currently in beta development of version 4.0.0, meaning the Zephyr port will likely also update in some time. This means that we will have to anticipate updating the code I just wrote in some time since I have been relying on legacy APIs to provide the proper functionality for things like RSA and ECDSA signatures.

# Timeline

I still aim to get libcups mostly operational by the midterm evaluation of the week of August 4th. Getting the extra headers from MDNS Responder and libc in place to satisfy the missing definitions libcups wants should not be too much of a hassle. However, getting past all of the tests is likely going to be a long task since libcups is still in a beta state and not all of the tests pass on my local machine to start with. However, there is always the option of keeping track of the unfinished parts which may not be entirely essential, and then revisiting at a future time when libcups is more mature to try to tackle the problem again. Lastly, I have updated the proposal timeline to reflect the current trajectory and plans for you to see below.

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2)