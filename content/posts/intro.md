---
date: '2025-05-27T15:17:36-07:00'
draft: false
title: 'Porting Printing to Zephyr'
---

![OpenPrinting and Zephyr](/images/openprintingzephyr.png#center)

# About the Project

Printers are a technology that we often think should “just work,” but in reality, there are often many hoops to jump through when dealing with a printer’s drivers. It is a common experience when dealing with printers to struggle to get a document printed when trying every which way on an OS like modern Windows, but then all the problems suddenly disappear when using an Apple device. This is in large part thanks to Apple’s AirPrint, which is the oldest and most widely supported driverless printing protocol. The driverless nature is important as it prevents the issue of having printer drivers that may not be updated to support the user’s current operating system. However, AirPrint is not an open standard, and only Apple operating systems are designed with it in mind. This leads us to IPP Everywhere, a driverless printing protocol that is an open standard, and its pivotal role in the modern open-source printing stack developed by OpenPrinting. This would theoretically allow any operating system that supports the printing stack to print, as long as the printer supports IPP Everywhere.

However, IPP Everywhere is not nearly as common as AirPrint, and printers that do not support any wireless printing bring even more complications. This is where our project steps in by bringing driverless wireless printing to all functional printers. Through leveraging OpenPrinting’s printing stack, consisting of CUPS, libcupsfilters, PAPPL, etc., one can already use a single board computer (SBC) such as a Raspberry Pi to run a printing server for such printers. However, the footprint of a Raspberry Pi is quite large, and such a computer is much more expensive than a simple microcontroller. This project aims to reduce the size of machines required for running these driverless printing servers to simpler SBCs and microcontrollers running real-time operating systems, such as Zephyr, in this case. Not only will this allow more legacy printers to become wireless, but this could also aid future wireless printer development by allowing printers to easily come as driverless out of the box.

Proposal, with timeline, that is subject to updates over time: [https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing](https://docs.google.com/document/d/1cYL6S2JSkzY0ln1w_s3qwpm85-keB9jaVjGSwyPg5NM/edit?usp=sharing)

# Design Diagram

[https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w](https://excalidraw.com/#room=0a35b04c6340f86386ea,xwV6ErRli602L8FTUikm2w)

---