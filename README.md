<p align="center"> <img src="https://raw.githubusercontent.com/qeeqbox/local-file-inclusion/main/local-file-inclusion.png"></p>

A threat actor may trick a vulnerable target to include/retrieve local file

## Example #1
1. A threat actor sends a malicious request that includes the local file name to a vulnerable target
2. The vulnerable target includes the malicious local file as PHP and outputs it

## Code
#### Target-Logic
```php

#allow_url_include = On

<?php 
    $file = fopen($_GET["file"], 'r');
    echo fread($file, 1);
    fclose($file);
?>
```

#### Target-In
```
http://vulnerable.test/index.php?file=config.php
```

#### Target-Out
```
db_ip:10.0.0.10
db_name:r&d.db
db_user:root
```

## Impact
High

## Names
- Local file inclusion
- LFI

## Risk
- Read data

## Redemption
- Input validation
- Whitelist

## ID
2690f163-038a-4bc5-9ff3-3a02ba5f84ee

## References
- [Wikipedia](https://en.wikipedia.org/wiki/file_inclusion_vulnerability)
