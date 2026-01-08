---
title: Bruteforce
---

## Hashcat

```bash
hashcat -a 0 hash.txt /usr/share/wordlists/rockyou.txt 
```

## Zip
https://book.hacktricks.xyz/generic-methodologies-and-resources/brute-force#zip

```bash
zip2john file.zip > zip.john
john zip.john
```

## PFX Certificates

```bash
# From https://github.com/crackpkcs12/crackpkcs12
apt install libssl-dev
crackpkcs12 -d /usr/share/wordlists/rockyou.txt ./cert.pfx
```
