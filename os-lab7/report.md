# Lab 7 - Three Small Programs

Temciuc Adelina, FAF-242

## Part 1 - fork + exec + wait

```
gcc -Wall -Wextra -o run run.c && ./run
```

```
parent: my PID is 12479
child:  my PID is 12480, my parent is 12479
total 40
-rw-r--r--. 1 adelina adelina  1326 Oct 23 10:46 pipeline3.c
-rw-r--r--. 1 adelina adelina  1013 Oct 23 10:46 pipeline.c
-rw-r--r--. 1 adelina adelina   703 Oct 23 10:46 redirect.c
-rw-r--r--. 1 adelina adelina  3112 Oct 23 10:46 report.md
-rwxr-xr-x. 1 adelina adelina 12968 Oct 23 10:49 run
-rw-r--r--. 1 adelina adelina   911 Oct 23 10:42 run.c
-rw-r--r--. 1 adelina adelina   979 Oct 23 10:46 zombie.c
parent: child 12480 ended with status 0
```

With `date` instead of `ls`:

```
parent: my PID is 12488
child:  my PID is 12489, my parent is 12488
Fri Oct 23 10:49:08 AM EEST 2026
parent: child 12489 ended with status 0
```

With `nosuchprog`:

```
parent: my PID is 12495
child:  my PID is 12496, my parent is 12495
exec: No such file or directory
parent: child 12496 ended with status 127
```

**1. Why is `perror("exec")` printed only with nosuchprog?**
If exec works, the child becomes `ls` (or `date`) and the rest of my code is gone,
so the lines after `execlp` never run. With `nosuchprog` exec fails, so it comes back
and goes to the next line, which is `perror`.

**2. Which number does the child print as "my parent"?**
It prints 12479, which is the PID of the parent (the `./run` program). It is the same
number the parent printed in the first line.

**3. What status does the parent see?**
With `ls` it is 0 (everything ok). With `nosuchprog` it is 127, because of `exit(127)`
after the failed exec.

## Part 2 - A zombie, on purpose

```
gcc -Wall -Wextra -o zombie zombie.c
./zombie > /dev/null &
sleep 2; ps -o pid,ppid,stat,cmd
```

```
    PID    PPID STAT CMD
  12566    7287 Ss   bash
  12572   12566 S    ./zombie
  12574   12572 Z    [ls] <defunct>
```

After 20 seconds:

```
    PID    PPID STAT CMD
  12566    7287 Ss   bash
```

**1. Which letter does ls have in STAT?**
It has `Z`, it means zombie. The child already finished, but the parent did not call
wait yet, so it still stays in the process table.

**2. Is the `[ls] <defunct>` line still there after 20 seconds?**
No, it is gone. The line `waitpid(pid, &status, 0);` removed it, because the parent
finally took the exit status of the child.

## Part 3 - Redirection: ls -l > out.txt

```
gcc -Wall -Wextra -o redirect redirect.c && ./redirect
head -3 out.txt
```

```
open gave descriptor 1
done: the listing is in out.txt
total 56
-rw-r--r--. 1 adelina adelina     0 Oct 23 10:49 out.txt
-rw-r--r--. 1 adelina adelina  1326 Oct 23 10:46 pipeline3.c
```

**1. Which number did open give, and why?**
It gave 1. `open` always gives the smallest free number, and I closed 1 just before,
so 1 was the smallest free one (0 and 2 are still used).

**2. Why does the listing of ls not appear on the screen?**
`ls` always writes to descriptor 1. But now 1 is the file `out.txt`, not the screen,
so everything goes into the file.

**3. To redirect the errors, which descriptor would I close?**
I would close 2 (stderr) before `open`.

## Part 4 - A pipe: ls /etc | wc -l

```
gcc -Wall -Wextra -o pipeline pipeline.c && ./pipeline
ls /etc | wc -l
```

```
279
279
```

After making `close(p[1]);` a comment in the parent, the program does not stop and I
had to press Ctrl-C. While it was stuck, only `wc` was still alive:

```
    PID    PPID STAT CMD
  12536   12534 S    wc -l
```

**1. Did they give the same number?**
Yes, both printed 279.

**2. Which child never ends?**
`wc`. The `ls` child finishes normally.

**3. What is wc waiting for?**
`wc` waits for the end of the input (EOF), and it never comes because the parent
still has the write end `p[1]` open.

## Optional - pipeline3.c (ls /usr/bin | grep x | wc -l)

```
$ ./pipeline3
221
$ ls /usr/bin | grep x | wc -l
221
```

Both numbers are the same.

## Conclusion

In this lab I saw how the shell runs a command: it uses `fork` to make a copy, `exec`
to turn the copy into another program and `wait` to wait for it. If the parent does not
wait, the child stays as a zombie. I also saw that redirection and pipes work only by
changing descriptors 0 and 1 before exec, and the program itself (`ls`, `wc`) does not
know about it. And it is important to close the pipe ends you do not use, otherwise
the reader waits forever.
