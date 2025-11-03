```sh
tshark -n -q -r bussing_around.pcapng \
  -o tcp.desegment_tcp_streams:TRUE \
  -Y 'ip.src==172.20.10.2 && modbus.func_code==6 && (modbus.reference_num<2)' \
  -T fields -e modbus.regval_uint16 \
| tr -d '\n' \
| perl -0777 -ne 'while(/(.{8})/g){ print chr(oct("0b$1")) }' > mypants.zip && unzip mypants.zip
```
