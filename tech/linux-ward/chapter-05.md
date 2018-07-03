# Chapter 5. How the Linux kernel boots

Overview of the boot process:
1. BIOS or boot firmware loads and runs boot loader
2. boot loader finds kernel image ondisk, loads it into memory and starts it
3. kernel initializes devices and drivers
4. kernel mounts the root filesystem
5. kernel starts *init* program (process ID of 1), which is the *user space start*
6. init sets system processes in motion
7. init starts a process that lets you log in (near the end of boot)

## Startup messages
- look at the kernel system log file */var/log/kern.log* or */var/log/messages*
- use `dmesg`, piped to `less` (screenful at a time)
- after kernel starts, the user-space startup procedure often generates messages printed to console that are erased after boot
	- usually each script writes its own log thought

## Kernel initialization and boot options
Kernel initializes in this general order:
1. CPU inspection
2. Memory inspection
3. Device bus discovery
4. Device discovery
5. Auxiliary kernel subsystem setup (networking, etc.)
6. Root filesystem mount
7. User space start

## Kernel parameters
- boot loader passes in a set of text-based kernel parameters to tell kernel how to start (e.g. what diagnostics to output, driver-specific options, etc.)
	- `cat /proc/cmdline` to view the kernel parameters
	- one word flags (e.g. `ro` or `quiet` or key=value pairs like `vt.handoff=7`)
- root parameter points to location of the root filesystem (where the kernel can find the init program and perform user space start)
	- can be specified by device `root=/dev/sda1` but more commonly by UUID `root=UUID={some id}`
	- usually mounted initially in read-only mode so `fsck` can check the root fs safely; its remounted in read-write mode for later use
- parameters the kernel doesn't understand are passed to init when performing the user space start (e.g. `-s` will tell init to start in single-user mode)

## Boot loaders
- loads the kernel into memory, then starts the kernel with the kernel parameters
- kernel and its parameters are both somewhere in the root filesystem (usually)
	- boot loader can't use the kernel to access the disk (obviously, since the kernel doesn't exist yet) so it uses the BIOS (basic I/O system) or UEFI (unified extensible firmware interface) to access disks (poorer performance than kernel drivers)
	- most boot loaders can read partition tables and other functions for read-only access to filesystems
- boot loader can also 1) select among multiple kernels 2) switch between sets of kernel parameters 3) allow user to manually override and edit kernel image names and parameters 4) provide support for booting other OS's

Main boot loaders:
- GRUB - near universal standard on Linux systems
- LILO - one of the first Linux boot loaders (ELILO for UEFI version)
- SYSLINUX - can be configured to run from many types of filesystems
- LOADLIN - boots kernel from MS-DOS
- efilinux - UEFI boot loader
- coreboot (formerly LinuxBIOS) - high-performance replacement for the PC BIOS that can include a kernel
- Linux Kernel EFISTUB - kernel plugin for loading the kernel directly from the EFI/UEFI System Partition (ESP)

## GRUB introduction
- __Grand Unified Bootloader__
- Getting the GRUB menu to even appear may be tricky (most Linux distributions try to hide the boot loader)
	- try to press and hold SHIFT when the BIOS/firmware startup screen first appears, press ESC to disable the automatic boot timeout after GRUB menu appears
- Press `e` to view the boot loader configuration commands for the default boot option
- kernel image loaded by GRUB something like `/boot/vmlinuz-{some kernel}`
- GRUB has its own root where it searches for the kernel and RAM filesystem image files (different than the root used for the kernel)
- `initrd` command used to specify the file for the initial RAM filesystem

GRUB has its own device names (e.g. `(hd0, msdos1)`)
- access GRUB command line by pressing `c` at boot menu/configuration editor
- `ls` to list disk devices and their partitions (`msdos` for MBR partition tables)
	- `ls -l` for more details, including the UUIDs of partitions on the disk
- `echo $root` to print the GRUB root where it expects to find the kernel 
	- `ls ($root)/` or `ls (hd0,msdos1)/` to list the files and directories in that partition (i.e. access the filesystem of the partition)
- use `set` to view all currently set GRUB variables
	- `$prefix` variable gives location of the filesystem and directory where GRUB expects to find its configuration and auxiliary support (see below)
- enter `boot` command to boot your current configuration
- press `esc` to return to the GRUB menu

