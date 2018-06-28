# Chapter 3. Devices

Basic tour of the kernel-provided device infrastructure: how it presents devices to the user, device configuration info via sysfs, udev allows user-space programs to automatically configure and use new devices.

## Device files
- Many device I/O interfaces are presented as files, i.e. you interact with them as if they were files. These device files are called *device nodes*
	- `echo blah blah > /dev/null`
	- redirects output to the file; kernel decides what to do with any data written to this device (in this case nothing)
- Device files are stored in */dev*
	- `ls -l` to identify a device and view its permissions
	- `b, c, p, s` as the file type (first character in file mode column) indicates a device (block, character, pipe, socket)
	- in the detailed listing, the 2 numbers (sometimes only 1 number) before the date column are the *major* and *minor* device numbers that help kernel id the device
		- similar devices usually have the same major number
- Block device
	- data is accessed by programs in fixed chunks
	- total size fixed, easy to index => can have random access to any block with help of kernel
- Character device
	- work with data streams (can only read/write characters like with */dev/null*)
	- no buffer size
	- during character device interaction, the kernel cannot back up and reexamine the data stream after it has passed data to the device or process
	- e.g. printers
- Pipe device
	- *named pipes* are like character devices but with another process at the other end of the I/O stream instead of a kernel driver
- Socket device
	- sockets are special-purpose interfaces, e.g. for interprocess communications
	- often found outside */dev* directory
	- socket files represent Unix domain sockets (see Chapter 10)
- Not all devices have device files (e.g. network interface cards
)

## The sysfs device path
- The sysfs interface provides a uniform view for attached devices based on their hardware attributes
	- the */dev* directory is there fore user processes to use a device
	- the */sys/devices* directory is there to view information about and manage devices
- Used primarily by programs, not humans
- */sys/block* contains all the block devices available on the system
	- these are just symbolic links to paths in the */sys/devices* directories (use `ls -l` to see)
- To get the sysfs path of a device in */dev* use: `udevadm info --query=all --name=/dev/sda`
	- sysfs path under `DEVPATH`
	- it also provides other info like the major and minor numbers of the device
- See 3.5 for more on udev

## dd and devices
- `dd` program reads from an input file/stream and writes to an output file/stream
	- useful when working with block and character devices that you read/write from like a file
