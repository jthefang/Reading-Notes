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
- Not all devices have device files (e.g. network interface cards)

## The sysfs device path

## dd and devices

## Device name summary

## udev

## In-depth: SCSI and the Linux kernel