GRUB configuration
- the GRUB (configuration) directory (*/boot/grub* or */boot/grub2*) contains the central configuration file *grub.cfg* and loadable modules with the *.mod* extension
- can use `grub-mkconfig` or `grub2-mkconfig` to modify the grub configuration file (or simply run those commands naked to print the configuration file to terminal)
	- `grub.cfg` consists of a bunch of GRUB commands: initialization steps then series of menu entries for different kernel and boot configurations 
	- submenus for older kernel versions
	- `grub.cfg` automatically generated and overwritten ocassionaly by the system
- every file in */etc/grub.d* is a shell script that produces a piece of the *grub.cfg* file
- to edit your GRUB configuration: make a custom `.cfg` file and use `grub-mkconfig` to generate the new configuration
	- add new menu entries/GRUB commands to a new file */boot/grub/custom.cfg*
		- can also edit the */etc/grub.d/40_custom* script, but a package upgrade is likely to destroy any changes you make
		- can also use the *41_custom* script to load *custom.cfg* when GRUB starts (changes don't appear when you generate your configuration file)
	- use `grub-mkconfig -o /boot/grub/grub.cfg` to write and install a newly generated GRUB configuration file to your GRUB directory
		- backup your old configuration, make sure you're installilng to the correct directory
- a whole section on how to install GRUB

## UEFI secure boot problems
- Secure boot feature of UEFI on recent PCs require boot loaders to be digitally signed in order to load and run them (Microsoft requires vendors shipping Windows 8 to use secure boot)
	- disable secure boot in the EFI settings
	- exist signed boot loaders for Linux distributions 

## Chainloading other OS's
- can install multiple boot loaders in the EFI partition (for UEFI), allows for loading other OS's
- MBR-style boot loaders don't support multiple boot loading
- chainloading can get GRUB to load and run a diffrent boot loader on a specified partition on your disk
	- create a new menu entry in your GRUB configuration
e.g. for a Windows installation on the 3rd partition of a disk:
```
menuentry "Windows" {
	insmod chain
	insmod ntfs
	set root=(hd0, 3)
	chainloader +1
}
```
	- the +1 option to `chainloader` tells it to load whatever is at the first sector of a partition (or `chainloader /io.sys` to load the *io.sys* MS-DOS loader)

## Boot loader details
2 main schemes for boot mechanism:
1. MBR Boot
	- Master Boot Record for partition information of disks, also includes a small area that the PC BIOS loads and executes after its Power-On Self-Test (POST) 
	- this initial space stores code to load the rest of the boot loader code (that's why it's a multi-stage boot loader) stuffed between the MBR and the first partition on disk 
	- this scheme doesn't work with a GPT-partitioned disk using the BIOS to boot because the GPT table info resides in the area after the MBR; GPT usually uses the UEFI boot scheme though
	- GPT can create a small partition, a BIOS boot partition, with a special UUID to give the full boot loader code a place to reside
2. UEFI Boot
	- Unified Extensible Firmware Interface (UEFI) is a standardized replacement to BIOS
	- has a built-in shell, ability to read partition tables on disks and navigate filesystems
		- GPT partitioning scheme is part of UEFI standard
	- EFI System Partition (ESP) is the special filesystem containing the executable boot code in the *efi/* directory
		- each boot loader has its own identifier and corresponding subdirectory, e.g. *efi/microsoft, efi/apple, efi/grub* with a *.efi* boot loader file in it
		- need the UEFI version of boot loaders (the BIOS versions won't work)
	- Secure boot incomptability with unsigned boot loaders
3. How GRUB works
	1. The PC BIOS or firmware intializes hardware and searches boot-order storage devices for boot code
	2. BIOS/firmware loads and executes boot code (e.g. GRUB)
	3. GRUB core loads and initializes; GRUB can now access disks and filesystems
		- the GRUB core is 1) partially stuffed between the MBR and the beginning of the first partition 2) in a regular partition OR 3) in a special boot partition, e.g. a GPT boot partition, an ESP, or elsewhere
		- PC BIOS loads 512 bytes from the MBR (and that contains start location of the core)
		- with an ESP (EFI System Partition), GRUB core goes there as a file; firmware can navigate the ESP and directory execute the GRUB core or any other OS loader there
	4. GRUB identifies its boot partition and loads a boot configuration there
	5. Gives user change to change configuration
	6. GRUB executes the configuration (sequence of commands, see 5.5.2 GRUB configuration)
	7. GRUB may load additional code/modules in the boot partition
	8. GRUB executes a `boot` command to load an execute the kernel (as specified by the configuration's `linux` command)
	- boot loader might also need to load an initial RAM filesystem image (via `initrd`) into memory before loading/executing the kernel