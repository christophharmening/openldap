# Install OpenLdap Server

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
