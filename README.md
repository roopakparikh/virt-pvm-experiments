# virt-pvm-experiments
Virtualization with PVM, host and guest configuration and experiments

These are some basic experiments to incorporate changes proposed by excellent work
by the authors of the paper

__PVM: Efficient Shadow Paging for Deploying Secure Containers in Cloud-native Environment__
https://github.com/virt-pvm/misc/blob/main/sosp2023-pvm-paper.pdf


__This repository will contain examples and build configuration for both the guest and the hypervisor__



This repository contains some examples on how to run an ubuntu cloud image using PVM virtualization

I ran these experiment on an AWS machine but it should be possible to run it in other places as well

I have used the following repository as the base: https://github.com/virt-pvm/misc/blob/main/pvm-get-started-with-kata.md#install-pvm-hypervisor

and it largely works, unfortunately I wasn't able to find a way to contribut to the repo and had to create additional instructions here

# Host Instance

My environment

1 VM using `m6i.4xlarge` flavor 


# Compiling Kernel for the Host

There is a change in the Linux Kernel to add support for PVM. This largely works, but there some additional kernel module that you need to enable in the configuration for most applications. The kernel patches are maintained in the pvm branch of the linux fork maintained on the `virt-pvm` 

Base instructions: 
https://github.com/virt-pvm/misc/blob/main/pvm-get-started-with-kata.md#install-pvm-hypervisor

The instructions above are accurate and works very well, I tried it on rocky linux for my epxeriments.

# Guest Kernel, Qemu and Initramfs

Compiling guest kernel can be challenging depending on what you are trying to run under your OS.

These are my experiments, you mileage would vary based on usage:
# Guest Kernel
Additional configuration to enable:

### SquashFS
Enable SquashFS - specifically if you want to run Ubuntu which uses Snap that uses SquashFS. I haven't enabled all, but you can further optimize it to your experiments.

```
CONFIG_SQUASHFS=y
CONFIG_SQUASHFS_FILE_CACHE=y
# CONFIG_SQUASHFS_FILE_DIRECT is not set
CONFIG_SQUASHFS_DECOMP_SINGLE=y
CONFIG_SQUASHFS_DECOMP_MULTI=y
CONFIG_SQUASHFS_DECOMP_MULTI_PERCPU=y
CONFIG_SQUASHFS_CHOICE_DECOMP_BY_MOUNT=y
CONFIG_SQUASHFS_MOUNT_DECOMP_THREADS=y
CONFIG_SQUASHFS_XATTR=y
CONFIG_SQUASHFS_ZLIB=y
# CONFIG_SQUASHFS_LZ4 is not set
CONFIG_SQUASHFS_LZO=y
CONFIG_SQUASHFS_XZ=y
CONFIG_SQUASHFS_ZSTD=y
CONFIG_SQUASHFS_4K_DEVBLK_SIZE=y
CONFIG_SQUASHFS_EMBEDDED=y
CONFIG_SQUASHFS_FRAGMENT_CACHE_SIZE=3
```

### CODEPAGE
Not sure if it is necessary, but I ended up adding following to help with my Ubuntu boot

```
CONFIG_NLS=y
CONFIG_NLS_DEFAULT="iso8859-1"
CONFIG_NLS_CODEPAGE_437=y
CONFIG_NLS_ISO8859_1=y
```

### PVM enable
Verify
```
CONFIG_PVM_GUEST=y
```

## Compile Qemu
Use https://github.com/virt-pvm/qemu and branch  `pvm_qemu_migration`

# Script to run ubuntu

## Download Ubuntu cloud image

```
 wget https://cloud-images.ubuntu.com/jammy/20250108/jammy-server-cloudimg-amd64-disk-kvm.img
 mv jammy-server-cloudimg-amd64-disk-kvm.img ubuntu-vanilla.img
 ```

 ## Use Qemu to start the VM 
 ```
QEMU_BIN=~/qemu/build/qemu-bundle/usr/local/bin/qemu-system-x86_64

$QEMU_BIN \
   -enable-kvm \
   -m 6G \
   -bios qboot.bin \
   -kernel ~/linux/vmlinux \
   -initrd ~/ubuntu-vanilla-initrd.gz \
   -nographic \
   -append "root=/dev/vda1 rw pti=off nokaslr lapic=notscdeadline console=ttyS0" \
   -drive file=ubuntu-vanilla.img,format=qcow2,if=virtio
 ```