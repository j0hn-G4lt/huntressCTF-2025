## Challenge Prompt

vx-underground, widely known across social media for hosting the largest collection and library of cat pictures, has been plagued since the dawn of time by people asking: *"what's the password?"*

Today, we ask the same question. We believe there are secrets shared amongst the cat pictures... but perhaps these also lead to just more cats.

Uncover the flag from the file provided.

---

- Extracting the zip we have:
  - **prime_mod.jpg**
  - **flag.zip** - password protected
  - Directory "**Cat Archive**"
- Inspecting the **prime_mod.jpg** we find User Comment PRIME MODULUS

```sh
$ exiftool -A prime_mod.jpg

*snip*

User Comment                    : Prime Modulus: 010000000000000000000000000000000000000000000000000000000000000129

*snip*
```

- [Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)
- Metadata in each image [1-457]
- So we extract and order them

```sh
$ exiftool -a * | grep "User Comment" | awk -F': ' '{print $2}' | sort -n > comments.txt
```

- Lagrange python decode

```python
P = 0x10000000000000000000000000000000000000000000000000000000000000129
xs, ys = [], []
for line in open("comments.txt"):
    if line := line.strip():
        i, h = line.split("-", 1)
        xs.append(int(i)); ys.append(int(h, 16))
inv = lambda a: pow(a, -1, P)
w = [1] * len(xs)
for i in range(len(xs)):
    for j in range(len(xs)):
        if i != j: w[i] = (w[i] * (xs[i] - xs[j])) % P
    w[i] = inv(w[i])
num = den = 0
for i in range(len(xs)):
    t = (w[i] * inv((-xs[i]) % P)) % P
    num = (num + ys[i] * t) % P; den = (den + t) % P
s = (num * inv(den)) % P
print(s.to_bytes((s.bit_length() + 7) // 8 or 1, "big").decode("ascii"))
```

```sh
$ python3 decode.py
*ZIP password: FApekJ!yJ69YajWs
```

- Unzip **flag.zip** to reveal a new file **cute-kitty-noises.txt** filled with meows
- Script that counts the number of Meows between semicolons, and uses that count as ASCII if possible

```python
import re

chunks = open("cute-kitty-noises.txt").read().strip().split(";")
result = "".join(chr(c) if (c := len(re.findall(r'Meow', s))) == 10 or 32 <= c <= 126 else "" for s in chunks)
print(result)
```

```sh
$ python3 decode_meow.py
malware is illegal and for nerds
cats are cool and badass
flag{35dcba13033459ca799ae2d990d33dd3}
```
