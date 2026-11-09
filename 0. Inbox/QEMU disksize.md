https://www.google.com/search?q=EXPAND+DISK+SIZE+ON+QEMU&rlz=1C1GCEU_enUS1079US1080&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIICAEQABgWGB4yCAgCEAAYFhgeMgoIAxAAGKIEGIkFMgoIBBAAGKIEGIkFMgcIBRAAGO8FMgcIBhAAGO8F0gEINjYzOGowajeoAgCwAgA&sourceid=chrome&ie=UTF-8&fbs=ACAhTvwdifjn2dKXmjtOfBxVFl_kNo0J7LRbgf5U9cgi9kOJRpGZXixKmdwaCd0VNh3itYyB_iE9lwVkLTpV2Rq87uAbHK7HK8kvM0iur6LCMvzCq4WE3IiowmQ2xOUlsDhR_5HC6E51-U8XzbLBOe-cAh9FsZVS_4-Irz2pj2o00lTpVj0Ci5RrNGQHmg4dMJL7J3x9RS4q8VlwYO5JqmcIFgvb3lAo3w&aep=10&ntc=1&sxsrf=AKL5yDgLn3rImHb_5fetQTt-41zf2_v7xg%3A1793740331376&mq=create+a+markdown+format+of+those+steps&mstk=AUtExfDa3klGgVCH-EhJMQUW38Jdtihwue8PdTyuf-t6NgnGBRCGrcPRdFAd-HtIlPRz0JN_bZ63P2sE6mpUDAW1S6T5gQOo-dk0-shF5UOCng_ypz67Euw5gKvpY5hPgRHiNFudMQqMBFqFsCQUr1o7Riijz7HNBgpmRWi8Rem-wKKqSOSNY2ywDvZIIBvhvU5_agplClbKvBAB8jIyJpnLJO0S7-RBSyKPc6hvkpnqeUHx-YXRsY2UGkIW9g06t9irpINMtC2En8pRtA&aioh=3&csuir=1&ved=2ahUKEwi11Ka62-yXAxXySjABHUGjHoIQk8ITegQIAxAA&cs=0&mtid=juDxatyKNZriwN4PprXO6QY&udm=50


Expanding QEMU Disk Size

Expanding a QEMU disk size is a two-step process: first, you must resize the physical disk block file on the host machine, and second, you must expand the partition and filesystem inside the guest OS so it can utilize the new space.

Step 1: Expand the Disk Image on the Host

Before modifying the disk image, shut down your virtual machine completely. Do not attempt this on a running VM or an image with active snapshots.
1. Locate your virtual disk image. Standard paths are often under /var/lib/libvirt/images/. If you aren't sure of the exact path, use virsh to find it:bash
sudo virsh domblklist <vm-name>
Use code with caution.
2. Resize the file. Use the qemu-img command to add your desired amount of storage.
	• To add a specific amount (e.g., 20 Gigabytes):bash
sudo qemu-img resize /path/to/image.qcow2 +20G
Use code with caution.
	• To change it to an absolute size (e.g., make it 60 Gigabytes total):bash
sudo qemu-img resize /path/to/image.qcow2 60G
Use code with caution.

Step 2: Expand the Filesystem Inside the Guest

Now, power on your virtual machine. The operating system will see the larger hardware drive, but the partition containing your files remains the original size. Follow the instructions below depending on your guest OS.

Option A: Linux Guest (Standard Partition)

The easiest way to safely update the partition table from the terminal is by using growpart.
1. Install the guest utility if it is not already present:
	• Debian / Ubuntu:bash
sudo apt update && sudo apt install cloud-guest-utils
Use code with caution.
	• RHEL / CentOS / Rocky Linux:bash
sudo dnf install cloud-utils-growpart
Use code with caution.
2. Identify your target partition name using lsblk. Let's assume your main partition is partition 3 on device /dev/vda (indicated as /dev/vda3).
3. Grow the partition:bash
sudo growpart /dev/vda 3
Use code with caution.
4. Resize the filesystem to fill that partition:
	• For ext4 filesystems:bash
sudo resize2fs /dev/vda3
Use code with caution.
	• For XFS filesystems:bash
sudo xfs_growfs /
Use code with caution.
(Note: xfs_growfs takes the mount point—like /—rather than the device path).

Option B: Linux Guest (LVM Partition)

If your Linux guest relies on Logical Volume Management (LVM), adjust the layout using LVM-specific tools:
1. Grow the partition block:bash
sudo growpart /dev/vda 3
Use code with caution.
2. Tell LVM the physical volume grew:bash
sudo pvresize /dev/vda3
Use code with caution.
3. Extend the logical volume to use 100% of the free space:bash
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
Use code with caution.
4. Resize the filesystem:bash
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
Use code with caution.

Option C: Windows Guest

If you are running a Windows virtual machine, you can expand the space entirely using the graphical interface:
1. Log into your Windows guest.
2. Right-click the Start Menu and select Disk Management.
3. Locate your primary volume (usually C:) right next to a block of unallocated space.
4. Right-click your C: partition and select Extend Volume.
5. Click Next through the wizard to consume the unallocated space.
If you hit any errors during these steps, let me know:
• What specific error message did you see?
• What command triggered it?
• What Linux distribution or OS version are you using?
