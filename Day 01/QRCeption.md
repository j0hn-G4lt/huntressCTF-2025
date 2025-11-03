**Challenge Prompt**

Wow, that's a big QR code! I wonder what it says!

![Challenge](https://raw.githubusercontent.com/j0hn-G4lt/huntressCTF-2025/main/Images/Pasted%20image%2020251101134137.png)

- Reused script from last year [MatryoshkaQR](https://github.com/j0hn-G4lt/huntressCTF-2024/blob/main/HuntressCTF2024/Day01/matryoshkaqr.md) which recursively decodes nested QR codes

```python
from PIL import Image
from pyzbar.pyzbar import decode
import os

def process(original):
    current = original
    count = 0

    while True:
        try:
            decoded = decode(Image.open(current))
            if decoded:
                qr_data = decoded[0].data.decode('utf-8')
                parse_data = bytes(qr_data, 'utf-8').decode('unicode_escape').encode('latin1')

                if parse_data.startswith(b'\x89PNG\r\n\x1a\n'):
                    with open(f"tmpimg{count}.png", 'wb') as f:
                        f.write(parse_data)
                    current = f"tmpimg{count}.png"
                    count += 1
                else:
                    print(f"{qr_data}")
                    break
            else:
                break
        except Exception as e:
            print(f"Error: {e}")
            break

    for i in range(count):
        os.remove(f"tmpimg{i}.png")

process('qrception.png') # Path to qrcode image
```

- Returns QR code which can be parsed with [cyberchef](<https://gchq.github.io/CyberChef/#recipe=Parse_QR_Code(false)&input=iVBORw0KGgoAAAANSUhEUgAAAYIAAAF9CAIAAACGXPgSAAAABGNJQ1ABDQABnGk7MgAAAAFzUkdCAK7OHOkAAAeMSURBVHic7d1Bbts6FEDRuPhby6q9uc77ATMGQ15KPmfcKrJsXHDAJz6ez%2BcXQOdPfQPAp5MhICZDQEyGgJgMATEZAmIyBMRkCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiIyRAQkyEgJkNATIaAmAwBMRkCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmIyBMT%2B2/aXvr%2B/t/2tyvP5fP0PXj%2BE4X8/3/Bbzj/j/B3e/pe8/zuyGgJiMgTEZAiIyRAQkyEgJkNATIaAmAwBMRkCYvt2UQ/l%2B2uHzt8%2Bm9/h/Jc4ucs5fwJffsnvsxoCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmIyBMQOGuYYWr0DPd%2BDP/%2B29vnXuU8%2BhNOmBA604RHlv%2BR3WQ0BMRkCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmJXGuZg9SjGT/7EpNVzBpebY%2BDLagjIyRAQkyEgJkNATIaAmAwBMRkCYjIExGQIiMkQEDPMsdXrUYMTjrVwMgf7WQ0BMRkCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmJXGua4waEL%2BazD8Bm%2BvsPhf5%2B8/k%2Bu0F5/Xn4DB7IaAmIyBMRkCIjJEBCTISAmQ0BMhoCYDAExGQJiB%2B2izncY5%2Ba3IA9NbjI%2BYY/y5D7vDR/BL/ldVkNATIaAmAwBMRkCYjIExGQIiMkQEJMhICZDQEyGgNjDC7ovZH5KYPUL5ze4/TvzP5DVEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiIyRAQkyEgtu9kjk8YRMjnADac7XG4Ez7g6rNDVtv/M7YaAmIyBMRkCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTErnQyx%2BpN7hsexeQ2/1w%2BkTO8h9XX/4nX9zB/NMjqKxjmAD6ODAExGQJiMgTEZAiIyRAQkyEgJkNATIaAmAwBsX0ncwydv0V99TTJDUYlhh9h/lu%2BuvNP5tjPagiIyRAQkyEgJkNATIaAmAwBMRkCYjIExGQIiO3bRT2/NzTfgJvvf92wEXzyT%2BR3eIOd6EP322huNQTEZAiIyRAQkyEgJkNATIaAmAwBMRkCYjIExGQIiB30SvyhyT3sJ7xpfPU2/NXHCgxtmHc54XtcasOsxmnjIFZDQEyGgJgMATEZAmIyBMRkCIjJEBCTISAmQ0BMhoDYQcMc%2BbkXQ/koxtD8M5w89yI/PWXe6m/hE84OeZfVEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiIyRAQkyEgtm%2BYY/WcwdCGMyGcezHv/HGQfOro/Ef0LqshICZDQEyGgJgMATEZAmIyBMRkCIjJEBCTISAmQ0Bs3zBHPmewYYf7%2BbvsJwcR5ucYbjDokE8d5dMkv85qCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiI7dtFPXSDd%2BZPmv%2BA99tf%2B3%2BTH2H%2BCeRb4e/HagiIyRAQkyEgJkNATIaAmAwBMRkCYjIExGQIiMkQEHts25l%2BwrvKV9/ADaye9rjBQ84f0ep35u//jqyGgJgMATEZAmIyBMRkCIjJEBCTISAmQ0BMhoCYDAGxfSdz3GBWI99Ev2Eg5vzTTa7%2BDIc3sOEOT2M1BMRkCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiI7RvmuIHVgwL5JMTX9B3OP6Lc6lmKDaMYlxsHsRoCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmIyBMT2DXNs2MV//qkSq50/bPEJAzGTTpu02MBqCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiIPT5wy2bo9Qbc1TuMf%2BUKk9cfmvwI%2BRMYOmGcYPIZ/jqrISAmQ0BMhoCYDAExGQJiMgTEZAiIyRAQkyEgJkNA7FavxM8Nd8GfPyhw/vvez38l/upv%2BX4DWFZDQEyGgJgMATEZAmIyBMRkCIjJEBCTISAmQ0BMhoDYvmGOofO3qK8eldjwBPJzL/JhkQ1nhxx%2B/T1/4i1WQ0BMhoCYDAExGQJiMgTEZAiIyRAQkyEgJkNATIaA2EHDHEP5HEBu/g7Pf4aT516cfzbJBvnZIe%2ByGgJiMgTEZAiIyRAQkyEgJkNATIaAmAwBMRkCYjIExK40zHED%2BSb6yYMrNgw6bDhaY/IGhlafvzI/q5GfEPMPqyEgJkNATIaAmAwBMRkCYjIExGQIiMkQEJMhIGYX9ZVseNX55BVWb/Adyjdhb3C/z2g1BMRkCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiIXWmYI3%2Bf/Lx8l/3ksEV%2B/1/r73DDxAz/sBoCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmIyBMQOGuY4YVCgNT8lcP7BGOfLTzfZ4LQ7tBoCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmIyBMQep23rBj6N1RAQkyEgJkNATIaAmAwBMRkCYjIExGQIiMkQEJMhICZDQEyGgJgMATEZAmIyBMRkCIjJEBCTISAmQ0BMhoCYDAExGQJiMgTEZAiIyRAQkyEgJkNATIaAmAwBMRkCYjIExGQIiMkQEJMhICZDQOwvBHcVOq1hMpYAAAAASUVORK5CYII>)

![Solution](https://raw.githubusercontent.com/j0hn-G4lt/huntressCTF-2025/main/Images/Pasted%20image%2020251101135052.png)

**flag{e1487f138f885bfef64f07cdeac96908}**
