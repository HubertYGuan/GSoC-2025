---
date: '2025-07-06T20:17:23-07:00'
draft: false
title: 'Week 3: PDFio and zlib Updates'
---
This week marks the point where zlib and PDFio are wrapped up for now. This update is fairly short since there has not been a lot of new stuff going on this week, since most of the work has been with testing, and I celebrated Independence Day last Friday.

I can now proudly say that zlib passes the provided `example` and `infcover` tests at least on the ESP32. Luckily, I did not have to go digging around in GDB very much since the problems seemed to just be in the compile options and library choice. Now that I have enforced the `_POSIX_C_SOURCE` parameter and switched to newlib, things have been going fine so far.

As for PDFio, it does compile fine with newlib, but that is basically where it is going to be left at for now. Since the project is not focused on manipulating, rasterizing, etc. PDF files on Zephyr, there is not much of a need for PDFio for now, and I can move forward with libcups. However, if given time in the future, I could revisit the prospect of doing rasterization on Zephyr using some stronger hardware if there is a reasonable use case for Zephyr over embedded Linux with something like a Raspberry Pi.

As for the next steps, libcups should hopefully not be too much of a hassle since I have already ported a library with zlib. I also planned to port PAPPL after that, but it seems like we may only have version 2.0b1 so far, so the code will be pretty experimental, and I may have to work on something different, like ippusb\_bridge, while some of the software gets updated.

As always, the design docs and proposal are listed below:  
Design Diagram:  
[https://excalidraw.com/\#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)

Project Proposal:  
[https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w\_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)