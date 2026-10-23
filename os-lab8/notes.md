# Lab 8 - The Same Calls, Inside Your Own OS (xv6)

Temciuc Adelina, FAF-242

I added `user/run.c` and `user/pingpong.c` to xv6, and these two lines to UPROGS
in the Makefile:

```
	$U/_run\
	$U/_pingpong\
```

## Part 1 - run, inside xv6

```
$ run
parent: my pid is 3
child: my pid is 4
hello from the child
parent: child 4 ended with status 0
```

With `ls` instead of `echo`:

```
$ run
parent: my pid is 3
child: my pid is 4
.              1 1 1024
..             1 1 1024
README         2 2 2441
cat            2 3 35944
echo           2 4 34840
...
run            2 23 35104
pingpong       2 24 35104
console        3 25 0
parent: child 4 ended with status 0
```

With `nosuchprog`:

```
$ run
parent: my pid is 3
child: my pid is 4
exec failed
parent: child 4 ended with status 1
```

**1. The three calls in xv6 and on Linux. What is the same?**
In xv6: `fork()`, `exec()`, `wait()`.
On Linux: `fork()`, `execlp()`, `waitpid()`.
The idea is the same: fork makes a copy, exec turns the child into another program,
and wait makes the parent wait for the child. Only the names are a bit different,
`fork` is exactly the same.

**2. Who prints "hello from the child"?**
`echo` prints it. After exec the child is not `run` anymore, it is `echo`, and
"hello from the child" are just the arguments I gave to it.

**3. Why is the status 1 here and 127 on Linux?**
Because after the failed exec, in xv6 I have `exit(1)` and in Lab 7 on Linux I had
`exit(127)`. The parent just gets the number the child gave to exit.

## Part 2 - pingpong

```
$ pingpong
6: received ping
5: received pong
```

The child (pid 6) prints ping first, and then the parent (pid 5) prints pong.

**1. Why do we need TWO pipes?**
A pipe goes only one way, from the write end to the read end. If we use one pipe for
both directions, the parent could read back its own byte instead of the child's reply.
So one pipe is for parent -> child and the other is for child -> parent.

**2. Which call makes the parent wait for the reply?**
`read(c2p[0], &b, 1)`. If the pipe is empty, read blocks, so the parent stays there
until the child writes the byte back.

## Conclusion

The program from Lab 7 works almost the same in xv6, because xv6 has the same main
system calls: fork, exec, wait, pipe, read, write. This shows these calls are the basic
way any Unix-like OS starts programs. With pingpong I saw that two processes can talk
with pipes, and that `read` waits by itself when there is no data yet, so I did not need
any extra code to wait for the other process.
