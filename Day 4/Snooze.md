- UNIX compressed file (Z → zzz → snooze),

```sh
$ file snooze snooze: compressd data 16 bits
$ uncompress -c snooze > snooze_decompressed
$ cat snooze_decompressed

flag{c1c07c90efa59876a97c44c2b175903e}
```
