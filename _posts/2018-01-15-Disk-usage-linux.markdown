---
title: Checking disk and file space usage in linux
layout: post
tags: linux unix
categories: linux
---

Here are some notes on the various ways to check disk space usage on linux:

# Disk usage of all (non-hidden) files and folders

Using the `du` command (disk use) with the `-s` (summarize) and `-h` (human-readable) options. 

```
du -sh *
```
Prints a list of all files and folders in the current directory. Folder/directory sizes given includes all the subdirectories and any files they contain.

**BONUS**: You can add the `-c` option to get the total size of all the files that this command lists. (I.e. `du -sch *`)

# Total disk usage including hidden files

This is useful, particularly if you are trying to get comparable outputs from the `quota` command (see below). Hidden files are not included by default, so we have to use wildcard matching like so:

```
du -sch .[!.]* * 
```

The first `.` matches files beginning with a dot, but we want to exclude `..`, since this would match the directory above (as in when you do `cd ..` etc.) and we don't want to include that. To exclude that pattern we add `[!.]`, in other words, match a single dot but not two in a row. The final asterisk is to match all the non-hidden files as before. 

Another way to do this is to specify the `-ahd1` set of options:

```
du -ahd1
```
However, this also includes the file `.`, which refers to the current directory. If you add the `-c` option to this  

# Sorted disk usage
The easiest way to do this is to just pipe the results into the sort command. To sort them numerically, we can just add the `-n` flag to sort.

```
du -sch * | sort -n
```
This will sort them numerically in ascending order. Have a look at the sort manual pages for more sorting opttions. 

# Disk usage by file system

`df` is a slightly different unix command which lists disk free space by file system. Typing it with no options will give a list of all mounted file systems, their total size available, how much space has been used, and their linux mount points. The `-h` option gives a more human-readable form. 

```
df -h
```

# Disk quota information

On systems where you have an allocated disk quota, the `quota` utility tells you how much of your quota has been used and how much you have available. The `-s` option gives a nice summary of disk quota:

```
quota -s
```

Outputs:

```
Disk quotas for user bob (uid 123456): 
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
mydomain:/disk/someserver/u1234//dvalters
                  2000M  15000M  15000M           12000       0       0  
```

The `space` output is sometimes replaced with `blocks`, which is a slightly less helpful measure. On POSIX systems, 1 block is befined as 512 bytes. `space` (or `blocks` tells you how much you have used, `quota` is your total quota allowance. If you are over the quota, an asterisk appears next to the number for space/blocks. 

# Other utilities

I have only covered the most common GNU/Linux utilities for monitoring/measuring disk usage. There are othe utilities available that have nicer outputs by default, such as: ncdu, freespace etc. 

