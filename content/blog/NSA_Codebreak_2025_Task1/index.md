---
title: "NSA Codebreak 2025 Writeup - Task 1"
date: 2026-01-21
tags: ["CTF", "NSA"]
categories: ["Computer Science"]
description: "Forensic analysis on suspicious device"
draft: false
---

# Task 1 - Getting Started - (Forensics)

{{< figure src="badge1.png" alt="Badge" caption="Badge as proof for completing task 1" >}}

## Problem Statement

You arrive on site and immediately get to work. The DAFIN-SOC team quickly briefs you on the situation. They have noticed numerous anomalous behaviors, such as; tools randomly failing tests and anti-virus flagging on seemingly clean workstations. They have narrowed in on one machine they would like NSA to thoroughly evaluate.

They have provided a zipped EXT2 image from this development machine. Help DAFIN-SOC perform a forensic analysis on this - looking for any suspicious artifacts.

Provide the SHA-1 hash of the suspicious artifact.

## Solution

### Task 1 - Setup Workspace

An EXT2 image file stores the file system information on a Linux system. The goal of this challenge is to use the provided EXT2 image to find a suspicious file lurking somewhere in the extract file system.

After downloading the zip file, it is extracted to obtain the EXT2 image file called `image.ext2`.

> One approach is to mount the image to your system and navigate through the system like a USB. This can be dangerous, as it can expose malicious files to your system or could tamper with the file system. Therefore, it is recommended to open mount the image in a virtual box in read-only mode with the command:
>
> ```bash
> sudo mount -o ro,loop image.ext2 /mnt/forensic/
> ```

### Task 2 - Snooping

The first step is to generate a map of the system to get a general idea of where a suspicious file could be hiding:

```bash
fls -r image.ext2
```

This returns a list of indicating everything in the file system. Each entry begins with a number of plus signs indicating if it is nested in the directory above it. Next is the file type identifier, followed by the inode and the file name.

I used this command to look for `$OrphanFiles` (files that had been deleted, but some data still remains), files with weird names, or important directories where executables and authentications are often found.

The following steps were taken:

* Look for files under the inode of `$OrphanFiles`.

* Create a list of files names and their inode by looking at files names I thought were weird. The criteria for weird, included names that weirdly misspelled, files with names I could not understand, and files with short names.

* Create a list of directory names and their inode of directories using `fls -r image.ext2 | grep d|d`, making note of directories commonly containing executables like `bin` or authentications like `apk`.

Finding no orphaned files remaining, I began looking at each directory to cross-compare with the list of weird files. For files that matched, I then read the content of the file. I used the following commands: 

```bash
fls image.ext2 DIRECTORY_INODE
icat image.ext2 FILE_INODE
```

In my image, a weird file named `wxmzuoevnc` in the `\etc\apk\keys` directory. This directory store RSA keys used by Alpine Linux. These are files that usually end in `.rsa.pub` and contain a public key. The file `wxmzuoevnc` instead has no extension and contains lookup information on another device. This does not align with the other entries and thus identified as the suspicious file.

### Task 3 - Preparing Solution

With the artifact identified, the solution requests to pipe the file into the `sha1sum` function.