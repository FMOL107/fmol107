---
title: Fuzzing
---

### ffuf

```bash
ffuf -c -fc 404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://hospital.htb/FUZZ 
```

```bash
ffuf -c -fc 404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://analysis.htb/ -H "Host: FUZZ.analysis.htb"
```

```bash
ffuf -c -fc 404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://internal.analysis.htb/users/list.php?FUZZ=test'
```

### wfuzz

```bash
wfuzz -c --hc 403,404 -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u https://hospital.htb/FUZZ
```

### gobuster

```bash
gobuster dir -u http://hospital.htb:8080/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20
```

### Dirsearch

```bash
dirsearch -u http://cat.htb
```