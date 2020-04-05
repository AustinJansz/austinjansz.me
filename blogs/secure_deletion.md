# 3 Secure Deletion Tools You Should Know About

_Authors: Kinder Celemin, Austin Jansz, Ryan Schuette_

_Date: 2020-04-05_

## Summary

## Introduction

## Tools

### SRM

#### Description

SRM is a tool that is available with the secure_deletion toolkit offered on many UNIX-like operating systems (ex. Debian). This toolkit offers four tools, SMEM + SSWAP (secure memory and swap space wiping), SFILL (used to wipe unused space) and SRM [1]. As an alternative to the remove command (rm) which is a common command line tool that provides simple non-secure data removal, SRM is a secure deletion tool that offers users control over how the data is deleted and subsequently overwritten with support for common data deletion standards used by government agencies.

#### How it Works

SRM is much like the remove command such that it takes a pointer to a path and can use similar options such as:
- Directory
- Forceful removal
- Interactive removal (prompted)
- Verbose

With that said, SRM incorporates different overwrite methods to supplement data deletion. By default, it uses a simple single pass of zero’s through data blocks affected by the deletion. In many cases, this is sufficient for the security conscious individual as there is less than a 1% chance of recovering a single byte [2]. Many governments require a higher degree of certainty that the data has been destroyed. SRM offers the following methods to accomplish this task [3]:
- Openbsd: 0xFF pass, 0x00 pass, 0x00 pass
- DoD: 7 pass overwrite
- DoE: 2x random pass, ‘DoE’ pass
- RCMP: 0x00 pass, 0xFF pass, ‘RCMP’ pass
- Gutmann: 35 passes as pictured in the below figure

<center>
	<img src="https://austinjansz.me/images/gutmann_method.png" alt="Gutmann 35 Pass Method"/>
	<p><em>Gutmann Method [4]</em></p>
</center>

#### Advantages

- Works for file and directories
- Works for block devices (HDD/SSD)
- Multiple modes allowing for customization
	- Allows for the usage of multi-pass overwriting
- Nearly 100% (99.03%) chance of byte non-recovery with the most insecure method (simple/default) [2]
- Part of toolkit that offers a well-rounded sanitization experience


#### Disadvantages

- Not efficient for use on SSD
	- Secure Erase is a better alternative for flash deletion [3]
- Should be used in conjunction with SFILL (apart of the same toolkit) to clear temporary copies of the files that were not securely deleted
- Does not offer the same level of fine tuning as SCRUB
- Cannot solely remove all remnants of data
- Default mode is not secure enough for critical data removal and does not meet the standards of government organizations


#### Impact

SRM is a simple-to-use security tool that offers multiple modes for overwriting.  This includes modes used by different government agencies and created by data scientists. With that said, securely deleting data from the source media does not guarantee complete due to data remnants that can be found in alternate file streams  [6]. It has also been researched by Peter Gutmann that magnetic data storage can hold variations in magnetivity that could be used for data retrieval [4]. With this information, SRM can take advantage of the modes created by Peter Gutmann to securely delete data from a storage device however this tool cannot be used to completely clear all traces found elsewhere in the system such as memory on temporary folders. Fortunately this is a part of a toolkit that offers tools to close these alternate data discovery methods making this a very secure tool when used for its designed purpose.

#### Demonstation and Post-Operation Investigation

### SCRUB

### SHRED

### Conclusions

### References

[1] http://fsmsh.com/3063
[2] https://web.archive.org/web/20131208184307/http://www.h-online.com/newsticker/news/item/Secure-deletion-a-single-overwrite-will-do-it-739699.html
[3] https://www.systutorials.com/docs/linux/man/1-srm/
[4] https://www.cs.auckland.ac.nz/~pgut001/pubs/secure_del.html
[5] https://www.unix.com/man-page/debian/1/srm/
[6] https://www.sciencedirect.com/topics/computer-science/data-remanence



