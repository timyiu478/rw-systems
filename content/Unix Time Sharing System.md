---
tags:
  - operating-systems
reference: obsidian://open?vault=systems&file=papers%2Funix_time_sharing_system.pdf
title: Unix Time Sharing System
---
## Takeaways

* **Administrative delegation** is one of the differences between Unix layering and DNS layering
* The implementation of the mount system call

## Q&A

**Q. What things in UNIX are named?**

* file
* user

**Q. Why set-user-ID?**

It provides for privileged programs which may use files inaccessible to other users. For example, a program may keep an accounting file which should neither be read nor changed except by the program itself.

**Q. How does naming in UNIX compare to naming in DNS? How do layering and hierarchy apply (if at all)?**

- UNIX filesystem  
	* Layering mainly provides naming context and access control  
    * No strong notion of administrative delegation between directories
- DNS  
    * Layering primarily enables **administrative delegation** and distributed authority  
    * Each zone (subtree) can be delegated to different organizations  

**Q. n = read(filep, buffer, count); when n == 0?**

The file descriptor's read pointer reached the end of the file.

**Q. What is the purpose of the open() syscall?**

The open() syscall is used to use the pathname to search the corresponding *i-number*. Then it will store the *i-number*, *device*, *read/write pointer* into the system table indexed by the file descriptor.

The *i-number* is used to find the corresponding *i-node*.

**Q. Device number vs subdevice number**

The device type indicates which system routine(code) will deal with I/O on that device.
* for look up the driver table

One driver can handle many similar devices. The subdevice number tells the driver which one you're talking to.
* e.g. tty driver for all pesudo-terminals

**Q. Explain the implementation of the mount syscall**

The pathname of the mount point refers to the *i-number* of the special file that the file is specified in the *mount* syscall. 

The path resolution of the mount point can find the root directory of the correct file system in the *kernel's Mount table*.

Ref: https://landley.net/toybox/doc/mount.html

