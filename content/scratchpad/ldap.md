---
date: '2026-02-12T23:42:32Z'
draft: true
title: 'Ldap'
tags: ["ldap"]
---

> [NOTE]
> JDK 21 
> [ApacheDirectoryStudio](https://directory.apache.org/studio/download/download-windows.html)
> Port 389 TCP

```
sudo apt update
sudo apt install slapd ldap-utils
    -> admin Password
sudo dpkg-reconfigure sladp
    -> Omit OpenLDAP server configuration: no
    -> DNS domain name: lab.local
    -> Organization name: Lab Local
    -> Administrator password: password
    -> Do you want the database to be removed when slapd is purged?: yes
    -> Move old database?: no
```

check config:
```
sudo ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=config
```

## ADS
user: olcRootDN
password: password

## LDAP
User federation -> LDAP
Mapper -> groups

## Hash passwords by default

```
dn: cn=config
changetype: modify
replace: olcPasswordHash
olcPasswordHash: {SSHA}  
```
```
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f password_hash.ldif
```