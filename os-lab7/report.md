# Lab 7 - Three Small Programs

Done by: st. gr. FAF-242, Temciuc Adelina

## Part 1 - fork + exec + wait

```
gcc -Wall -Wextra -o run run.c && ./run
```

```
parent: my PID is 46466
child:  my PID is 46467, my parent is 46466
total 52
-rwxr-xr-x. 1 adelina adelina 12872 Oct 25 17:47 pipeline3
-rw-r--r--. 1 adelina adelina  1326 Oct 25 17:45 pipeline3.c
-rw-r--r--. 1 adelina adelina  1013 Oct 25 17:47 pipeline.c
-rw-r--r--. 1 adelina adelina   703 Oct 25 17:45 redirect.c
-rwxr-xr-x. 1 adelina adelina 12968 Oct 25 17:47 run
-rw-r--r--. 1 adelina adelina   911 Oct 25 17:46 run.c
-rw-r--r--. 1 adelina adelina   979 Oct 25 17:45 zombie.c
parent: child 46467 ended with status 0
```

With `date` instead of `ls`:

```
parent: my PID is 45754
child:  my PID is 45755, my parent is 45754
Sun Oct 25 05:46:11 PM EET 2026
parent: child 45755 ended with status 0
```

With `nosuchprog`:

```
parent: my PID is 45824
child:  my PID is 45825, my parent is 45824
exec: No such file or directory
parent: child 45825 ended with status 127
```

**1. Why is `perror("exec")` printed only with nosuchprog?**
If exec works, the child becomes `ls` (or `date`) and the rest of my code is gone,
so the lines after `execlp` never run. With `nosuchprog` exec fails, so it comes back
and goes to the next line, which is `perror`.

**2. Which number does the child print as "my parent"?**
It prints 46466, which is the PID of the parent (the `./run` program). It is the same
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
  45857   27608 Ss   bash
  45888   45857 S    ./zombie
  45890   45888 Z    [ls] <defunct>
  45900   45857 R+   ps -o pid,ppid,stat,cmd
```

After 20 seconds:

```
    PID    PPID STAT CMD
  45857   27608 Ss   bash
  45957   45857 R+   ps -o pid,ppid,stat,cmd
[1]   Done                    ./zombie > /dev/null
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
total 68
-rw-r--r--. 1 adelina adelina     0 Oct 25 17:46 out.txt
-rw-r--r--. 1 adelina adelina  1326 Oct 25 17:45 pipeline3.c
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
282
282
```

After making `close(p[1]);` a comment in the parent, the program does not stop. I ran it
in the background, and while it was stuck only `wc` was still alive:

```
./pipeline &
sleep 2; ps -o pid,ppid,stat,cmd --ppid $!
    PID    PPID STAT CMD
  46181   46178 S    wc -l
kill -INT $!
282
[1]   Interrupt               ./pipeline
```

`kill -INT` sends the same signal as Ctrl-C. When the parent died, its copy of `p[1]` was
closed, so `wc` finally got EOF and printed 282.

**1. Did they give the same number?**
Yes, both printed 282.

**2. Which child never ends?**
`wc`. The `ls` child finishes normally.

**3. What is wc waiting for?**
`wc` waits for the end of the input (EOF), and it never comes because the parent
still has the write end `p[1]` open.

## Optional - pipeline3.c (ls /usr/bin | grep x | wc -l)

```
$ ./pipeline3
248
$ ls /usr/bin | grep x | wc -l
248
```

Both numbers are the same.

## Conclusion

In this lab I saw how the shell runs a command: it uses `fork` to make a copy, `exec`
to turn the copy into another program and `wait` to wait for it. If the parent does not
wait, the child stays as a zombie. I also saw that redirection and pipes work only by
changing descriptors 0 and 1 before exec, and the program itself (`ls`, `wc`) does not
know about it. And it is important to close the pipe ends you do not use, otherwise
the reader waits forever.
