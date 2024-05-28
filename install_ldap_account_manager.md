# Install ldap account manager (lam)
First thank your for your great product

Download deb package from
https://www.ldap-account-manager.org/lamcms/releases

Install package
```
apt-get install Downloads/name.deb
```

Got to url
> http://ldapserver.example.net

The basic configuration ist to do here "LAM configuration" USER: LAM PW: LAM
> Create a new Password for lam user



Now back to the main page and configure your server with "Edit Server profiles"
> create password

> Set servaddresee to ldaps://ldapserver.example.net

> Activate TLS = yes

> Tree suffix = dc=example,dc=net

> List of valid users* = cn=admin,dc=elozbw,dc=net


goto "Account Types"
> LDAP Suffix = ou=users,dc=example,dc=net

> Groups LDAP Suffix = ou=groups,dc=example,dc=net

