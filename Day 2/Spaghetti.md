- Create clean copy without comments

```sh
grep -v '^#' spaghetti > spaghetti_clean.ps1
```

- Reverse each function and encoding to reveal flags

```python
import re

# FLAG 1: MainFileSettings - Extract from hex data
with open('AYGIW.tmp', 'r') as f:
    hex_data = f.read().strip().replace('WT', '00')
binary_data = bytes.fromhex(hex_data)
flag1 = re.search(rb'flag\{[^}]+\}', binary_data).group().decode()

# FLAG 2 & 3: Decode binary-encoded variables
def decode(s):
    b = s.replace('~', '0').replace('%', '1')
    return ''.join(chr(int(b[i:i+8], 2)) for i in range(0, len(b), 8))

with open('spaghetti_clean.ps1', 'r') as f:
    ps = f.read()

# FLAG 2: My Fourth Oasis - Decode MyOasis4 then HTML entities
marker = '$MyOasis4 = (FonatozQZ("'
s = ps.find(marker) + len(marker)
e = ps.find('"', s)
decoded = decode(ps[s:e])
html_entities = ''.join(re.findall(r'(?:&#\d+;)+', decoded))
flag2_decoded = ''.join(chr(int(n)) for n in re.findall(r'&#(\d+);', html_entities))
flag2 = re.search(r'flag\{[^}]+\}', flag2_decoded).group()

# FLAG 3: MEMEMAN - Decode TDefo and extract flag
marker = '$TDefo = (FonatozQZ("'
s = ps.find(marker) + len(marker)
e = ps.find('".Replace', s)
decoded = decode(ps[s:e])
flag3 = re.search(r'flag\{[^}]+\}', decoded).group()

# Print flags
print(f"Flag 1 (MainFileSettings): {flag1}")
print(f"Flag 2 (My Fourth Oasis):  {flag2}")
print(f"Flag 3 (MEMEMAN):          {flag3}")
```