- `dd if=/dev/zero of=new_file bs=1024 count=1`
	- uses old IBM Job Control Language (JCL) style instead of the standard Unix dash format for signalling options
	- `if` points to the input file, `of` the output file
	- `bs` is the block size that `dd` reads/writes, can be abbreviated using `b` or `k` to signify 512 and 1024 bytes (e.g. `bs=1k` instead of `bs=1024`)
	- `ibs=size, obs=size` specify different block sizes for input and output
	- `count=num` specifies total number of blocks to copy (if a huge file, you need to tell `dd` to stop at a fixed point or you'll waste a lot of disk space, CPU time, etc.)
	- `skip=num` specifies how many blocks to skip in the input file before starting to copy its data to the output
- Be careful using `dd`: easy to corrupt files and data on devices with careless mistakes 

## Device name summary
- The kernel may not have a device file for your device
- To find the name of a device:
	- query `udevd` using `udevadm` (see next section)
	- Look for the device in the */sys* directory
	- guess name from output of `dmesg` command (prints last few kernel messages) or the kernel system log file (see Chapter 7 on system logging); output might contain a description of the system devices
	- check output of the `mount` command for devices already visible to the system
	- `cat /proc/devices` to see block and character devices for which your system has drivers configured; displays the major number and name of the device
- Hard disks attached to the system have device names with *sd* prefix (e.g. */dev/sda, /dev/sdb*, etc)
	- */dev/sda1, /dev/sda2* are partitions of the disk
	- sd = SCSI disk (Small Computer System Interface, a protocol for communication between storage devices and other peripherals)
	- `lsscsi` to list the SCSI devices of the system: gives address of device on system, type of device, and path of device file (rest is vendor info)
	- use UUID (Universally Unique Identifier) for consistent disk device access
	- optical storage drives (CD, DVD) are recognized as SCSI devices */dev/sr0, /dev/sr1*, etc; they're read-only
		- for write capabilities use the "generic" SCSI devices */dev/sg0*
- If a drive uses an older interface, it may show up as a PATA device: block devices */dev/hda, /dev/hdb, /dev/hdc, /dev/hdd* 
	- if a SATA drive is recognized as one of these disks, it's running in compatability mode which hinders performance (check BIOS settings to see if you can switch the SATA controller to its native mode)
- Terminals are devices for moving characters between a user process and an I/O device (*/dev/tty\*, /dev/pts/\*, /dev/tty*)
	- pseudoterminal devices are emulated devices that presents an I/O interface to a piece of software (e.g. a shell window)
	- */dev/tty1* is the first virtual console, */dev/pts/0* is the first pseudoterminal device
	- */dev/tty* is the controlling terminal of the current process (i.e. if you start a program from terminal, it is reading and writing to the terminal a.k.a. */dev/tty*)
- Display modes & virtual consoles
	- Linux starts in text mode --> usually switches to graphics mode (an X Window System server)
	- Virtual consoles multiplex the display, each one may one in graphics or text mode (think of it like virtual desktops)
		- use CTRL + ALT + F# (if in graphics mode) to switch to another virtual console
		- or ALT + F# (if in text mode); e.g. ALT + F1 switches you to */dev/tty1*
	- can also switch consoles using `chvt 1`, which switches to *tty1*
- COM1 and COM2 are serial ports on Windows systems, corresponding to the special terminal devices */dev/ttyS0, /dev/ttyS1*
	- USB serial adapters show up with USB and ACM in the name: */dev/ttyUSB0, /dev/ttyACM0*
- 2 sets of audio devices: one for the Advanced Linux Sound Archtiecture (ALSA) system interface and another for the older Open Sount System (OSS)
	- ALSA devices in */dev/snd/*
	- OSS *dsp* and *audio* devices: any WAV file you send to */dev/dsp* will be played but there may be some frequency mismatches and the device is usually busy
	- sound is a messy Linux topic
- On modern systems, you don't create your own device files (instead devtmpfs and udev do it for you)
	- you can manually create a device with `mkdnod /dev/sda1 b 8 2` (creates */dev/sda1* as a block device with major 8 and minor 2)
	- need to know the major and minor; for character or named pipe devices use `c` or `p` instead of `b` (and omit major/minor for named pipes)

## udev
- The kernel taps a user-space process `udevd` to create device files and initialization (e.g. when it detects a new USB flash drive)
	- devtmpfs filesystem early device configuration during booting
	- kernel sends `udevd` a *uevent* notification
	- `udevd` loads all attributes in the uevent
	- `udevd` parses its rules (in */lib/udev/rules.d/* for defaults, */etc/udev/rules.d/* for overrides), takes actions or sets more attributes based on those rules
	- creates symbolic links for devices
- `udevadm` is adminstration tool/program for `udevd`
	- `udevadm info --query=all --name=/dev/sda` to see all the udev attributes used and generated with the rules for the */dev/sda* device
	- `udevadm monitor` to monitor uevents 
		- shows incoming kernel messages and outgoing udev messages (sent to other programs)
		- `--kernel` or `--udev` to see only incoming/outgoing messages 
		- `--property` to see the whole incoming uevent including its attributes
		- `--subsystem-match=scsi` to see only messages about the SCSI subsystem

## In-depth: SCSI and the Linux kernel
- kernel talks via a SCSI host adapter to devices connected via a SCSI bus
- the stack:
	- device driver on top layer: handles operations for a class of device, i.e. translates requests from kernel block device interface to device specific commands in the SCSI protocol and vice versa
	- bus management core in middle layers: moderates and routes SCSI messages between top and bottom layers, tracking all the SCSI buses and devices attached to the system
	- host controller drivers at bottom layer: handles hardware-specific actions, sending SCSI protocol messages to specific host adapters/hardware, extracting incoming messages from the hardware
- `lsusb` command for listing usb devices, similar to `lsscsi`
	- USB interface sandwiched in the SCSI stack (i.e. can use USB using SCSI)
- user space programs usually communicate with the SCSI subsystem through the kernel (i.e. the block device layer)
	- but you can bypass device class drivers and give SCSI protocol commands directly to devices through their generic drivers (do `lsscsi -g` to show)