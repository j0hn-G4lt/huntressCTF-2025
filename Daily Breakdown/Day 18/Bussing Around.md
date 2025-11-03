- Found a good example with similar scenario [HERE](https://notcicada.medium.com/write-up-the-magic-modbus-2692eaf5ee73)

```sh
tshark -n -q -r bussing_around.pcapng \
  -o tcp.desegment_tcp_streams:TRUE \
  -Y 'ip.src==172.20.10.2 && modbus.func_code==6 && (modbus.reference_num<2)' \
  -T fields -e modbus.regval_uint16 \
| tr -d '\n' \
| perl -0777 -ne 'while(/(.{8})/g){ print chr(oct("0b$1")) }' > mypants.zip && unzip mypants.zip

Archive:  mypants.zip
The password is 5939f3ec9d820f23df20948af09a5682 .
[mypants.zip] flag.txt password:
replace flag.txt? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
 extracting: flag.txt

cat flag.txt
flag{4d2a66c5ed8bb8cd4e4e1ab32c71f7a3}
```
