# **Troubleshooting**
This file documents the issues I encountered while setting up and running my Splunk home lab.These are the problems I faced, along with their causes, fixes, and outcomes.

## **Issue 1: Search not executed (disk space error)**
### **Error Message**:
![issue1](/screenshots/setup/issue1.png)
That error means your Ubuntu Server VM is low on disk space

### **Cause**:
Splunk blocks searches by default when free space drops below 5000MB (5GB), to protect itself from crashing due to full disk.

**There two common issues:**
1. Ubuntu Server VM is low on disk space
2. Only limited space allocated root filesystem (/dev/mapper/ubuntu--vg-ubuntu--lv).

```bash
df -h
```
Look at the line for / (root) or wherever ubuntu lives — check the "Avail" column.
Size should 30+GB and avail space should be over 5GB.

### **Fix**:
We can fix both issues at once.

**Step 1: Shut down the Ubuntu VM**
```bash
sudo shutdown now
```
**Step 2: Expand the virtual disk in VMware** (skip this if your disk size is 40GB or more)
- Select the Splunk-Server VM → VM → Settings → Hard Disk
- Increase size (e.g., 19GB → 40GB) — gives real headroom for log growth.
- Click OK, then power the VM back on

**Step 3: Extend the partition and LVM volume (inside Ubuntu)**
```bash
# Confirm the new raw disk space is visible or 
sudo lsblk

# Grow the physical partition (Do this if you done step 2 otherwise skip)
sudo growpart /dev/sda 3

# Resize the physical volume (Do this if you done step 2 otherwise skip)
sudo pvresize /dev/sda3

# Extend the logical volume to use all the free space (compulsory for both issue)
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

# Resize the filesystem to match(compulsory  for both issue)
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```
**Step 4: Verify**
```bash
df -h
```
You should now see much more available space on /dev.