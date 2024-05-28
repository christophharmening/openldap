# Install OpenLdap Server
Tested on Debian 12

## Install Server

Install required packages and 
> create an administrator password
> 
> create your domain like example.net
```
apt install slapd slapd-utils
```

Goto ldap configuration and create your ldif files
```
cd /etc/ldap
```
```
mkdir ldif-files
```
```
nano ldif-files/base.ldif
```
```
dn: ou=users,dc=example,dc=net
objectClass: organizationalUnit
ou: users
 
dn: ou=groups,dc=example,dc=net
objectClass: organizationalUnit
ou: groups
```

Load config file in ldap configuration
```
ldapadd -x -D cn=admin,dc=example,dc=net -W -f ldif-files/base.ldif
```

## Ldap TLS
Create your ca and your ldap server certificate like here
https://github.com/christophharmening/linux-misc/blob/main/ca_and_ssl.md

Create the tls config file for your ldap server
```
nano ldif-files/tls.ldif
```
```
dn: cn=config
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ldap/ssl/certs/ldapserver.example.net.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/ldapserver.example.net.key
-
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/myCA.crt
```

Load config in your ldap server
```
ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f ldif-files/tls.dlif
```

Activate your ldap server to use ldaps
```
nano /etc/default/slapd
```
```
...
SLAPD_SERVICES="ldap://127.0.0.1:389/ ldapi://127.0.0.1:389/ ldaps:///"
...
```

Restart ldap daemon
```
systemctl restart slapd
```

## Install ldap Client

Copy your myCA.crt certificate to /etc/ssl/certs/myCA.crt

Install required packages
```
apt install ldap-utils
```

Configure ldap client
```
nano /etc/ldap/ldap.conf
```
```
...
BASE    dc=example,dc=net
URI     ldaps://ldapserver.example.net
 
#SIZELIMIT      12
#TIMELIMIT      15
#DEREF          never
 
# TLS certificates (needed for GnuTLS)
TLS_CACERT      /etc/ssl/certs/myCA.crt
```

### Userlogin with nslcd
Install required packages
```
apt install libnss-ldapd
```

Check the nsswitch
```
nano /etc/nsswitch.conf
```
```
...
passwd:         files ldap
group:          files ldap
shadow:         files ldap
...
```

Configure the nslcd daemon
```
nano /etc/nslcd.conf
```
```
...
uri ldaps://ldapserver.example.net
...
base dc=example,dc=net
...
ssl on
tls_reqcert allow
tls_cacertfile /etc/ssl/certs/myCA.crt
...
```
Restart nslcd daemon
```
systemcrtl restart nslcd
```

Check if ldap users are present
```
getent passwd
```

Update your pam login to allow to create home directory on first login
```
pam-auth-update
```

### Userlogin with sssd
Install required packages
```
apt install sssd
```

Check the nsswitch file
```
cat /etc/nsswitch
```
```
...
passwd:         files sss
group:          files sss
shadow:         files sss
...
```

Configure sssd daemon
```
nano /etc/sssd/sssd.conf
```
```                                                                                            
[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldaps://ldapserver.example.net/
ldap_search_base = dc=example,dc=net
ldap_id_use_start_tls = true
ldap_tls_cacertdir = /etc/ssl/certs
cache_credentials = True
ldap_tls_reqcert = allow
enumerate = true

[sssd]
services = nss, pam, autofs
domains = default
```

Check permissions from sssd.conf
```
ls -l /etc/sssd/sssd.conf
-rw------- 1 root root 397 28. Mai 14:13 /etc/sssd/sssd.conf
```
Restart sssd daemon
```
systemctl restart sssd
```

Check if ldap users are present
```
getent passwd
```

Update your pam login to allow to create home directory on first login
```
pam-auth-update
```


