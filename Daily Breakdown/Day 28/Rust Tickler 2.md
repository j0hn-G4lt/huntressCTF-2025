- Still fluffing out details

```sh
# Loop processing 2 bytes at a time:
140004380:  imul   $0x19660d,%eax,%r8d      # r8d = seed * A (A = 0x19660D)
140004387:  add    $0x3c6ef35f,%r8d          # r8d += B (B = 0x3C6EF35F)
14000438e:  shr    $0x18,%r8d                # r8d >>= 24 (extract top byte)
140004392:  xor    %r8b,(%rbx,%rcx,1)        # XOR with first byte

140004396:  imul   $0x17385ca9,%eax,%eax     # seed = seed * C (C = 0x17385CA9)
14000439c:  add    $0x47502932,%eax           # seed += D (D = 0x47502932)
1400043a1:  mov    %eax,%r8d
1400043a4:  shr    $0x18,%r8d                # Extract top byte
1400043a8:  xor    %r8b,0x1(%rbx,%rcx,1)     # XOR with second byte

1400043ad:  add    $0x2,%rcx                 # i += 2 (next pair of bytes)
1400043b1:  cmp    %rcx,%rdx
1400043b4:  jne    0x140004380               # Loop if not done

# Handle odd byte if length is odd:
1400043bc:  imul   $0x19660d,%eax,%eax       # seed * A
1400043c2:  add    $0x3c6ef35f,%eax           # + B
1400043c7:  shr    $0x18,%eax                # >> 24
1400043ca:  xor    %al,(%rbx,%rcx,1)         # XOR last byte
```

- Script to extract and decode the blob

```python
import re, struct, subprocess
from pathlib import Path
from pwn import log

OBJ_START = "0x1400019c5"
OBJ_STOP = "0x140003200"

def extract_blob(binary):
    log.info("dumping movb immediates from %s", binary)
    text = subprocess.check_output(
        ["objdump", "-D", f"--start-address={OBJ_START}", f"--stop-address={OBJ_STOP}", str(binary)],
        text=True,
    )
    pairs = re.findall(r"movb\s+\$0x([0-9a-f]+),0x([0-9a-f]+)\(%rbp\)", text)
    blob = bytearray(max(int(off, 16) for _, off in pairs) + 1)
    for val, off in pairs:
        blob[int(off, 16)] = int(val, 16)
    log.success("extracted %d bytes", len(blob))
    return bytes(blob)

def parse_table(decoded):
    if decoded[:4] != b"HNTS":
        raise ValueError("bad magic")
    count = struct.unpack_from("<I", decoded, 4)[0]
    log.success("found header HNTS with %d entries", count)
    for idx in range(count):
        yield struct.unpack_from("<IIII", decoded, 8 + 16 * idx)

def lcg(seed, data):
    A, B, C, D, M = 0x19660D, 0x3C6EF35F, 0x17385CA9, 0x47502932, 0xFFFFFFFF
    buf = bytearray(data)
    i = 0
    while i + 1 < len(buf):
        tmp = (seed * A + B) & M
        buf[i] ^= (tmp >> 24) & 0xFF
        seed = (seed * C + D) & M
        buf[i + 1] ^= (seed >> 24) & 0xFF
        i += 2
    if i < len(buf):
        tmp = (seed * A + B) & M
        buf[i] ^= (tmp >> 24) & 0xFF
    return bytes(buf)

binary = Path("rust-tickler-2.exe")
blob = extract_blob(binary)

log.info("XORing blob with 0x33")
decoded = bytes(b ^ 0x33 for b in blob)

log.info("parsing entry table")
entries = list(parse_table(decoded))

targets = {
    0xAAAA: "favorite cat",
    0x83: "hint",
    0x7F: "flag",
    0xAAAAAA: "failure",
    0x0AAAAA: "entry_0",
    0x8F: "entry_1",
    0x51: "entry_5",
    0x63: "entry_6",
    0xAAAAAAAA: "entry_7",
    0x40: "entry_8",
    0xA9: "entry_9",
    0x9F: "entry_11",
    0xA1: "entry_12"
}
log.info("decrypting %d target entries (all)", len(targets))

for entry_id, offset, seed, length in entries:
    if entry_id not in targets:
        log.warning("unknown entry_id=0x%08x (skipping)", entry_id)
        continue

    payload = decoded[offset:offset + length]
    plain = lcg(seed & 0xFFFFFFFF, payload)
    log.info("id=0x%08x (%s)", entry_id, targets[entry_id])
    log.info("  cipher : %s", payload.hex())
    log.info("  text   : %s", plain.decode("utf-8", "replace"))
    log.info("  hex    : %s", plain.hex())
```

