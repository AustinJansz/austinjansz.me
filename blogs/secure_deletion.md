# 3 Secure Deletion Tools You Should Know About

_Authors: Kinder Celemin, Austin Jansz, Ryan Schuette_

_Date: 2020-04-05_

## Summary

## Introduction

## Tools

### SRM - Secure File Deletion

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
	<img src="https://austinjansz.me/images/secure_deletion/1-2.png" alt="Gutmann 35 Pass Method"/>
</center>

#### Advantages

- Available by default on some operating systems such as Security Fedora
- Works for file and directories
- Works for block devices (HDD/SSD)
- Multiple modes allowing for customization
	- Allows for the usage of multi-pass overwriting
- Nearly 100% (99.03%) chance of byte non-recovery with the most insecure method (simple/default) [2]
- Part of toolkit that offers a well-rounded sanitization experience



#### Disadvantages

- Time intensive, especially for methods with a high amount of passes
- Not efficient for use on SSD
	- Secure Erase is a better alternative for flash deletion [3]
- Should be used in conjunction with SFILL (apart of the same toolkit) to clear temporary copies of the files that were not securely deleted
- Does not offer the same level of fine tuning as SCRUB
- Cannot solely remove all remnants of data
- Default mode is not secure enough for critical data removal and does not meet the standards of government organizations



#### Impact

SRM is a simple-to-use security tool that offers multiple modes for overwriting.  This includes modes used by different government agencies and created by data scientists. With that said, securely deleting data from the source media does not guarantee complete due to data remnants that can be found in alternate file streams  [6]. It has also been researched by Peter Gutmann that magnetic data storage can hold variations in magnetivity that could be used for data retrieval [4]. With this information, SRM can take advantage of the modes created by Peter Gutmann to securely delete data from a storage device however this tool cannot be used to completely clear all traces found elsewhere in the system such as memory on temporary folders. Fortunately this is a part of a toolkit that offers tools to close these alternate data discovery methods making this a very secure tool when used for its designed purpose.

#### Demonstation and Post-Operation Investigation

##### Baseline Environment

A FAT partition was loaded from an image with a single file called readme.txt.

Commands:

- mmls: read the contents of the image to retrieve partition data
- dd: extract 2 copies of the partition, one to be used for standard removal and the other to be used with srm removal methods
- mount: mount the partition data to a folder for alteration
- sync: apply the changes back to the extracted partition

##### Removal (rm) 

Command: 

- rm -f removal_demo/readme.txt (Execution time 0.003s)

Retrieval commands:

- istat -r fat_partition_std 3 (entry numbers can be iterated to find files)
- icat -f fat fat_partition_std 3 | head -n 10 (read the first 10 lines of the found file)

<center>
	<img src="https://austinjansz.me/images/secure_deletion/1-5-1.png"/>
</center>

##### Secure Removal (srm)

Commands (time command added to show duration of process): 

- cp fat_partition_srm fat_partition_modename
- mkdir modename && mount fat_partition_modename modename/
- srm -f readme.txt 
- srm -f --dod readme.txt
- srm -f --rcmp readme.txt
- srm -f --gutmann readme.txt

Retrieval commands:

- istat -r fat_partition_modename 3 (entry numbers can be iterated to find files)
- icat -f fat fat_partition_modename 3 | head -n 10 (read the first 10 lines of the found file)

<center>
	<img src="https://austinjansz.me/images/secure_deletion/1-5-2-3.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/1-5-4-5.png"/>
</center>

By auditing the File Allocation Table attributes, each of the methods begin to separate from one another. The simple method wipes the data found and removes the name pointer however the rest of the name can still be found. The DoD method is able to completely remove the file contents from the system however the name is still in place and can be easily found. The RCMP method is similar to the simple method in that the original contents are removed and the namespace becomes unallocated however the data area is replaced by ‘RCMP.’ Lastly the Gutmann method showed a very similar outcome to the simple method however took significantly longer to complete the rest of its security passes.

Names of the files for each overwrite method (when using SRM on a per file basis), are likely to be discovered however both the simple and Gutmann methods remove the first letter pointer (in the FAT table) as well as the data. These options should be considered depending on the level of deletion required for the data. When the discovery of filenames must be prevented, SRM can be applied to the entire image which would remove the file system attributes.

The two methods most likely to be fingerprinted are the DoD and RCMP. The DoD method does not deallocate the file area, and RCMP explicitly adds the letters RCMP to the file area. This is likely due to the chain of custody required by different government agencies. This is similar to the action of the DoE method adding its abbreviation to the file space.

The execution time is directly related to the pass count used by each method, the standard removal feature, which took 0.003s which is assumed to be much faster than srm methods as it does require the performing of an overwrite on the dataspace. 

- Standard RM command
	- no overwrite passes; 0.003s
- The simple method
	- 1 pass; 0.0013s; nearly double RM
- The DoD method
	- 7 passes; 0.025s; 8 times RM
- The RCPM method
	- 3 passes; 0.019s; 6 times RM
- The Gutmann method
	- 35 passes; 0.069s; 23 times RM

A logarithmic relationship was found for the execution time of these methods showing that if this secure deletion is used, increasing the passes past a factor of 3 has a lessened effect on the execution time of the removal.

<center>
	<img src="https://austinjansz.me/images/secure_deletion/1-5-6.png"/>
</center>

### SCRUB

### SHRED

### Conclusions

### References

- [1] http://fsmsh.com/3063
- [2] https://web.archive.org/web/20131208184307/http://www.h-online.com/newsticker/news/item/Secure-deletion-a-single-overwrite-will-do-it-739699.html
- [3] https://www.systutorials.com/docs/linux/man/1-srm/
- [4] https://www.cs.auckland.ac.nz/~pgut001/pubs/secure_del.html
- [5] https://www.unix.com/man-page/debian/1/srm/
- [6] https://www.sciencedirect.com/topics/computer-science/data-remanence



