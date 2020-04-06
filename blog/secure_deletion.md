# 3 Secure Deletion Tools You Should Know About

_Authors: Kinder Celemin, Austin Jansz, Ryan Schuette_

_Date: 2020-04-05_

## Summary

Three command-line based tools used for the secure deletion of files were investigated for their ability to successfully remove content from systems. SRM, Scrub, and Shred were investigated for their performance on live system files as well as images of systems. Through research and trial runs of this software, it was found that each had common advantages and disadvantages. SRM offered multiple common overwriting methods used by government agencies such as the United States Department of Defense (DoD) and Canada's Royal Canadian Mounted Police (RCMP) however did not allow the user to fine tune the process. Scrub allows for the overwriting of files using patterns similar to SRM; it too is limited to standard patterns. Where Scrub lacks, however, is in its ability to wipe space that has already been deallocated. Lastly, Shred is a core GNU utility that is widely available and offers a great deal of customizability and user control and is only victim to limitations shared by the other two tools. These tools share the limitation of lacking complete data remnant removal. Files often leave copies of themselves while being processed in not only temporary directories but also in memory. This requires either a full device wiping or the use of additional tools such as SMEM, SSWAP and SFILL (which come packaged with SRM). All three tools offer multiple pass overwriting and high level of security, where SRM stands out with its packaged software and Shred with its high level of user customizability.

## Introduction

The following report was created as an investigation into the secure deletion of data in the form of files and devices. Digital Forensics is a field where data retrieval and secure data destruction are partnered in their use by law enforcement and government agencies. In many cases critical data must be destroyed in the event of a system being decommissioned and its data must be destroyed to prohibit any adversaries from accessing its data.

This report is broken up into three section, one for each SRM, Scrub, and Shred and is then segmented the following stages:
1. An intro to the tool and why it was developed
2. A brief overview of how it works
3. The specific advantages and disadvantages of the tool
4. Its impact and how well it accomplishes its objectives
5. Lastly, a demonstration and post-operation investigation (retrieval attempts)


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
	<img width="50%" src="https://austinjansz.me/images/secure_deletion/1-2.png" alt="Gutmann 35 Pass Method"/>
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

SRM is a simple-to-use security tool that offers multiple modes for overwriting.  This includes modes used by different government agencies and created by data scientists. With that said, securely deleting data from the source media does not guarantee complete due to data remnants that can be found in alternate file streams  [5]. It has also been researched by Peter Gutmann that magnetic data storage can hold variations in magnetivity that could be used for data retrieval [4]. With this information, SRM can take advantage of the modes created by Peter Gutmann to securely delete data from a storage device however this tool cannot be used to completely clear all traces found elsewhere in the system such as memory on temporary folders. Fortunately this is a part of a toolkit that offers tools to close these alternate data discovery methods making this a very secure tool when used for its designed purpose.

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
- icat -f fat fat_partition_std 3

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
- icat -f fat fat_partition_modename 3

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
	<img width="50%" src="https://austinjansz.me/images/secure_deletion/1-5-6.png"/>
</center>

### SCRUB

#### About

Scrub is another tool that is included in most Linux distributions that again handles secure file deletion. The main difference between this utility and Secure Remove is that rather than overwriting renaming and truncating, this will just write different patterns to the files in question.

#### How it works

Scrub works by iterating over the files subject to deletion, while also writing different patterns in hopes to make recovering or undeleting the data much more difficult. Scrub has three distinct methods of operation, those being special file, regular, and directory mode.
    
Special file mode will target the specific file corresponding to the entire disk, then will scrub that file using the method explained above, which in turn destroys all the data. This is considered the most effective method of the three.

File mode will target only the file and will scrub it completely with the option to also do the same to the directory entry, which would completely destroy it.

Lastly, directory mode will fill the entire filesystem with files until there isn't any more space left, then each and every file is scrubbed as it would be in the regular mode. This mode has a distinct flag that must be noted in order to use it to make sure it's something you really want to do.

<center>
	<img src="https://austinjansz.me/images/secure_deletion/2-2.png"/>
</center>

#### Impact

Ideally, the impact of this would be that the files that were “securely deleted” with this utility, shouldn’t be recoverable. Methods of undeleting should be unsuccessful therefore making the file deleted forever.

#### Advantages

- Multiple standards of deletion
- Great default settings

#### Disadvantages

- Limited by OS
- Limited by file system
- Need to specify if file should be deleted or just the contents