```r
python wd40.py
[*] dumping movb immediates from rust-tickler-2.exe
[+] extracted 616 bytes
[*] XORing blob with 0x33
[*] parsing entry table
[+] found header HNTS with 13 entries
[*] decrypting 13 target entries (all)
[*] id=0x000aaaaa (entry_0)
[*]   cipher : 22a5441866eaa6880fbb2cf785335aac62618fab2da471367e129451186f149f2b1ef24d
[*]   text   : Correct, but that is not the flag...
[*]   hex    : 436f72726563742c206275742074686174206973206e6f742074686520666c61672e2e2e
[*] id=0x0000008f (entry_1)
[*]   cipher : f2095cc905c3259a49df150f1cb8e2a211d664f1e3fe0e928dad
[*]   text   : Follow the White Rabbit...
[*]   hex    : 466f6c6c6f7720746865205768697465205261626269742e2e2e
[*] id=0x00aaaaaa (failure)
[*]   cipher : cc6b98246c6b2b43846e45e749423e00ff255b87335d379776b4e5261feed94b894e38e8
[*]   text   : Wrong, that`s not my favorite cat...
[*]   hex    : 57726f6e672c20746861742773206e6f74206d79206661766f72697465206361742e2e2e
[*] id=0x0000007f (flag)
[*]   cipher : b92e1fb46ac6340012c90a7e5bb93039585a2f8f4f3f7acfed29e153fd69074eadfdb61d4aa7
[*]   text   : flag{f59a5f604d236425490133c3fac89a88}
[*]   hex    : 666c61677b66353961356636303464323336343235343930313333633366616338396138387d
[*] id=0x0000aaaa (favorite cat)
[*]   cipher : 7cf0e0fe907d
[*]   text   : Bingus
[*]   hex    : 42696e677573
[*] id=0x00000051 (entry_5)
[*]   cipher : b09ed9a4a7d7c1ff7b0a4199dbe7c5caa04fe959cd51
[*]   text   : The flag is protected!
[*]   hex    : 54686520666c61672069732070726f74656374656421
[*] id=0x00000063 (entry_6)
[*]   cipher : 4f4c7b2294e22d0e0a2947065b066758cfd7da352011d004ca36896bffed64023df5ed7712c815be37fea58d2c3cc136b07792
[*]   text   : Do you think you`re going to find the flag in here?
[*]   hex    : 446f20796f75207468696e6b20796f7527726520676f696e6720746f2066696e642074686520666c616720696e20686572653f
[*] id=0xaaaaaaaa (entry_7)
[*]   cipher : 223a02970fb4601d8eb2e5f0d9b652475b106a07d5b78b
[*]   text   : Who is my favorite cat?
[*]   hex    : 57686f206973206d79206661766f72697465206361743f
[*] id=0x00000040 (entry_8)
[*]   cipher : 08b8b0cda4353497d4bede5fbf
[*]   text   : Crab tickler!
[*]   hex    : 43726162207469636b6c657221
[*] id=0x000000a9 (entry_9)
[*]   cipher : 3f3e8b08a114c7f6a0516d0ac85f61220f3b228e9ae5f3b5abc394e5f4d64ede077ad0
[*]   text   : Please don`t look in the icon file!
[*]   hex    : 506c6561736520646f6e2774206c6f6f6b20696e207468652069636f6e2066696c6521
[*] id=0x00000083 (hint)
[*]   cipher : 4b8a88d03ccc6b78de6b982642d08bcca9564797c8b06e52b9dc5fc6855d90c9d10f64c9357281aa35af683f9c86ff5f7025bb951f1c3deca5a7ccf1b3a1
[*]   text   : The flag is the hash of the name of a certain Infosec YouTuber
[*]   hex    : 54686520666c6167206973207468652068617368206f6620746865206e616d65206f662061206365727461696e20496e666f73656320596f755475626572
[*] id=0x0000009f (entry_11)
[*]   cipher : a68df6bc88040dc0bad529474f4ef0d49e8d484392657ea206
[*]   text   : The ducks are unionizing!
[*]   hex    : 546865206475636b732061726520756e696f6e697a696e6721
[*] id=0x000000a1 (entry_12)
[*]   cipher : 6ac997462bf30fc37207a35b591bbde8cd48486e8b0b2a5c296b2b
[*]   text   : I am totally a blue herring
[*]   hex    : 4920616d20746f74616c6c79206120626c75652068657272696e67
```
