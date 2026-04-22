# Mounting a 1TB NFTS HDD on Ubuntu

## Overview
After installing Ubuntu 24.04.4 on my laptop, the 1TB HDD was visible to the system but not accessible. This write-up documents how I identified the drive, mounted it, and configured it to auto-mount on every boot by editing: `/etc/fstab`

## System Info
- **OS:** Ubuntu 24.04 LTS
- **Drive being mounted:** 1TB (NTFS, previously used with Windows 10)
- **Mount Point:** `/mnt/storage`

## Step 1 - Identify the Drive 
Opened Terminal with `Ctrl` + `Alt` + `T` and ran:
```bash
lsblk
```
This listed all connected devices. I could see my 1TB HDD showing as `sda` with a single partition `sda1`. It had no mount point, which confimred that it wasn't yet accessible. 

## Step 2 — Create a Mount Point
The next course of action was to run: 
```bash
sudo mkdir /mnt/storage
```
"Make Directory" creates a new empty folder empty folder at `/mnt/storage` to act as the mount point, which is the location in the Linux file system where the contents of my HDD will appear.

## Step 3 - Test Mount the Drive
The next step was to check if the drive worked by running:
```bash
sudo mount /dev/sda1 /mnt/storage
```
This temporarily mounted the drive to confirm it worked. Running `ls /mnt/storage` showed the drive contents (there were leftover Windows system files), confirming the mount was succesful.

## Step 4 - Get the Drive's UUID
My next move was to get the drive's UUID. I did this by entering the following: 
```bash
sudo blkid /dev/sda1
```
Block ID queries a specific partitiion and returns detailed information about it. It would return the exact UUID, filesystem, type, and label. Prior research into the process informed me that this method is more reliable than `lslbk -f` for getting the precise UUID. 

**UUID:** `5C82099B8207B32`

## Step 5 - Back up fstab before editing 
After obtaining the UUID, the next part of my plan involved backing up the fstab by running the following: 
```bash
sudo cp /etc/fstab /etc/fstab.backup
```
I learnt beforehand that backing up the filesystem table is standard practice before editing it. A corrupted file system table can prevent Ubuntu from booting properly. The "Copy" command makes a backup copy of the filesystem before editing it. 

## Step 6 - Edit the fstab
```bash
sudo nano /etc/fstab
```
Added the following line at the bottom of the file 
`UUID=5C82099B8207B32  /mnt/storage  ntfs-3g  defaults,nofail  0  0`

`defaults,nofail` was added so that the device mounts normally and the booting process does not halt if the drive isn't found when I reboot the system. 

The first `0` indicates not to include it in the backup utility and the second `0` is tells the system does not check filesystem integrity on boot. 

The line was saved with `Ctrl+O`, `enter`, and then `Ctrl+X`. 

## Step 7 - Reboot and Test
My next move wasto test that the test whether it will auto-mount on reboot. This was done by doing:
```bash
sudo reboot
lsblk
```
Unfortunately the results of `lsblk` showed that the device was no longer mounted. 

insert results here

My first thought was that it could either be: 
1) a typo in the UUID
2) extra spaces or wrong spaces between the fields
3) or a missing or extra character somewhere

To check the fstab entry if there was a typo, I used: `cat /etc/fstab`. The results are shown below:
```bash
UUID=5C82099B8207B32 /mnt/storage ntfs-3g defaults,nofail 0 0
```

