---
title: Android Anti-debugging Tricks - Part 1
date: 2021-04-25 15:29:53
categories: Reverse Engineering
tags: [Android, anti-debugging]
keywords: [Android, debug, anti-debugging]
description: This is a series of Android anti-debugging proposals, in particular for the dynamic debugging. Maybe a little more anti-anti-debugging tricks will be mentioned in this series.
---
## Ptrace Anti-debugging
The debuggers like gdb utilize the ptrace() function to attach to a process at runtime. Since only one process is allowed to do this at a time, having a call to ptrace() in your code can be used as an anti-debugging technique.

For the code example:

```C
#include <stdio.h>
#include <unistd.h>
#include <sys/ptrace.h>

void f (int n)
{
    printf ("Number: %d\n", n);
}

int main (int argc, char * argv[])
{
    if (ptrace(PTRACE_TRACEME, 0, 0, 0) < 0) {
        printf("Don't trace me!\n");
        return 1;
    }
    
    int i = 0;

    while (1)
    {
        f (i++);
        sleep (1);
    }
}
```
If we execute the program we can get the normal output every second, but if we debug it with *strace* the process will exit abnormally.
```shell
$ strace ./a.out
...
mprotect(0xb6fb4000, 4096, PROT_READ)   = 0
munmap(0xb6f57000, 94267)               = 0
ptrace(PTRACE_TRACEME)                  = -1 EPERM (Operation not permitted)
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x2), ...}) = 0
brk(NULL)                               = 0x82c000
brk(0x84d000)                           = 0x84d000
write(1, "Don't trace me!\n", 16Don't trace me!
)       = 16
exit_group(1)                           = ?
+++ exited with 1 +++
```
To bypass it, we can patch the ptrace part with NOP. And also, this anti-debugging method can be bypassed by using the *LD_PRELOAD* environment variable, which allows us to control the loading path of a shared library.

Create the following file:
```C
long ptrace(int request, int pid, void *addr, void *data) {
    return 0;
}
```
Compile it as a shared library:
```shell
gcc -shared -fPIC ptrace.c -o ptrace.so
```
Set the LD_PRELOAD environment variable:
In the shell:
```shell
export LD_PRELOAD=./ptrace.so
```
In gdb:
```shell
set environment LD_PRELOAD=./ptrace.so
```
The output will be like this:
```shell
(gdb) file a.out
Reading symbols from a.out...(no debugging symbols found)...done.
(gdb) r
Starting program: /tmp/a.out
Don't trace me!
[Inferior 1 (process 30567) exited with code 01]
(gdb) set environment LD_PRELOAD=./ptrace.so
(gdb) r
Starting program: /tmp/a.out
Number: 0
Number: 1
Number: 2
Number: 3
...
```

## Monitor TracerPid

TracerPid is a value which can reflect the status of the process at runtime. If the process is being attached to a debugger, the value of TracerPid won't be 0 or the PID of the target process. Instead, the TracerPid indicates the PID of the debugger process.

We have the following code:
```C
#include <stdio.h>
#include <unistd.h>

void f (int n)
{
    printf ("Number: %d\n", n);
}

int main (int argc, char * argv[])
{
    int i = 0;

    while (1)
    {
        f (i++);
        sleep (1);
    }
}
```
Execute the program and let's take a look at the process status:
```shell
$ cat /proc/31589/status
Name:	a.out
Umask:	0022
State:	S (sleeping)
Tgid:	31598
Ngid:	0
Pid:	31598
PPid:	30967
TracerPid:	0
Uid:	1000	1000	1000	1000
Gid:	1000	1000	1000	1000
FDSize:	256
Groups:	4 20 24 27 29 44 46 60 100 105 109 995 997 998 999 1000
NStgid:	31598
...
```
We can find that the TracerPid is 0 which means the program itself is not being debugged.

Debug the program with gdb:
```shell
(gdb) file ./a.out
Reading symbols from ./a.out...(no debugging symbols found)...done.
(gdb) r
Starting program: /tmp/a.out
Number: 0
Number: 1
Number: 2
Number: 3
...
```
Check the TracerPid:
```shell
$ cat /proc/31652/status
Name:   a.out
Umask:  0022
State:  S (sleeping)
Tgid:   31652
Ngid:   0
Pid:    31652
PPid:   31649
TracerPid:      31649
Uid:    1000    1000    1000    1000
Gid:    1000    1000    1000    1000
FDSize: 32
Groups: 4 20 24 27 29 44 46 60 100 105 109 995 997 998 999 1000 
NStgid: 31652
NSpid:  31652
NSpgid: 31652
...
```
Then let's trace the PID to which the TracerPid indicates:
```shell
$ ps aux | grep 31649
root       31649  1.3  0.8  42720 33484 pts/2    S    17:51   0:00 gdb
```

##

