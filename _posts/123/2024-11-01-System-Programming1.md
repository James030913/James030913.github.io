---
title: "COMP3438 System Programming"
date: 2024-11-01 20:00 +0800
categories: [PolyU]
tags: [C, OS]
author: James
---

# Lec 4 Intro to device driver

##  What is device driver?

Device driver is a special kind of library, which can be loaded into OS kernel, and links the user program with IO devices.

## OS infrastructure

#### Related to devices driver:

**Unix system architecture**  
**File subsystem and its relations with char/block device driver tables**  
**Char/block device driver tables**

#### Unix System Architecture 
![Desktop View](/assets/img/comp3438/UnixSystemArch.png){: .normal }

#### File subsystem & char/block device driver tables
![Desktop View](/assets/img/comp3438/cbdevice.png){: .normal }

#### Char/block device driver tables
![Desktop View](/assets/img/comp3438/cbtable.png){: .normal }

#### Advantage to Sperate device driver from OS 

For OS designer:  
Devices may not be available when OS is desiged.  
No need to worry about operate device eg. set up register, check status.  
Focus on OS itself in order to provide interface for device drivers development.  

Device driver designers:  
No need to worry about how I/O managed in OS.  
Focus on implementing functions of devices with device-related commands
following the generic I/O interface.  

#### Devices and Files

use `mknod <file_name> <c or b> <major_number> <minor_number>` to create device file.  

#### Type of device

Two type of device driver look like    
![Desktop View](/assets/img/comp3438/twotype.png){: .normal }  

##### Block Device Driver
Communicate with the OS through a collection of fixed size buffers  
![Desktop View](/assets/img/comp3438/block.png){: .normal }  
Driver is invoked when requested data is not in the cache; when buffers in the cache have been changed and must be written out (write back to the devices).  
By using buffer cache, block
drivers are insulated from
the many details of user
requests; only need to handle
requests from the OS to fill or
empty fixed size buffers.

##### Character Device Driver
![Desktop View](/assets/img/comp3438/char.png){: .normal }  
Character devices can handle I/O requests of arbitrary size (Support any type of device)  
Be used to handle data a byte at a time (e.g. keyboard); or work best with data in chunks smaller or larger
than the standard fixed size buffer used by device driver (e.g. ADC)

##### Major Differences
***block:***  only interacts with buffer cache  
***char:***  directly interacts with user requests from user
processes  
* I/O requests are directly passed (essentially unchanged) to the drivers
from the user processes  
* Character driver is responsible for transferring data directly to/from
the kernel memory space and the user memory space.





