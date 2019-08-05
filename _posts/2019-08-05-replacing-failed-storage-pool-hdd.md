---
layout: post
title: Replace Failed Storage Pool HDD
category: powershell
description: Replacing a failed hard disk drive in Windows Server Storage Spaces Storage Pool.
tags: powershell windows server
---

In my day to day jobby, I manage multiple Hyper-V clusters, and these clusters use shared storage.  Most of the storage is a JBOD shelf with dual SAS connections to the hypervisor nodes.  Because the shelf is JBOD, Windows is the _brains_ for the storage.  Enter [Windows Storage Spaces](https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/overview).  This basically a Software RAID that you configure a _pool_ of disks, and then you layer a VHD over that pool.  After a volume is created on the VHD, it can be used by the hosts.  These volumes can be added to [Cluster Shared Volumes](https://docs.microsoft.com/en-us/windows-server/failover-clustering/failover-cluster-csvs) which allows the volume to move between host nodes, creating redundancy.  All the VMs hosted in the cluster, live on the CSV, making them Highly Available.

As hardware always does, physical disks in the pool fail.  Most of the Microsoft docs on fixing this issue are quite cumbersome to follow, and even more so, they tend to deal with [Storage Spaces Direct](https://docs.microsoft.com/en-us/windows-server/storage/storage-spaces/storage-spaces-direct-overview), which is not the same technology.  So I thought I would make up a quick little walk through on replacing a failed HDD in a storage pool.  I have to do this about once a month, so this is just as much for me, as it is for you.

In my configuration, I run Windows Server Core exclusively for Hypervisors, so we will tackling this with PowerShell, tbh, not really even sure how to do this with a GUI.

## Identifying a failed HDD
Identifying a failed HDD is pretty simple, it can be done with a simple PowerShell command:

```powershell
Get-PhysicalDisk | ? HealthStatus -ne "Healthy"
```

This will show you any disks that have failed, I typically use a 3-way mirror on my Storage Pools, so if that number is greater then 2, then well, might as well just start over.  Anyway, if the `Usage` property of the failed disk is not marked as _Retired_, you will need to mark it as such:

```powershell
Get-PhysicalDisk | ? HealthStatus -ne "Healthy" | Set-PhysicalDisk -Usage Retired
```

So, this next part tends to vary between JBOD models, but to actually _identify_ the failed HDD for physical replacement, there are a few things you can do, first off, you can actually trigger the slots LED to indicate the failed drive:

```powershell
# This command assumes you already completed the previous command and 'Retired' the failed disks.
Get-PhysicalDisk -Usage Retired | Enable-PhysicalDiskIdentification
```

You can also get the physical `SlotNumber`:

```powershell
Get-PhysicalDisk -Usage Retired | select SlotNumber
```

However, this number is 0 indexed so when looking at the JBOD shelf, be sure to start counting at 0.

## Replacing the HDD
After you have identified the failed HDD and marked it as retired, you can now physically replace the drive.  In my experience, after the new drive is inserted and running `Get-PhysicalDisk` the new drive always hangs in a `Starting` Operational Status.  And because of this you cannot add it to the storage pool (notice the `CanPool` Property will be `False`).  This can be cleared up by Resetting the disk:

```powershell
Get-PhysicalDisk | ? OperationalStatus -eq "Starting" | Reset-PhysicalDisk
```

After this is complete, running the `Get-PhysicalDisk` will show the disk is `Healthy` and the `CanPool` property will be `True`.  Now, you need to identify the Storage Pool's Friendly Name, this can be found by running the `Get-StoragePool` command.  Once you have the Friendly Name, you can add the new disk to the pool:

```powershell
Add-PhysicalDisk -StoragePoolFriendlyName MyStoragePool -PhysicalDisks ( Get-PhysicalDisk -CanPool:$true )
```

Now that the drive is replaced, go ahead and repair the virtual disk:

```powershell
Get-VirtualDisk | ? HealthStatus -ne "Healthy" | Repair-VirtualDisk
```
This will take some time depending on how large the virtual disk is, resiliency settings, and hardware, but there will be a progress bar.  Or you can add the `-AsJob` switch to the `Repair-VirtualDisk` command and let it run in the background.  You can fetch the status with `Get-StorageJob`.

Be sure to disable the LED identification:

```powershell
# I always just run this on against all the disks, it won't hurt anything for the ones where its not enabled.
Get-PhysicalDisk | Disable-PhysicalDiskIdentification
```

After the repair is complete, the last step is to remove the failed drive from the pool.  Remember, the repair has to fully complete before removing the failed drive from the pool, else data loss will occur.

```powershell
Get-PhysicalDisk -Usage Retired | Remove-PhysicalDisk
```

And thats it, good to go!  Cheers.