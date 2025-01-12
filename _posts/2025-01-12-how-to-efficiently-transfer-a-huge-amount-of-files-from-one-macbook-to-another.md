---
layout: post
title: 'How to efficiently transfer a huge amount of files from one MacBook to another'
date: '2025-01-12T12:00.000+02:00'
author: Torben Hansen
tags:
- macbook
- copy
- files
modified_time: '2025-01-12T12:00.000+02:00'
permalink: /2025/01/2025-01-12-how-to-efficiently-transfer-a-huge-amount-of-files-from-one-macbook-to-another.html
---

As a web developer working on various projects, I often deal with project sources that include **tens or even 
hundreds of thousands of files**. This happens because modern development environments, especially those using 
Node.js or PHP, rely on extensive package ecosystems. These packages, installed from source, often contain a 
massive number of files—ranging from code libraries and dependencies to metadata and configuration files.

Managing and transferring such large file collections can become a timeconsuming task, especially when you 
get a new MacBook and have to move files from the old MacBook to the new one. In this article, I’ll share the 
most efficient method I’ve found to move these huge amounts of files between MacBooks without wasting hours 
or days for the transfer projess.

## Traditional approaches are timeconsuming

Transferring files between MacBooks can be done using e.g. **AirDrop**, **Migration Assistant**, **direct 
Thunderbolt connection**, or **network share**. While these methods are convenient for smaller datasets, they 
become inefficient when handling millions of files. The sheer number of individual file operations leads to 
slow performance, making these options less practical for large-scale transfers.

A more effective approach involves creating an **uncompressed tar archive** (tarball) of the files, which can 
significantly expedite the transfer process.

## Why Use an Uncompressed Tarball?

Creating an uncompressed tarball consolidates numerous files into a single archive without the overhead of 
compression. This method offers several advantages:

* Handling a single large file is often more efficient than managing millions of individual files, reducing the strain on the file system during transfer.
* By avoiding the compression step, you save processing time, and the data can be transferred more quickly, especially over high-speed connections like Thunderbolt or Ethernet.
* A single archive file is easier to manage and less prone to errors during the transfer process compared to numerous individual files.

## Steps to create and transfer an uncompressed Tarball

- Ensure that the source MacBook has **enough free disk space**. You need at least the same amount of free space that the total size of files that must be transfered.

- Open a terminal, navigate to the parent directory and create an uncompressed tar archive as shown below 

```
cd /path/to/parent_directory
tar -cvf archive_name.tar directory_name
```

- Copy the tar archive to the new MacBook. I recommend to do so using a high-speed connection such as Thunderbolt of by using a dedicated SSD drive.

- Extract the Tarball on the destination MacBook as shown below 

```
tar -xvf archive_name.tar
```

## Considerations

* Ensure both the source and destination MacBooks have sufficient disk space to accommodate the tar archive and its extracted contents.
* After extraction, verify the integrity of the transferred files to ensure no data corruption occurred during the process.
* Be mindful of file permissions and ownership, as they may need to be adjusted after extraction to maintain proper access controls.

By following this method, you can achieve a more efficient and streamlined transfer of a large number of files between 
MacBooks, minimizing the time associated with traditional file transfer methods.