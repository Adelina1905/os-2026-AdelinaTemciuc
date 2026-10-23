# Lab 1 - The OS as a Resource Manager

Temciuc Adelina, FAF-242

Done on Linux under WSL.

## Part 1 - Files and directories

```
$ ls -l note.txt
-rw------- 1 adi adi 0 Sep 28 16:30 note.txt
```

**(1) Who owns the files, and which group?**
The owner is `adi` and the group is also `adi`. The first `adi` is the user, the second
one is the group.

**(2) What do the ten characters mean after chmod 600?**
`-rw-------`: the first `-` means it is a normal file. Then `rw-` means the owner can
read and write, `---` the group can do nothing, and `---` others can do nothing.
So only I can read and change the file.

**(3) Two directories under /**
- `/home` - the home folders of the users, for example `/home/adi` with my files.
- `/etc` - the configuration files of the system and of the services.

## Part 2 - Processes

```
$ ps aux | head
USER    PID %CPU %MEM VSZ   RSS TTY STAT START TIME COMMAND
root      1 0.0 0.1 21868 12504 ?   Ss   15:34 0:00 /sbin/init
```

**(1) What is the PID of process number 1, and what is it?**
Its PID is 1 and it runs `/sbin/init`, which here is systemd. It is the first process
the kernel starts, and it starts all the other ones.

**(2) How many processes were running on the idle system?**
About 53. `top` showed `Tasks: 53 total, 1 running, 52 sleeping, 0 stopped, 0 zombie`,
and `ps aux | wc -l` gave 55 (it counts the header line too, and it was run at another
moment). Almost all of them were sleeping.

**(3) What does "State:" say for the sleeping process?**
`State: S (sleeping)`. The process does not use the CPU, it just waits for its timer.

## Part 3 - Memory

```
$ free -h
          total    used    free    shared    buff/cache     available
Mem:      7.6Gi   736Mi   6.6Gi     21Mi        487Mi         6.9Gi
Swap:     2.0Gi      0B   2.0Gi
```

**(1) How much total RAM, and how much is free?**
7.6 GiB total and 6.6 GiB free (6.9 GiB available). `/proc/meminfo` shows the same:
7977176 kB total and 6922876 kB free.

**(2) What is swap, and how much is configured?**
Swap is space on the disk that the OS uses as extra memory when the RAM is not enough;
it moves pages that are not used there. I have 2.0 GiB of swap and 0 B is used, because
there was a lot of free RAM.

**(3) How much VmRSS does a bare sleep use?**
`VmRSS: 1792 kB`, so about 1.75 MiB. It surprised me a bit, because `sleep` does
nothing, but it still needs memory for its code, the shared libraries and the stack.

## Part 4 - Devices and storage

```
$ df -h
/dev/sdd             1007G    45G   911G     5% /
$ lsblk
sdd     8:48     0       1T   0 disk /
```

**(1) Which device is / mounted on?**
On `/dev/sdd`, a 1 TB disk.

**(2) One entry from /dev and what it stands for**
`crw------- 1 root root 5, 1 Sep 28 15:34 console` - this is the system console.
The `c` means it is a character device, so the data goes as a stream of characters.

**(3) What does "everything is a file" mean?**
Disks, terminals and other devices are shown as files in `/dev`, so you can work with
them with the same operations as with normal files (open, read, write).

## Conclusion

The OS manages four resources. **Files**: I saw them with `ls -l`, which shows the owner
and the permissions. **Processes**: I saw them with `ps aux` and `top`, and in
`/proc/<pid>/status`. **Memory**: I saw it with `free -h`, and the memory of one process
with `grep VmRSS /proc/<pid>/status`. **Devices**: I saw them with `lsblk`, `df -h`
and `ls -l /dev`.
