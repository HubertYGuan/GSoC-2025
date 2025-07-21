---
date: '2025-07-21T16:56:01-07:00'
draft: false
title: 'Week 5: libcups Continued'
---
This week was again spent mostly on refactoring libcups to support Mbed TLS, and besides some slight issues, I should be able to finish implementing it soon and then make some additional header files (or forks) to implement the rest of the functionality needed from libc and mDNS Responder so that I can get a test application to compile. Another thing to note is that I feel like I have learned a lot about TLS and cryptography just from working with Mbed TLS. I used to not pay too much attention to the field of cryptography in my work and research, but now I realize more and more that cryptography is definitely relevant to this field of embedded systems and my research in computer architecture.

# Some Issues of Importance

Although Mbed TLS matches the functionality of GNU TLS for the most part, there are still some features that GNU TLS/OpenSSL support that Mbed TLS currently does not support. One of them is storing the extended key usage (purpose in GNU TLS) extension in CSRs (Certificate Signing Requests), at least from what I have seen. Since the current libcups API expects us to have CSRs with this functionality, the CSR functionality is currently left in a work-in-progress state.

Another factor to take into account is memory usage. Since my ESP32 has only about 170KB of memory to work with, I have halved all of the larger 65536-byte buffers so far, but I will have to see in testing what other things I may have to change to fit memory constraints.

# Timeline

I aim to have a compiled version of a test application done later this week, complete with the aforementioned changes. However, I will have to see how much extra time I will need to complete testing and debugging. If I were to estimate now, it would likely stretch on for another week, and I may have to come back to it later if there are some issues not found in the testing. Again, this means that libcups should be in a presentable state by the time the midterm evaluation comes, and I will also be working on PAPPL during that time.

As always, you can check the documents below for a recap and some further details:  
Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)