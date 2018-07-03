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
- init is a user-space program (like any other program) in */sbin*
- responsible for starting and stopping the essential service processes on the system

3 major versions of init in Linux distributions
- __System V init__ ("sys-five") used by Red Hat and some other distributions
	- performs only one startup task at a time (easy to resolve dependencies, just slow)
	- systemd and Upstart allow many services to start in parallel
- __systemd__ is the emerging standard for Linux systems
	- goal oriented: you define a target along with its dependencies and when you want to reach it; systemd satisfies the dependencies and resolves the target
- __Upstart__ for Ubuntu
	- reactionary: receives events that trigger jobs that can produce more events, etc.

- service daemons start themselves from scripts; use `ps` to find the PID of a service daemon
	- systemd and Upstart init systems manage individual service daemons from the get-go
	- can defer the start of a service until it's abs. needed (lazy init)

## System V runlevels
- a certain base set of processes (e.g. `crond` and `udevd`) running define a runlevel
- span levels 0 through 6; a system spends most of its time in a single runlevel
- `who -r` to see your machine's current runlevel
- runlevels used to distinguish between system startup, shutdown, single-user mode, and console mode states
	- e.g. Fedora-based systems use runlevels 2-4 for text console, runlevel 5 to start a GUI login 
- becoming obsolete

## Identifying your init
- If you have */usr/lib/systemd* and */etc/systemd* directories, you have systemd
- If you have */etc/init* directory with several *.conf* files, you're probably running Upstart (except Debian 7 uses System V init)
- If neither of the above and you have an */etc/inittab* file, you're probably running System V init
- can use the init(8) manual page to help identify your version

## systemd
- handles the regular boot process and incorporates some standard Unix services (e.g. `cron` and `inetd`)
- can defer the start of services and OS features until they're needed

1. systemd loads its configuration
2. systemd determines its boot goal, usually named *default.target*
3. systemd determines all dependencies of the default boot goal, dependencies of those dependencies, and so on
4. systemd activates the dependencies and the boot goal
5. After boot, systemd can react to system events (e.g. uevents) and activate additional components

- flexible startup sequence

Units
- systemd can operate processes and services, but also mount filesystems, monitor network sockets, run timers, etc.
	- each type of capability is a unit type
	- each specific capabilitiy is a unit that you can activate
	- take a look at the systemd(1) manual page
- __Service units__ control the traditional service daemons on Unix system
- __Mount units__ control attachment of filesystems to the system
- __Target units__ control other units, usually by grouping them
	- default boot goal is a target unit grouping a bunch of service and mount units as dependencies
	- use `systemctl dot` to create a dependency tree diagram (or `systemd-analyze dot | dot -Tpng -o graph.png` with a dot program)

Dependencies
- 4 basic types of dependencies to allow for flexibility of the system
- __Requires__ indicates a strict dependency: systemd attempts to activate the dependency unit, and if it fails then systemd deactivates the dependent unit
- __Want__ indicates other units to activate in tandem: upon activating a unit, systemd activates the unit's Wants dependencies but doesn't care if those fail
	- contains failures to current unit
- __Requisite__ indicates other units that must already be active: systemd checks the status of the Requisite dependency and if it hasn't been activated, activation of the dependent unit fails
- __Conflicts__ indicates negative dependencies: systemd deactivates the Conflicts dependencies if they're active
- `systemctl show -p {type} {unit}` to view a unit's dependencies, where type specifies the type of dependencies you want to see (e.g. Wants or Requires)
- usually a unit and all its dependencies are activated at the same time (with Wants or Requires) for efficiency
	- can use dependency modifiers to specify an order: e.g. `Before=bar.target` in *foo.target* means that systemd activates *foo.target* before *bar.target*; also an After
