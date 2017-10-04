### Crack Hashed Password In Pyton

You can use this script to do a bruteforce attack on a hash list.

```python
#!/usr/bin/env python2
#
# Made By SakiiR 

import sys
import hashlib

if len(sys.argv) < 3:
    print '[+] Usage : ./hashcrack.py <wordlist> <hashfile>'
    sys.exit(0)

def md5hash(s):
    m = hashlib.md5()
    m.update(s)
    return m.hexdigest()

wordlist_path = sys.argv[1]
hashs_path = sys.argv[2]


words = open(wordlist_path).read().split('\n')
hashs = open(hashs_path).read().split('\n')
l = len(hashs)
i = 0
for h in hashs:
    if len(h) < 5:
        break
    if i % 10== 0:
        print '[^] Processing .. {}/{} - {}%'.format(i, l, i * 100 / l)
    for word in words:
        if h == md5hash(word):
            print '[+] FOUND ! {} -> {}'.format(h, word)
    i += 1
```

