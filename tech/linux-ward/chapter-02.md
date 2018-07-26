# Chapter 2. Basic Commands and Directory Hierarchy
Guide to the Unix commands and utilities that will be referenced later.

## 2.1 The Bourne Shell: /bin/sh
Shell: a program that runs commands (e.g. `ls`)
- also a small programming environment
- many important parts of the system are shell scripts (i.e. shell programs)
- see Chapter 11 for in-depth treatment on shell scripts
- many different flavors of Unix shells all dervied from the Bourne shell (/bin/sh)
- Linux uses the enhanced Bourne shell called bash (Bourne-again shell)
- `chsh` to change shells

## 2.2 Using the shell
The terminal application is a shell window.

```console
$ cat file1 file2 ...
```
The `cat` command outputs the contents of one or more files (concatenating them if needed), then exits. 

- Processes read data from input streams and write data to output streams
	- Standard input stream for shell commands is input from the terminal (e.g. if you just run `cat`, it will adopt interactive behavior accepting input from stdint)
	- Standard output stream for shell commands is output to the terminal
		- You can have output sent directly to files
	- Kernel gives each process a standard output stream where it can write its output
	- 3rd output stream called standard error

- CTRL + D on empty line stops the current standard input entry from the terminal (and often terminates a program)
- CTRL + C terminates a program regardless of its input or output

## 2.3 Basic commands
- `ls` lists contents of directory
	- `-l` gives detailed "long" listing (permissions, owner, group, file size, modification date of each file)
		- `ls -l file` gives detailed info only on the specified file in the directory
	- `-F` displays file type info
 - `cp file1 file2` copies `file1` to `file2`
 	- `cp file1 ... fileN dir` copies all N files to a directory
- `mv file1 file2` renames file1 to file2
	- `mv file1 ... fileN dir` moves all N files to a directory
- `touch file` creates an empty file'
	- if one already exists with that name, `touch` does not change it but does update the modification timestamp
- `rm file` to delete file
	- generally cannot be undone
- `echo text` prints text to the standard output
	- useful for finding expansions of shell globs (wildcards like "\*") and variables like $HOME


## 2.4 Navigating directories
- "/" is the root directory
- `mkdir dir` creates a new directory
- `rmdir dir` deletes an empty directory
	- `rm -rf dir` to delete a directory and all of its contents
	- can do serious damage (especially if run as root), `-r` specifies recursive delete, `-f` forces the delte operation
	- don't use the `-rf` falgs with globs such as a star (\*)
- Globbing (wildcards): simple pattern matching for file/directory names
	- wildcard \* matches any number of arbitrary characters
		- `echo *` prints a list of file sin the current directory
		- shell matches, substitutes the filenames that match (expansion), and runs the revised command
		- `*.*` only matches files that contain dots in their names, but Unix filenames don't need extensions and often don't have them, so this won't match every file
	- wildcard ? matches exactly one arbitrary character
	- Surround glob in single quotes to prevent expansion `echo '*'` prints the literal \*

