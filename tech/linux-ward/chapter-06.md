# Chapter 6. How User Space Starts

- init is the first user-space process the kernel starts
- that's when the memory and the CPU are finally ready for normal system operation
- relatively easy to change user space startup (no low-level programming)

User space starts in roughly this order:
1. init
2. essential low-level services (e.g. `udevd` and `syslogd`)
3. network configuration
4. mid and high-level services (e.g. cron, printing, etc.)
5. login prompts, GUIs, other high-level apps

## Introduction to init
## System V runlevels
## Identifying your init
## systemd
## Upstart
## System V init
## Shutting down your system
## The initial RAM filesystem
## Emergency booting and single-user mode