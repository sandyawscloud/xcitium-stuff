# Beluga Server (KVM) Disk Resize Procedure

## Root Partition Expansion Using Swapfile

Increase the root filesystem size of a KVM virtual machine where the root partition is blocked by an extended partition and swap partition.

---

## Table of Contents

* [Prerequisites](#prerequisites)
* [Step 1 - Extend the Virtual Disk](#step-1---extend-the-virtual-disk)
* [Step 2 - Install Required Package](#step-2---install-required-package)
* [Step 3 - Disable Swap](#step-3---disable-swap)
* [Step 4 - Expand the Root Partition](#step-4---expand-the-root-partition)
* [Step 5 - Resize the Filesystem](#step-5---resize-the-filesystem)
* [Step 6 - Create a Swapfile](#step-6---create-a-swapfile)
* [Step 7 - Verify the Configuration](#step-7---verify-the-configuration)

---

## Prerequisites

> [!IMPORTANT]
> Before proceeding:
>
> * Disk size has already been increased at the manifest level.
> * VM is running a Debian-based Linux OS.
> * Root filesystem type is `ext4`.
> * A backup or VM snapshot is strongly recommended.

---

# Step 1 - Extend the Virtual Disk

Login to the physical host and extend the VM disk by **5 GB**.

```bash
lvextend -L +5G /dev/instances/testvm-virt.belugacdn.com-disk1
```

> [!TIP]
> Verify that the VM can see the updated virtual disk size before proceeding.

---

# Step 2 - Install Required Package

Login to the VM and install `parted`.

```bash
apt update
apt install parted -y
```

---

# Step 3 - Disable Swap

Disable swap before modifying partitions.

```bash
swapoff -a
```

Verify swap is disabled:

```bash
swapon --show
```

Expected output:

```text
(no output)
```

> [!IMPORTANT]
> Swap must be disabled before deleting the swap partition.

---

# Step 4 - Expand the Root Partition

## Current Partition Layout

```bash
lsblk
```

Example:

```text
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda     254:0    0 25G  0 disk
├─vda1  254:1    0 19G  0 part /
├─vda2  254:2    0 1K   0 part
└─vda5  254:5    0 975M 0 part [SWAP]
```

## Modify Partitions

Start `parted`:

```bash
parted /dev/vda
```

Execute:

```text
print
rm 5
rm 2
resizepart 1 100%
quit
```

> [!WARNING]
> `vda5` (swap) and `vda2` (extended partition) must be removed because they prevent `vda1` from expanding into the newly added disk space.

Verify the new layout:

```bash
lsblk
```

Expected:

```text
NAME    MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda     254:0    0 25G  0 disk
└─vda1  254:1    0 25G  0 part /
```

---

# Step 5 - Resize the Filesystem

Expand the ext4 filesystem to use the newly available space.

```bash
resize2fs /dev/vda1
```

Verify:

```bash
df -h /
```

Example:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        25G  4.1G   19G  15% /
```

> [!SUCCESS]
> The root filesystem should now reflect the expanded size.

---

# Step 6 - Create a Swapfile

Create a new 1 GB swapfile.

```bash
fallocate -l 1G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Configure swap persistence:

```bash
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

> [!NOTE]
> Swapfiles are easier to manage and simplify future disk expansion operations.

---

# Step 7 - Verify the Configuration

Verify disk layout:

```bash
lsblk
```

Verify filesystem size:

```bash
df -h /
```

Verify swap:

```bash
swapon --show
```

---

# Validation Checklist

| Validation              | Command                    |
| ----------------------- | -------------------------- |
| Root partition expanded | `lsblk`                    |
| Filesystem expanded     | `df -h /`                  |
| Swapfile active         | `swapon --show`            |
| Swapfile persistent     | `grep swapfile /etc/fstab` |

---

# Expected Outcome

After completing this procedure:

* ✅ Root partition uses all available disk space.
* ✅ Filesystem has been expanded.
* ✅ Swap partition has been removed.
* ✅ Swapfile is active.
* ✅ Future disk expansions are simpler because no swap partition blocks partition growth.

---

## Procedure Complete
