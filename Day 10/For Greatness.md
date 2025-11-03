- Read the malware file
- Extract the `$FRczk` variable (first layer of obfuscation)
- Decode the octal/hex escape sequences
- Base64 decode the first layer
- Gzip decompress the first layer
- Extract the second base64-encoded payload from `$___________` variable
- Base64 decode the second layer (no gzip this time)
- Find the reversed flag in the email header
- Reverse it to get the actual flag

```php
<?php
$code = file_get_contents('j.php');
preg_match('/\$FRczk\s*=\s*"([^"]+)";/',$code,$m);
eval('$d="'.$m[1].'";');
$first = gzuncompress(base64_decode($d));
preg_match('/\$___________\s*=\s*([\'"])([A-Za-z0-9+\/=]+)\1/',$first,$n);
$second = base64_decode($n[2]);
preg_match('/ghost\+\}([a-f0-9]+)\{galf@/i',$second,$f);
echo 'flag{'.strrev($f[1])."}\n";
```

```sh
$ php solve.php
flag{f791310cef49f4d25d0778107033117f}
```