#### Demonstation and Post-Operation Investigation

<center>
	<img src="https://austinjansz.me/images/secure_deletion/2-3-1-2.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/2-3-3.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/2-4-1-2.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/2-4-3-4.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/2-4-5-6.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/2-4-7-8.png"/>
</center>

### SHRED

#### About

Again, shred is another utility or tool that is available to use in most distributions of Linux. This one is a little different however because while this tool can delete files but it is entirely optional. Shred is meant to make sure that the contents of a file cannot be recovered. Or at least it will provide the least likely able to do so.

#### How it Works

Shred works a little differently than the previous two secure deletion utilities. Shred aims to make sure that the contents of a file can not be accessed, and it does this by overwriting the contents of the file over and over. The default number of overwrites this process will do is three, however, this can be changed to any custom value. Of course, the more times a file is overwritten the more secure you can be in the idea that the original contents will not be recovered.

Now with all that being said, none of this process actually deletes the files in question. To actually delete the file after the overwriting process a specific option must be specified in the command before execution.

#### Impact

Ideally, the impact of this would be that the files that were “securely deleted” with this utility, shouldn’t be recoverable. Methods of undeleting should be unsuccessful therefore making the file deleted forever.

#### Advantages

- Great for wiping files
	- Standard delete uses 3 iterations of different methods
- Fully customizable

#### Disadvantages

- Shred leaves behind traces of deleted files

#### Demonstration and Post-Operation Investigation

Similar details before and after deletion. We can see where the file is and it’s content location

<center>
	<img src="https://austinjansz.me/images/secure_deletion/3-1-1-2.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/3-2.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/3-3-1-2.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/3-3-3-4.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/3-3-5-6.png"/>
	<img src="https://austinjansz.me/images/secure_deletion/3-3-7-8.png"/>
</center>

### Conclusions

Overall we can see that these are only three utilities that can be used to securely delete files from your system. While there are many many more, they all do the same thing in different ways, but that doesn't mean these tools are foolproof. Each of these tools makes it much more difficult to recover the data, but it isn't necessarily an impossible task. There are more steps that should be taken in order for you to be sure that your data is gone for good.

File systems hold on to temporary files and other data in the background that isn't necessarily out in the open. So other tools must be used in order to make sure the data is gone. First off it is recommended to wipe the entire drive rather than just individual files to make sure those temporary files aren't there. As an example, SRM has sister tools that can be used to wipe memory and unallocated data that SRM cannot do directly. After wiping the drive, memory, and unallocated data, you can rest assured that the data you had is gone for good.


### References

- [1] Richmond, G. (2008, November 29). Shred and secure-delete: tools for wiping files, partitions and disks in GNU/Linux. In Free Software Magazine. Retrieved from [fsmsh.com/3063](http://fsmsh.com/3063)
- [2] Secure deletion: a single overwrite will do it (2009, January 17). In H-Online. Retrieved from [www.h-online.com/newsticker/news/item/Secure-deletion-a-single-overwrite-will-do-it-739699.html](https://web.archive.org/web/20131208184307/http://www.h-online.com/newsticker/news/item/Secure-deletion-a-single-overwrite-will-do-it-739699.html)
- [3]  srm (1) - Linux Man Pages (n.d.). In SysTutorials. Retrieved from [www.systutorials.com/docs/linux/man/1-srm/](https://www.systutorials.com/docs/linux/man/1-srm/)
- [4] Gutmann, P. (1996, July 25). Secure Deletion of Data from Magnetic and Solid-State Memory. In University of Auckland. Retrieved from [www.cs.auckland.ac.nz/~pgut001/pubs/secure_del.html](https://www.cs.auckland.ac.nz/~pgut001/pubs/secure_del.html)
- [5] Data Remanence (n.d.). In ScienceDirect. Retrieved from [www.sciencedirect.com/topics/computer-science/data-remanence](https://www.sciencedirect.com/topics/computer-science/data-remanence)
- [6] scrub (1) - Linux Man Pages (n.d.). In SysTutorials. Retrieved from [www.systutorials.com/docs/linux/man/1-scrub/](https://www.systutorials.com/docs/linux/man/1-scrub/)
- [7] shred (1) - Linux Man Pages (n.d.). In SysTutorials. Retrieved from [www.systutorials.com/docs/linux/man/1-shred/](https://www.systutorials.com/docs/linux/man/1-shred/)