## 2.5 Intermediate commands
- `grep` prints the lines from a file or input stream that match an expression
	- `grep root /etc/passwd` prints the lines in the */etc/passwd* file that contain the text `root`
	- `grep root /etc/*` prints the filename and matching line for all files in */etc* that contains the word root
	- `-i` for case-insensitive matches
	- `-v` for inverted search (prints all lines that DON'T match)
	- `egrep` or `grep -E` is an enhanced grep search
	- understand regex patterns: `.*` matches any number of characters and `.` matches one arbitrary character (see Mastering Regular Expressions or grep manual page)
- `less` prints output stream one screenful at a time, spacebar to go forward in the output, b to skip back one screenful, q to quit
	- enhanced version of `more`
	- `/word` to search forward in output for word
	- `?word` to search backwards
	- `n` to find next/previous occurence
	- You can forward output of any program to input of another program: e.g. `grep ie /usr/share/dict/words | less` forwards output of grep search to be displayed by less
- `pwd` to print name of current working directory
	- `pwd -P` to show true full path (even if obscured by symbolic links)
- `diff file1 file2` outputs the differences between 2 text files
	- `diff -u` is another format
- `file file` outputs the format of a file
- `find dir -name file -print` finds file in dir
	- accepts pattern matching 
	- `locate` can also find files but searches an index that the system builds periodically; if your file is newer than the index, `locate` won't find it
- `head file` and `tail file` show the first and last 10 lines of the file
	- `-n` changes the number of lines displayed to n (e.g. `head -5 /etc/passwd`)
	- `tail +n` prints lines starting at line n
- `sort file` puts lines of text file in alphanumeric order
	- `-n` sorts lines numerically (if the lines start with numbers)
	- `-r` reverses the order of the sort

## 2.6 Changing your password and shell
- `passwd` to change your password
	- easy way to create a password: pick a sentence, produce an acronym from it, modify with a number or punctuation => just remember the sentence
- `chsh` to change shell to `ksh` or `tcsh`

## 2.7 Dot files
- `ls -a` to see dot files and directories (names begin with ".")
	- e.g. .bashrc, .login or the directory .ssh
- shell globs don't match dot files unless you explicitly use `.??*`
	- the `??` is to exclude the match `.` and `..`, the current and parent directories 

## 2.8 Environment and shell variables
Shell variables contain the values of text strings
- used to keep track of values (e.g. paths) in scripts
- some shell vars control the way the shell behaves (e.g. `bash` shell reads the `PS1` variable before displaying the prompt)
- `NAME=value` to assign shell variable
- see Chapter 11 for more in-depth treatment

Environment variables are accessible to all programs the shell runs (i.e. most programs)
- `export NAME=value` to set environment variable
- used by many programs for configuration and options
	- e.g. can put your favorite `less` command-line options in the `LESS` env var and `less` will use those options when you run them (see manual pages)

## 2.9 The command path
`PATH` is a special env var containing the command path, a list of directories the shell searches when trying to locate a command like `ls`. It searches `PATH` in order and executes the first matching program
- the paths are separated by a colon ":"
- be careful to not accidentally wipe out your entire path. Typically recoverable if you simply restart terminal

## 2.10 Special characters
| Character | Name(s) 			| Uses 					|
|-----------|-------------------|-----------------------|
| `*` 		| asterisk, star 	| Regex, glob character |
| `.` 		| dot 			 	| Curr dir, file/hostname delimiter |
| `!` 		| bang 				| Negation, command history |
| `|` 		| pipe				| Command pipes 		|
| `/` 		| forward slash 	| Directory delimiter, search command |
| `\` 		| backslash 		| Literals, macros (never directories) |
| `$`		| dollar			| Variable denotation, end of line |
| `'` 		| tick, single quote| Literal strings 		|
| \`		| backtick, backquote| Command substitution |
| `"`		| double quote 		| Semi-literal strings 	|
| `^`		| caret 			| Negation, beginning of line |
| `~` 		| tilde, squiggle	| Negation, directory shortcut |
| `#`		| hash, sharp, pound| Comments, preprocessor, substitutions |
| `[] ` 	| square brackets 	| Ranges 				|
| `{ }`		| braces, curly brackets| Statement blocks, ranges |
| `_`		| underscore, under | Cheap substitute for a space |

## 2.11 Command-line editing
| Keystroke | Action 			| 
|-----------|----------------------------|
| CTRL-B 	| Move the cursor left (same as left arrow)   |
| CTRL-F 	| Move the cursor right (same as right arrow) |
| CTRL-P 	| View the previous command (same as up arrow) |
| CTRL-N 	| View the next command (same as down arrow) |
| CTRL-A 	| Move the cursor to beginning of line |
| CTRL-E 	| Move the cursor to end of line |
| CTRL-W 	| Erase the preceding word |
| CTRL-U 	| Erase from cursor to beginning of line |
| CTRL-K 	| Erase from cursor to end of line |
| CTRL-Y 	| Past erased text (e.g. from CTRL-U) |
| ESC-F 	| Move to beginning of next word |
| ESC-B		| Move to beginning of current word |

## 2.12 Text editors
Use vi or emacs

## 2.13 Getting online help
- `man ls` to get manual page for the `ls` command or any other shell command
	- kind of dense, takes patience to sift through
	- `man -k keyword` to search manual pages by keyword (e.g. some command to `sort`)

The manual pages are divided into sections:

| Section 	| Description				|
|-----------|---------------------------|
| 1 	 	| User commands 			|
| 2 		| System calls 				|
| 3 		| Higher-level unix programming library docs |
| 4 		| Device interface and driver info |
| 5			| File descriptions (system config files) |
| 6 		| Games 					|
| 7 		| File formats, conventions, and encodings (ASCII, suffixes, etc) |
| 8 		| System commands and servers |

- `man` normally displays the first manual page it finds matching a search term
	- you can specify a manual page by section, `man 5 passwd` displays the `/etc/passwd` file description as opposed to the `passwd` command
	- when someone refers to a manual page, the section number is specified in parenthesis (e.g. ps(1) or ping(8))

- `--help` or `-h` option on commands usually displays some summary info and options for the command

- GNU has more in-depth pages accessed via `info ls` (substitute command for `ls`)

- Some packages dump their documentation into */usr/share/doc*


## 2.14 Shell input and output
- To send output of a command to a file instead of the terminal:
	- `command > file` for any command and file; creates file if it doesn't exist, else it clobbers (erases) original
	- `set -C` in bash to avoid clobbering
	- `command >> file` to append output to file instead of overwriting it
- To send stdout of a command to stdin of another command, pipe them:
	- e.g. `head /proc/cpuinfo | tr a-z A-Z`
	- can chain as many piped commands as you want
- Error messages from commands are sent to stderr (usually the terminal)
	- can send stderr output to a file as well: `ls /fffff 2> file`
	- the 2 specifies the stream ID (1 for stdout (default), 2 for stderr)
	- you can send the stderr to the same place as stdout with `>&`, e.g. `ls /ffffff > file 2>&1`
- To channel a file to a program's stdin use `<`:
	- e.g. `head < /proc/cpuinfo`
	- `<` is not usually needed, since programs typically accept filenames as arguments (e.g. `head /proc/cpuinfo` works as well)

## 2.15 Understanding error messages
- Address the first error first, since this is likely to resolve subsequent errors

- Permission denied
	- usually occurs when you attemp to access a file you have insufficient priveleges for
	- also shows up when you try to execute a file that doesn't have the execute bit set (even if you can read the file)
- Operation not permitted
	- you're trying to kill a process that you don't own
- Segmentation fault, Bus error
	- program tried to access a part of memory it's not allowed to touch => OS killed it
	- bus error means program tried to access some memory in a particular way that it shouldn't
	- you might be giving a program some input it didn't expect

## 2.16 Listing and manipulating processes
- A process is a running program, each is given a numeric process ID (PID)
- `ps` lists all running processes with the info:
	- PID - process ID
	- TTY - terminal device where process is running
	- STAT - process status, i.e. what it's doing, where its memory resides (e.g. S for sleeping, R for running, see ps(1) manual page for more)
	- TIME - total CPU time used by process so far
	- COMMAND - names the command for the program, but note that the process can change this field from its original value
- We use the BSD style for `ps` options:

| Command | Description |
|---------|-------------|
| `ps x`  | Show all of your running processes |
| `ps ax` | Show all processes on the system, even those you don't own |
| `ps u`  | More detailed info on processes |
| `ps w`  | Show full command lines, not just what fits on one line |

- can combine options like with other commands (e.g. `ps auxw`)
	- can check on specific process `ps u $$` checks the current shell process (`$$` is a shell variable evaluating to the current shell's PID)
- The kernel can send signals to processes with `kill` command
	- `kill pid` sends the `TERM` terminate signal
	- `kill -STOP pid` freezes process instead
	- `kill -CONT pid` continue process again
	- `kill -INT pid` (interrupt) is how CTRL+C terminates a process that is running in terminal
	- `kill -KILL pid` is the most brutal way to terminate the process since it doesn't give the process a chance to clean up after itself; the OS terminates the process and forcibly removes it from memory. Use this as last resort
	- `kill -9 pid` is the same as using `-KILL` since the number 9 corresponds with the signal for `KILL`
- Job control
	- CTRL-Z to send a TSTP (similar to STOP) signal to a running program --> start the program again with `fg` (brings program to foreground) or `bg` (moves program to background)
	- `jobs` to see suspended processes
- Background processes
	- to have a process run in the background (freeing the terminal for your use) append a ` &` to the end of the command
	- the shell will print the PID of the new background process
	- background processes continue to run even after you log out
	- the background process may need stdin, in which case it'll freeze (try `fg` to bring it back) or terminate, and stdout, in which case it'll print output into your terminal even as you're using it (solution is to redirect its output)
- CTRL+L redraws the entire screen

## 2.17 File modes and permissions
- `ls -l` displays file permissions (first column)
- this string is the file's mode
	- the first character denotes the file type: a dash `-` denotes a regular old file, a `d` denotes a directory
	- the following are permission sets for user, group, and other (world), in that order (e.g. next 3 characters `rw-` indicate user can read, write, next 3 characters for the file's group permissions, next 3 characters for anyone else on the system)

| Character | Permission |
|-----------|-------------|
| `r` | Means that the file is readable |
| `w` | Means that the file is writable |
| `x` | Means that the file is executable (can be run as a program) |
| `-` | Means nothing |

- An `s` instead of `x` means the executable is *setuid*, meaning it executes as the file owner instead of you. Used to run as root in order to get the privileges needed by the program

- Modifying permissions with `chmod`
	- `chmod g+r file` adds read permission for the group to file (replace g with u for user, o for others, a for all users)
	- can use absolute permission modes to set all permission bits at once (e.g. `chmod 644 file`); essentially predefined sets of permissions

| Mode 	| Meaning | Used for |
|-------|---------|----------|
| 644 	| user: r/w; group, other: r | files |
| 600	| user: r/w; group, other: none | files |
| 755 	| user: r/w/x; group, other: r/x | directories, programs |
| 700 	| user: r/w/x; group, other: none | directories, programs |
| 711 	| user: r/w/x; group, other; x | directories |

- Directories also have permissions
	- can list contents of directory if readable
	- can only access a file in a directory if the directory's executeable

- `umask` sets default permissions to any new file you create
	- `umask 022` if you want everyone to be able to see all files and directories you create, `umask 077` otherwise
	- need to put this command in a startup file to make the setting persist in later sessions

- Symbolic links are files that point to another file or directory (i.e. shortcuts or aliases)
	- in the long/detailed directory listing, their type is `l` in the file mode
	- `ln -s target linkname` to create a symbolic link called `linkname` that links to `target`
	- without `-s`, you create a hard link (see 4.5)
	- prefer symbolic links

## 2.18 Archiving and compressing files
- `gzip` (GNU Zip) is a standard Unix compression program making copmressed `.gz` files
	- `gunzip file.gz` to uncompress
	- `gzip file` to compress

- `tar cvf archive.tar file1 file2 ...`
	- compresses multiple files and directories into one `archive.tar` file
	- `c` for create mode
	- `v` for vorbose diagnostic output (prints names of the files and directories in the archive as it encounters them)
	- `f` for file option, creating the `archive.tar` file
- `tar xvf archive.tar`
	- unpacks `tar` file
	- `x` for extract/unpack mode
	- can specify individual parts of the acrhive by entering the names of the parts at the end of the command line
	- does not remove the `.tar` file after extraction
- `tar tvf archive.tar`
	- prints Table of Contents for the archive
	- `t` for ToC mode
	- make sure all contents have a rational directory structure (i.e. all pathnames should start with the same directory, else it will dump all the files into the current directory and create a mess)
	- `p` use this option to preserve permissions (else your systems default `umask` permissions hold for the extracted files)

- to compress an archive, run `tar` then `gz` to create a `.tar.gz` or `.tgz` file
	- unzip with `gunzip` then `tar`
	- shortcut is to use `tar zxvf file.tar.gz` which calls `zcat file.tar.gz | tar xvf -`
		- the `zcat` program is the same as `gunzip -dc` which `d` decompresses and `c` sends the result to stdout, or the input of `tar` which extracts the contents from `-` stdin
	- can also do `zcvf` to create a `.tar.gz` and `ztvf` to print the ToC of a `.tar.gz` file

- can unpack Windows `.zip` files

- `bzip2` compresses text files a bit more than `gzip` so is preferred for source code
	- `tar` option using `j`
- `xz` is also popular

## 2.19 Linux directory hierarchy essentials
- *bin* contains ready-to-run programs, i.e. executables (e.g. shell scripts, binary files)
	- */bin* is where the basic Unix commands are kept
- */dev* contains device files (see Chapter 3)
- */etc* (et-see) contains core system configuation files (password, boot, device, networking, other setup files specific to the hardware, e.g. */etc/X11* for graphics card and window system configurations)
- */home* holds personal directories for users
- */lib* holds library files for executables to use
	- see chapter 15 for discussion on libraries
	- */lib* only holds shared libraries
- */proc* provides system statistics about current processes and some kernel parameters via a browseable directory-and-file interface
- */sys* provides a device and system interface like */proc*
- */sbin* for system executeables relating to system management (usually must be run as root)
- */tmp* is a storage area for small, temporary files that you don't care much about
 	- will be cleared when machine boots, or even regularly
- */usr* includes the bulk of the Linux system
	- */include* holds header files for the C compiler
	- */info* contains GNU info manuals
	- */local* where admins install their own software
	- */man* contains manual pages
	- */share* contains files that should work on other Unix machines, usually used nowadays for doc pages
- */var* is the variable subdirectory where programs record runtime info (e.g. system logging, user tracking, caches, other files that system programs create and manage)
	- */var/tmp* is not wiped on boot
- */opt* for additional 3rd-party software
- */media* is a base attachment point for removeable media (USBs)
- */boot* contains kernel boot loader files for initial startup stage

- Kernel is the program first loaded on booting, in */boot/vmlinuz*
	- loadable kernel modules are located under */lib/modules*

## 2.20 Running commands as the superuser
- `su` to open a shell as the superuser
	- commands not logged
- `sudo` allows you to run a command on your user shell as superuser
	- configure privileged users in the */etc/sudoers* file to determine who can run programs as superuser