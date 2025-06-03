---
title: Resize VMWare vSphere Partition
date: 2025-05-21
draft: false
tags: 
description:
---
Created: 2025-05-21

At my internship, we needed to increase the storage space of an Ubuntu server. However, this is not as straightforward as resizing a regular ubuntu partition since the 1. server is usually always on and 2. the server runs on the main partition. Therefore, to avoid data loss, here are the steps I took to increase the servers storage space without losing data. 
1. Shut down the Ubuntu server and add more storage, e.g via VMWare vSphere dashboard.
2. Start the server.
3. SSH into the server.
4. Resize the main partion
	1. Open CFDISK to modify the partitions: `sudo cfdisk`
	2. Choose the partition to extend -> select "Resize"
	3. Set the "New Size" then press ENTER. Tip: this should automatically populate the newly added storage size.
	4. "Write" the changes to save
	5. "Quit" cfdisk
5. Reboot the server: `sudo reboot`
6. Identify the partition to be resized: `lsblk`
7. Get the name of the physical volume: `sudo pvs`
8. Resize the physical volume and replace "PV Name" with the name identified in the previous step, e.g "/dev/sda3": `sudo pvresize "PV Name"`
9. Get the name of the root logical volume, e.g "/dev/mapper/ubuntu--vg-root" by running: `df -h`
10. Expand the logical volume, replacing "root_logical_volume_name" with the path identified in the previous step: `sudo lvextend -r -l +100%FREE root_logical_volume_name
11. Reboot: `sudo reboot`
### Credits
Thanks to the authors of the following comments for tips on accomplishing this:
* [Method 2](https://askubuntu.com/a/940404)
* https://unix.stackexchange.com/a/583544
