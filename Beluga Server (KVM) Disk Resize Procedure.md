Beluga Server (KVM) Disk Resize Procedure
Root Partition Expansion Using Swapfile
Purpose

This procedure describes how to increase the root filesystem size of a KVM virtual machine when the root partition is blocked by an extended partition and swap partition. The process removes the existing swap partition, expands the root partition, and recreates swap using a swapfile.

Prerequisites

Before proceeding, ensure the following:

The virtual disk has already been increased at the manifest level.
The VM is running a Debian-based operating system.
The root filesystem is formatted as ext4.
A backup or snapshot of the VM is available.
Step 1: Extend the Virtual Disk on the Host

Login to the KVM host and extend the logical volume backing the virtual machine disk.

Example:

lvextend -L +5G /dev/instances/testvm-virt.belugacdn.com-disk1

After extending the disk, verify that the VM can see the updated disk size.

Step 2: Install Required Utilities

Login to the virtual machine and install the parted utility.

apt update
apt install parted -y
Step 3: Disable Swap

Before modifying disk partitions, disable swap.

swapoff -a

Verify that swap is disabled:

swapon --show

Expected result:

No output
Step 4: Expand the Root Partition
Current Partition Layout

Review the current partition layout:

lsblk

Example:

NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda     254:0    0 25G  0 disk
├─vda1  254:1    0 19G  0 part /
├─vda2  254:2    0 1K   0 part
└─vda5  254:5    0 975M 0 part [SWAP]
Modify Partitions

Start the partition editor:

parted /dev/vda

Execute the following commands:

print
rm 5
rm 2
resizepart 1 100%
quit
Explanation
vda5 is the existing swap partition.
vda2 is the extended partition container.

These partitions occupy space immediately after the root partition and prevent the root filesystem from expanding into the newly added disk space.

Verify Partition Layout
lsblk

Expected result:

NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda     254:0    0 25G  0 disk
└─vda1  254:1    0 25G  0 part /
Step 5: Resize the Filesystem

Expand the ext4 filesystem to utilize the additional partition space.

resize2fs /dev/vda1

Verify the filesystem size:

df -h /

Example:

Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        25G  4.1G   19G  15% /
Step 6: Create a Swapfile

Instead of recreating a swap partition, create a swapfile.

Create the Swapfile
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
Configure Swapfile Persistence

Add the following entry to /etc/fstab:

echo '/swapfile none swap sw 0 0' >> /etc/fstab
Step 7: Verification

Verify that the root filesystem has been expanded and the swapfile is active.

Verify Disk Layout
lsblk
Verify Filesystem Size
df -h /
Verify Swap
swapon --show
Expected Outcome

After completing this procedure:

The root partition occupies all available disk space.
The ext4 filesystem has been expanded successfully.
The original swap partition has been removed.
A swapfile is configured and enabled.
Future disk expansions are simplified because no swap partition exists to block partition growth.
Final Validation Checklist
Validation Item	Command
Root partition expanded	lsblk
Filesystem expanded	df -h /
Swapfile active	swapon --show
Swapfile persistent after reboot	grep swapfile /etc/fstab
Procedure Complete
