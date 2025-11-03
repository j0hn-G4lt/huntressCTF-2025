- Identify the file type

```sh
file nimcrackme1.exe
PE32+ executable for MS Windows 5.02 (console), x86-64, 19 sections
```

- This is a 64-bit Windows executable

## String Analysis

Running `strings` on the binary reveals some function names:

```bash
strings nimcrackme1.exe | grep -i flag
```

- We find a function called `buildEncodedFlag`

## Disassembly with Radare2 (or whatever)

- Using radare2 to analyze the binary

```sh
r2 -q -A -e bin.relocs.apply=true -c 'afl | grep -i flag' nimcrackme1.exe
```

- Key functions found:
- `sym.buildEncodedFlag__crackme_u18` at `0x140011e78`
- `sym.main__crackme_u20` at `0x140012b84`

## Analyzing buildEncodedFlag

The `buildEncodedFlag` function constructs a 38-byte array by assigning individual bytes

```sh
Position 0:  0x28
Position 1:  0x05
Position 2:  0x0c
Position 3:  0x47
Position 4:  0x12
Position 5:  0x4b
... (38 bytes total)
```

- The disassembly shows a pattern like:

```as
mov byte [rax + 0x08], 0x28
mov byte [rax + 0x09], 0x05
mov byte [rax + 0x0a], 0x0c
...
```

- Complete byte array:

```py
[0x28, 0x05, 0x0c, 0x47, 0x12, 0x4b, 0x15, 0x5c,
 0x09, 0x12, 0x17, 0x55, 0x09, 0x4b, 0x42, 0x08,
 0x55, 0x5a, 0x45, 0x58, 0x44, 0x57, 0x45, 0x77,
 0x5d, 0x54, 0x44, 0x5c, 0x45, 0x13, 0x59, 0x5b,
 0x47, 0x42, 0x5e, 0x59, 0x16, 0x5d]
```

## Analyzing the Main Function

Examining `sym.main__crackme_u20` reveals:

1. It calls `buildEncodedFlag` to generate the encoded byte array
2. It calls `sym.xorStrings__crackme_u3` with:
   - The encoded flag
   - A key stored at address `0x140021ae0`

## Extracting the XOR Key

- Using radare2 to examine the key location:

```sh
r2 -q -e bin.relocs.apply=true -c 'px 40 @ 0x140021ae0+8' nimcrackme1.exe

- offset -   E8E9 EAEB ECED EEEF F0F1 F2F3 F4F5 F6F7  89ABCDEF01234567
0x140021ae8  4e69 6d20 6973 206e 6f74 2066 6f72 206d  Nim is not for m
0x140021af8  616c 7761 7265 2100 1700 0000 0000 0000  alware!.........
0x140021b08  e01a 0240 0100 0000                      ...@....
```

- The key structure in Nim strings is:
- First 8 bytes: length (little-endian)
- Following bytes: actual string data
- `0x140021b00` we find length: `0x17` (23 bytes)
- `0x140021ae8` we find the string: **"Nim is not for malware!"**

## XOR Decoding

- The flag is decoded by XORing each byte of the encoded array with the key

```py
enc = bytes([0x28, 0x05, 0x0c, 0x47, 0x12, 0x4b, 0x15, 0x5c,
             0x09, 0x12, 0x17, 0x55, 0x09, 0x4b, 0x42, 0x08,
             0x55, 0x5a, 0x45, 0x58, 0x44, 0x57, 0x45, 0x77,
             0x5d, 0x54, 0x44, 0x5c, 0x45, 0x13, 0x59, 0x5b,
             0x47, 0x42, 0x5e, 0x59, 0x16, 0x5d])

key = b'Nim is not for malware!'

dec = bytes([enc[i] ^ key[i % len(key)] for i in range(len(enc))])
print(dec.decode())
```

## Flag

**flag{852ff73f9be462962d949d563743b86d}**
