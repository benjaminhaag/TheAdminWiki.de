---
date: '2026-02-14T10:47:39Z'
draft: true
title: 'Kerberos'
tags: ["kerberos"]
---


## Server 

```
sudo apt update
sudo apt install krb5-admin-server krb5-kdc
sudo krb5_newrealm
```

/etc/krb5.conf
```
[libdefaults]
    default_realm = LAB.ARPA

[realms]
    LAB.ARPA = {
        kdc = 192.168.2.113
        admin_server = 192.168.2.113
    }

[domain_realm]
    .lab.arpa = LAB.ARPA
    lab.arpa = LAB.ARPA
```

/etc/krb5kdc/kadm5.acl
```
*/admin *
```

## Join Domain

Server
```
sudo kadmin.local
    kadmin.local:   addprinc alice@LAB.ARPA
                    addprinc -randkey host/hostname.lab.arpa@LAB.ARPA
                    ktadd -k /etc/krb5kdc/hostname.keytab host/hostname.lab.arpa@LAB.ARPA
```

change password
```
sudo kadmin.local
    kadmin.local: cpw alice@LAB.ARPA
```

copy server:/etc/krb5kdc/hostname.keytab -> client:/etc/krb5.keytab

Client
```
chmod 6000 /etc/krb5.keytab

sudo kadmin
    kadmin: ktadd -k /etc/krb5.keytab host/hostname.lab.arpa@LAB.ARPA
```

## client

```
sudo apt update
sudo apt install krb5-user
```

```
kinit alice@lab.arpa
klist
kdestroy
```

## PAM

```
sudo apt update
sudo apt install libpam-krb5
```

/etc/pam.d/*

## SSH

/etc/ssh/sshd_conf
```
GSSAPIAuthentication yes
GSSAPICleanupCredentials yes
```

/etc/krb5.conf
```
    forwardable = true
```

/etc/ssh/ssh_config
```
Host *
GSSAPIAuthentication yes
GSSAPIDelegateCredentials yes
```