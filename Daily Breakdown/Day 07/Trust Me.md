- Admin cannot execute TrustMe.exe
- Instead spawn a shell as System **not** Admin ( I used [PowerRun](https://www.sordum.org/downloads/?power-run))

```sh
xfreerdp3 /u:Administrator /p:'h$4#82PSK0BUBaf7' /v:10.1.200.163 /cert:ignore /dynamic-resolution +clipboard
```

![Solution](https://raw.githubusercontent.com/j0hn-G4lt/huntressCTF-2025/main/Images/Pasted%20image%2020251101224844.png)