- conditional keywords allow systemd to only activate the unit if the condition is true (however systemd still attempts to activate all the unit's dependencies)
	- `ConditionPathExists=p` true if the file path `p` exists in the system
	- `ConditionPathIsDirectory=p` true if `p` is a directory
	- `ConditionFileNotEmpty=p` true if `p` is a file and not zero-length

Configuration
- system unit directory (globally configured, */usr/lib/systemd/system*)
	- use `pkg-config systemd --variable=systemdsystemunitdir` to see path
- system configuration directory (local definitions, */etc/systemd/system*)
	- use `pkg-config systemd --variable=systemdsystemconfdir` to see path
	- __as a general rule, when given the choice between modifying something in */usr* and */etc*, always change */etc*__
- use `systemctl -p UnitPath show` to check current systemd configuration search path, including precedence

Unit Files
- derived from the XDG Desktop Entry Specification (for *.desktop* files, similar to *.ini* files) with section names in brackets `[]` and variable and value assignments/options in each section
- e.g. service units have a `[Service]` section containing variables detailing how to prepare, start and reload the service

Enabling units and the [Install] section
- when you enable a unit, systemd reads the `[Install]` section where you can put WantedBy or RequiredBy dependency options (e.g. look at `ssh.service` in */usr/lib/systemd/system/*)
	- all this does is it creates a symbolic link `ln -s '/usr/lib/systemd/system/{dependency-unit}' '/etc/systemd/system/{dependent-unit}.wants/{dependency-unit}'`
	- you can add symbolic links manually as simple way to add a dependency without modifying a unit file that may be overwritten in the future by software updates
- enabling a unit installs it to systemd's configuration, making semipermanent changes that survive reboots; __enabling a unit doesn't activate it__
	- if the unit file has an `[Install]` section, you must enable it with`systemctl enable`, otherwise the existence of the file is enough to enable it
	- activating a unit file with `systemctl start` just turns it on in the current runtime environment

Variables and Specifiers
- The `sshd.service` unit file shows uses of environment variables passed in by systemd: `$OPTIONS, $MAINPID`
- specifiers start with `%`
	- `%n` specifier is the current unit name
	- `%H` specifier is the current hostname
	- can modify unit name, e.g. for `getty`, create unit file `getty@.service`, to spawn multiple copies of a service
		- allows you to refer to units like `getty@tty1` where tty1 is the instance
		- `%I` specifier for these unit files refers to the instance

systemd Operation
- `systemctl` command lets you activate/deactivate services, list status, reload the configuration, etc.
	- by default the command lists the active units on your system
	- `--all` option to list all units (not just active)
	- `--full` for ful names of the units
- `systemctl status {unit}` to get status message about unit (e.g. firewall)
	- shows control group name for the unit, which you can view independently with `systemd-cgls`
	- displays recent info from the unit's log/journal, which you can view independently with `journalctl _SYSTEMD_UNIT={unit}`
- `systemctl start, stop, restart` to activate, deactivate and restart units
- `systemctl reload {unit}` reloads just the configuration for the unit
- `systemctl daemon-reload` reloads all unit configurations
- `systemctl list-jobs` lists current requests/jobs to activate, reactivate, and restart units
	- should be no jobs in systems that have been running a while (every unit should have completed its state transition)

Adding units to systemd
- create unit file in */etc/systemd/system* (so you won't get confused with what came with your distribution in */usr/lib/systemd/system* and won't get overwritten by upgrades)
- enable and activate the unit file
	- activate with `systemctl start {unit}`
	- enable with `systemctl enable {unit}`, only needed if the unit file has an `[Install]` section
- to remove a unit: 
	- deactivate with `systemctl stop {unit}`
	- disable `systemctl disable {unit}` (if unit has an `[Install]` section, this removes any dependent symbolic links)
	- remove the unit file if you'd like

systemd process tracking and synchronization
- a service can start in different ways, forking new instances of itself or daemonizes and detaches itself from the original process
- uses control groups (cgroups) to track process hierarchy
- `Type` option in service unit file to indicate its startup behavior
	- `Type=simple` - service process doesn't fork
	- `Type=forking` - service forks, original service process should terminate indicating to systemd that the service is ready
	- `Type=notify` - service sends notification to systemd when it's ready (with `sd_notify()`)
	- `Type=dbus` - service registers itself on the D-bus (Desktop Bus) when it's ready
	- `Type=oneshot` - service terminates completely when it's finished; add a `RemainAfterExit=yes` so that systemd regards the service as active even after its processes terminate
	- `Type=idle` - systemd doesn't start service until there are no active jobs

systemd on-demand and resource-parallelized startup
- unit startup can be delayed until its absolutely needed
1. Create a systemd unit (Unit A) for the system service as normal
2. Identify a system resource (e.g. network port/socket, file, or device) that Unit A uses to offer its services
3. Create anothe systemd unit (Unit R) to represent that resource (these should be socket, path or device units)
- operationally:
	- upon activation of Unit R, systemd monitors resource
	- when anything tries to access the resource, systemd blocks the resource, buffers the input to the resource (caches it)
	- systemd activates Unit A
	- when the service from Unit A is ready, it takes control of the resource, reads the buffered input, and runs normally

- if a service unit file has the same prefix as a `.socket` file, systemd knows to activate that service unit when there's activity on the socket unit
	- you can explicitly use `Socket=bar.socket` inside `foo.service` to have `bar.socket` hand its socket to `foo.service`
- */lib/systemd* contains support programs for units (e.g. `udevd` or `fsck`)
	- often simply runs the standard systems utilities and notifies systemd of the results

## Upstart
## System V init
## Shutting down your system
## The initial RAM filesystem
## Emergency booting and single-user mode