# Root Priority List (root_pr) - Enablement Package

This directory contains everything needed to enable the **root_pr** kernel feature on Linux 6.1.x. The root_pr feature allows specifying multiple root filesystems in priority order—if the first fails, the kernel automatically tries the next.

## Contents

| File | Description |
|------|-------------|
| `root_pr.patch` | Unified diff patch for `init/do_mounts.c` (~210 lines) |
| `README.md` | Usage and application instructions |

## Quick Start

### 1. Apply the Patch

From the kernel source root directory:

```bash
# Apply the patch (for vanilla kernel 6.1.x)
patch -p1 < root_pr_enablement/root_pr.patch
```

If you get "already applied" or the kernel already has root_pr, the patch may fail—that's expected.

### 2. Build the Kernel

```bash
make -j$(nproc)
```

### 3. Boot with root_pr

Add to your kernel command line (GRUB, U-Boot, etc.):

```bash
root_pr="/dev/mmcblk1p2;/dev/mmcblk1p1;/dev/sda1"
```

For block devices only (`/dev/sda1`, `PARTUUID=...`), no extra config is needed beyond a standard kernel.

## Usage Examples

```bash
# Multiple block devices
root_pr="/dev/mmcblk1p2;/dev/mmcblk1p1;/dev/sda1"

# With NFS fallback (requires CONFIG_ROOT_NFS)
root_pr="/dev/mmcblk1p2;/dev/nfs;/dev/mmcblk1p1"

# With PARTUUID
root_pr="PARTUUID=12345678-1234-1234-1234-123456789abc;PARTUUID=87654321-4321-4321-4321-cba987654321"

# Full boot line
linux /vmlinux root_pr="/dev/mmcblk1p2;/dev/nfs;/dev/mmcblk1p1" rootfstype=ext4 ro
```

## Verification

Check kernel logs for root_pr activity:

```bash
dmesg | grep "VFS:"
```

Expected output when using root_pr:

```
VFS: Trying root filesystems from priority list: /dev/mmcblk1p2;/dev/mmcblk1p1
VFS: Attempting to mount root filesystem #1: /dev/mmcblk1p2
VFS: Successfully mounted root filesystem #1: /dev/mmcblk1p2
```

## Kernel Version

- **Target**: Linux 6.1.x (tested on 6.1.131)
- **Architecture**: All architectures supported by the kernel
