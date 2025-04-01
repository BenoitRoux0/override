# Level06

## Observations

```c
int auth(char *login, uint serial)
{
    size_t login_len;
    int ret;
    long traced;
    int i;
    uint hash;

    login_len = strcspn(login, "\n");
    login[login_len] = '\0';
    login_len = strnlen(login, 32);
    if ((int)login_len < 6)
        return 1;
    traced = ptrace(PTRACE_TRACEME);
    if (traced == -1)
    {
        puts("\x1b[32m.---------------------------.");
        puts("\x1b[31m| !! TAMPERING DETECTED !!  |");
        puts("\x1b[32m\'---------------------------\'");
        return 1;
    }
    hash = ((int)login[3] ^ 0x1337U) + 0x5eeded;
    for (i = 0; i < (int)login_len; i = i + 1)
    {
        if (login[i] < 32)
            return 1;
        hash = hash + ((int)login[i] ^ hash) % 0x539;
    }
    if (serial == hash)
        return 0;
    return 1;
}

int main(void)
{
    ...
    is_auth = auth(login, serial);
    if (is_auth == 0)
    {
        puts("Authenticated!");
        system("/bin/sh");
    }
    ...
}
```

We can see that the program calculates a hash using the username and compares it with the serial variable. If the two values are identical it launches a shell.

## Exploit

We can pregenerate the expected serial with a program that prints the hash of a given string.

[The program](./Ressources/exploit06.c)

```c
$ ./a.out beroux00
6234544
```

```shell
$ (echo "beroux00"; echo "6234544";  cat) | ./level06
cat /home/users/level07/.pass
```
