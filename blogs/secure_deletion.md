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
	![Gutmann Method](https://austinjansz.me/images/gutmann_method.png)
	
	Gutmann Method [4]
</center>

#### Advantages

- Works for file and directories
- Works for block devices (HDD/SSD)
- Multiple modes allowing for customization
	- Allows for the usage of multi-pass overwriting
- Nearly 100% (99.03%) chance of byte non-recovery with the most insecure method (simple/default) [2]

#### Disadvantages

- Not efficient for use on SSD
	- Secure Erase is a better alternative for flash deletion [3]
- Should be used in conjunction with SFILL (apart of the same toolkit) to clear temporary copies of the files that were not securely deleted
- Does not offer the same level of fine tuning as SCRUB





