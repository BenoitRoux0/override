# Level00

after decompilation we can see that the program launch a shell if input is `5276`

```c
if (input == 5276)
  {
    puts("\nAuthenticated!");
    system("/bin/sh");
  }
```

```bash
(echo "5276"; <<< "cat /home/users/level01/.pass" cat) | ./level00
```
