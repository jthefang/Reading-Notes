# Chapter 1. The Big Picture

The most effective way to understand how an OS works is through abstraction - a fancy way of saying that you can ignore most of the details. Only need to know what a component does, not how it does it, i.e. how to interact with it, but don't need to know about it's grandpappy's name.

This chapter provides a high-level overview of the components that makes up a Linux system. 

## 1.1 Levels and Layers of Abstraction in a Linux System
Abstraction splits computing systems into components so as to make things easier to understand, but this doesn't work without organization. We arrange components into layers or levels, groupings of components according to where that component sits between the user and the hardware (e.g. web browser at the top lay, memory at the bottom layer; OS occupies most of the middle layers).

3 main levels:	
1. Hardware is the base.
- Memory (RAM), CPU
- Devices: disks and network interfaces
2. Kernel: the core of the OS
- The software in memory that tells the CPU what to do 
- manages the hardware, acts as interface between the hardware and any running program
- System calls, process management, memory management, device drivers
3. Processes: running programs managed by kernel
- make up the upper level or the user space
- GUI, servers, shell

Kernel runs in kernel mode: unrestricted access to the CPU and RAM. Powerful, but dangerous privelege that can allow a kernel process to crash the system (imagine if you could directly modify how your heart beats). The area restricted to kernel access = kernel space.

User processes run in user mode: a restricted subset of memory and safe CPU operations. User space = parts of main memory that the user processes can access, so if these processes mess up their room, it doesn't interfere with any other processes. Some user processes are given enough permissions/access to do serious damage (e.g. to disk).

## 1.2 Hardware: Understanding Main Memory
Main memory is gigantic storage area for bits, on which the CPU operates. It reads instructions and data from memory and writes back data. 

State is the current arrangement of bits. It helps to think of state abstractly as what the process has done or is currently doing (e.g. "loading").

Image is thus more commonly used to refer to the exact physical arrangement of bits (so that state can be transferred and loaded).

## 1.3 The Kernel
The kernel does nearly everything via main memory. It's responsible for splitting memory into subdivisions and managing them; each process gets it's split and must stay within it's boundaries. 

The kernel is in charge of managing tasks in 4 general system areas:
1. __Process management__: kernel determines which processes are allowed to use the CPU.
- Manages/schedule time sharing of the CPU across the multiple processes (i.e. performs context switches).
- Processes accomplish their tasks within the time slice between context switches
- Illusion of multitasking but just really fast context switching
	- CPU interrupts after timer, hands control to kernel
	- kernel records current state and memory for future resumption of process
	- kernel handles its own shit that requires CPU time
	- kernel schedules another process to run (chooses from a list), telling CPU how long to let it go for
	- kernel switches CPU to user mode and hands control of CPU to the process
2. __Memory__: kernel needs to keep track of all memory: what's allocated to which process, what's shared between processes, what's free to be allocated.
- Kernel has its own private area in memory that user processes can't access
- Each user process has its own section of memory and can't interfere with the sections of other processes
- Some user processes can share memory
- Some parts of memory in user processes are read-only
- System can use more memory than physically present via using auxilary disk
- Manages memory using virtual memory scheme that simplifies how it treats the memory. Can interact with memory in a simple way that maps to true physical representation via page tables. Is responsible for management and swapping of this memory map across processes.
3. __Device drivers__: kernel is the interface between hardware (e.g. disk) and processes. It operates the hardware (usually).
- Device drives are part of the kernel; that way they can present a uniform/standard interface to user processes (abstraction for software dev) 
- By adding an extra abstraction layer, the device interface with the kernel can be anything; the kernel interface with user processes must be uniform
4. __System calls and support__: processes use system calls (syscalls) to communicate with the kernel.
- Syscalls perform specific tasks that a user process alone cannot do 
- Opening, reading, writing files all involve syscalls
- To start a process, you `fork()` a copy of the current process and then call `exec(program)` which tells kernel to replace the copy of the process with a newly started `program` process
	- e.g. typing `ls` into shell forks a copy of the shell; the new copy calls `exec(ls)` to run `ls`
- Pseudodevices are software that look like devices to user processes (usually for practical reasons)
	- e.g. /dev/random is the kernel's random number generator "device"

## 1.4 User Space
Refers to the memory for the entire collection of running processes.

Processes can also be organized into a hierarchy depending on what type of tasks they perform. 
- Basic services are at the bottom level (closest to kernel). Small components that perform single, uncomplicated tasks (e.g diagnostic logging, communication bus, network config)
- Utility services are at in the middle. Larger components (e.g. mail server).
- Applications are at the top level, i.e. what the user interacts with (e.g. web browser, GUI).

Services use other components at the same or lower levels.

## 1.5 Users
User: an entity that can run processes and own files
- associated with a username, identified by the kernel with a corresponding userid
- concept exists to set permissions/boundaries: every user space process has a user owner and processes are run as the owner
- can terminate or modify only processes it owns
- can access only its files and those shared by others
	- groups are sets of users, used to allow file sharing
- root user CAN terminate and alter other users' processes and read any file on the local system => it's the superuser
	- still runs in the OS's user mode, not kernel mode