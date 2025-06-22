---
date: '2025-06-21T18:05:27-07:00'
draft: false
title: 'Week 1: PDFio and zlib'
---

Now that this inaugural week is done, I have some time to reflect on what I’ve learned and what did/did not go well. I’ve learned that I definitely work best at the beginning of my day, as I can basically devote my time entirely to the project for a few hours before dealing with other things and working more intermittently throughout pretty much the rest of the day. However, there are some things that can distract me from keeping up the momentum for those first few hours. Therefore, I think that starting work earlier and adapting my environment to be free of those distractions is something I should look towards.

This week started with porting PDFio, which acted mainly as my transition back into working with Zephyr and now with the Zephyr CMake build system. PDFio is a simple, lightweight library for interacting with PDF files through reading/writing, interfacing with objects/streams, etc. It is one of the main dependencies of libcups, which will be one of the major milestones of this project, as it plays a pivotal role in IPP communication with printers.

# PDFio
Luckily, PDFio does not seem very tied to a specific OS, so most of the files looked like they would compile fine on a first overview. However, the main thing that I will have to pay attention to going over the codebase a second time is places to use Zephyr’s file systems API, since we’re likely not going to be actually writing PDF files to disk and instead storing them for use somewhere in memory in a sort of virtual file system.

Besides that, nearly all of the difficulty comes when presented with `pdfio-stream.c`, since that is the file responsible for compressing and decompressing streams in PDFs using the inflate and deflate algorithms, specifically using zlib. Although Zephyr supports LZ4 compression, to my knowledge, LZ4 is not used much with PDF streams, possibly due to its difficult-to-manage block structure. 

# zlib
I decided, with the help of my mentors, to then port zlib as well. Luckily, zlib is made up of basically all user-level C, meaning that it compiles fine after changing the build system over to Zephyr CMake. However, we may have to come back later to see the performance consequences of porting zlib, seeing as it is intended for general (not necessarily resource-constrained on a microcontroller) use. Nevertheless, zlib seems to work fine with the limited testing that I have conducted with basic compression using inflate and deflate in this [project](https://github.com/HubertYGuan/zephyr-example). However, one additional thing I should also set up is a basic testing framework that at least runs test applications sequentially as kernel threads so that I can easily build and run tests ported over from the original codebase. Therefore, although the code may work fine on Zephyr, the port for zlib is not entirely done due to the lack of adequate testing.

# Timeline and Next Steps
I will spend some time implementing those changes regarding testing and file i/o within the next week, so hopefully I will remain on schedule with porting PDFio by the second week. I will make sure to focus on those things so that I can then build off of that to implement libcups and possibly some other useful libraries for, e.g., PNGs, JPEGs, etc., within the next few weeks. The timeline will also be updated on the project proposal as I get a better sense of what the best next steps look like as I implement the rest of PDFio and libcups.

I am also thinking of slightly shifting the ordering within the current timeline, since I see that CUPS, PAPPL, and libcupsfilters may not be entirely necessary to get at least some barebones communication over IPP with a USB (“gadget mode”) printer. Therefore, I think that the foundation of libcups and also ippusb_bridge would be great to focus on to prove that SBC-driven printer communication would be possible.
Conclusion
Overall, it has been an eventful first week, mainly to get used to the development environment, but also to make some progress on porting the first libraries. As always, you can find links to the project proposal and design diagram below.

Design Diagram:
https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w

Project Proposal:
https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing



---