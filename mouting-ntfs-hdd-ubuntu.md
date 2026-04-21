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
Block ID queries a specific partitiion and returns detailed information about it. It would return the exact UUID, filesystem, type, and label. This method is more reliable than `lslbk -f` for getting the precise UUID. 

**UUID:** `5C82099B8207B32`

## Step 5 - Back up fstab before editing 
After obtaining the UUID, the next part of my plan involved backing up the fstab by running the following: 
```bash
sudo cp /etc/fstab /etc/fstab.backup
```
"Copy" makes a backup copy of the fstab before editing it. A corrupted fstab can prevent Ubuntu from booting properly. I learnt that backing up the fstab is standard practice before editing it. 
