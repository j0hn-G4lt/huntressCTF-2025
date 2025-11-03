- `strings` reveals a garbled value
- Attempting XOR on it yields the flag with XOR key 81

```python
s = '7=06*gagg30d03gf2`f5g5dba3c0hhcd2c`4b,'

for shift in range(1, 128):
    decoded = ''.join(chr(ord(c) ^ shift) for c in s)
    if 'flag{' in decoded or 'FLAG{' in decoded or 'htb{' in decoded:
        print(f'XOR {shift}: {decoded}')
```

**XOR 81: flag{6066ba5ab67c17d6d530b2a9925c21e3}**
