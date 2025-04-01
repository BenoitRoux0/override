# Level03

## Observations

```c
int main(int argc, const char **argv, const char **envp)
{
    ...
    test(savedregs, 322424845);
    ...
}

int test(int a1, int a2)
{
  int result;
  int rd;

  switch ( a2 - a1 )
  {
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
    case 6:
    case 7:
    case 8:
    case 9:
    case 16:
    case 17:
    case 18:
    case 19:
    case 20:
    case 21:
      result = decrypt(a2 - a1);
      break;
    default:
      rd = rand();
      result = decrypt(rd);
      break;
  }
  return result;
}

int decrypt(char a1)
{
    ...
    if ( !strcmp((const char *)buffer, "Congratulations!") )
        return system("/bin/sh");
    ...
}
```

We can observe that the test function call decrypt with non random value if the first param is between 322424845 - 21 and 322424845 - 1.
Therefore we can bruteforce the 21 values.

## Exploit
```shell
(for i in $(seq 322424824 322424845); do echo -n "$i: "; (echo $i; <<< "cat /home/users/level04/.pass" cat ) | ./level03; done) | grep -v '*' | grep -v 'Password'
```
