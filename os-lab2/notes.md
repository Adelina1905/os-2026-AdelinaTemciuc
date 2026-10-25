# Lab 2 - Meet the OS You Will Build

Done by: st. gr. FAF-242, Temciuc Adelina

## Part 2 - Use xv6 as the Unix it is

**(1) Three programs xv6 ships with**
- `cat` - shows the content of files;
- `echo` - prints text;
- `grep` - searches for text in its input.

There are also `forktest`, `kill`, `mkdir`, `rm`, `sh`, `wc` and `usertests`.

**(2) Which two OS features must exist for a pipe to work?**
Processes, because the shell starts a separate process for each side of the pipe,
and communication between processes, because the pipe is how the output of one
process goes into the input of the other. In `ls | grep c`, `ls` writes and `grep`
reads.

**(3) How does the xv6 shell compare to the Linux shell?**
It can do the basic things, like running programs and pipes, but it is much simpler
and has far fewer features than the Linux shell.

## Part 3 - Read the source

**(1) Which system calls does user/cat.c use?**
- `read()` - reads data from a file descriptor into a buffer;
- `write()` - writes data from a buffer to a file descriptor (for example the console);
- `open()` - opens a file and gives back a file descriptor;
- `close()` - closes a file descriptor;
- `exit()` - ends the process.

The main ones are `read()` and `write()`: cat reads a piece of the file and writes
it to the screen, again and again. All of them are declared in `user/user.h`.

**(2) Where is sys_read implemented?**
In `kernel/sysfile.c`, on line 69:

```
$ grep -n "sys_read\|sys_write" kernel/sysfile.c | head
69:sys_read(void)
83:sys_write(void)
```

**(3) Difference between kernel/ and user/**
`kernel/` is the operating system itself and does the low-level work, and `user/`
has normal programs that ask the kernel for things through system calls.

## Part 4 - sleep

`user/sleep.c` checks that it got exactly one argument, otherwise it prints
`usage: sleep <ticks>`, and then calls the `sleep()` system call with that number
of ticks. I added `$U/_sleep\` to UPROGS in the Makefile.

In August 2025 the xv6 authors renamed the system call `sleep()` to `pause()`, so on the
current xv6 the file calls `pause()` (see Lab 4). Tested inside xv6: `sleep 10` pauses and
returns to the prompt, `sleep` alone prints the usage message (Figure 4.4 in the report).
