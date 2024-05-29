Get the Root Credentials
```
 ldapsearch -LLL -Y EXTERNAL -H ldapi:/// -b  cn=config olcRootDN=cn=admin,dc=example,dc=net  dn olcRootDN olcRootPW
```

Create slapd pasword
```
slappasswd
```

Create ldif file changepwd
```
nano /etc/ldap/ldif-files/changepwd.ldif
```
```
dn: olcDatabase={2}bdb,cn=config
#olcRootDN: cn=admin,dc=example,dc=net
changetype: modify
replace: olcRootPW
olcRootPW: {SHA} (create your own) using above step
```

Modify Admin password 
``
ldapmodify -H ldapi:// -Y EXTERNAL -f /etc/ldap/ldif-files/changepwd.ldif
```
