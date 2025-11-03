- Windows trash creates 2 types of files
  - `$I` = metadata
  - `$R` = content
- `The metadata might not be what it should be. Can you find a flag?`
- After reading only the `$I` files they contain pieces of flag hidden in the `file size` section but in garbled order
- Also reveals version `02` = WIN 10 format
- Based on the other files hinting `When did I throw this out!?!?` we can assume they need to be sorted by timestamp

```sh
xxd \$I01XCGF.txt
00000000: 0200 0000 0000 0000 3100 0000 0000 0000  ........1.......
00000010: 8073 bd23 5d08 2f00 1e00 0000 4300 3a00  .s.#]./.....C.:.
00000020: 5c00 5500 7300 6500 7200 7300 5c00 6600  \.U.s.e.r.s.\.f.
00000030: 6c00 6100 6700 5c00 4400 6500 7300 6b00  l.a.g.\.D.e.s.k.
00000040: 7400 6f00 7000 5c00 6600 6c00 6100 6700  t.o.p.\.f.l.a.g.
00000050: 2e00 7400 7800 7400 0000                 ..t.x.t...
```

```python
$ python3 << 'EOF'
import os
import struct

# Collect all metadata files
results = []
for filename in os.listdir('.'):
    if filename.startswith('$I') and filename.endswith('.txt'):
        with open(filename, 'rb') as f:
            data = f.read()
            # Extract file size (bytes 8-16) and timestamp (bytes 16-24)
            file_size = struct.unpack('<Q', data[8:16])[0]
            timestamp = struct.unpack('<Q', data[16:24])[0]
            results.append((file_size, timestamp))

# Sort by timestamp (deletion order)
results.sort(key=lambda x: x[1])

# Decode file sizes as ASCII characters (every 3rd one, since they repeat)
flag = ''.join(chr(results[i][0]) for i in range(0, len(results), 3))
print(flag)
EOF

flag{1d2b2b05671ed1ee5812678850d5e329}
```
