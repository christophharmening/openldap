# Backup and restore your OpenLdap Server
Tested on Debian 12

Our backupdestination is a mounted nfs filesystem on /backups

## Backup

### OpenLdap

Disable your ldap server
```
systemctl stop slapd
```

Backup your config
```
slapcat -n 0 -l /backups/ldap/config.example.net-$(date +%y%m%d).ldif
```

Backup your data
```
slapcat -n 0 -l /backups/ldap/data.example.net-$(date +%y%m%d).ldif
```
Backup your Serverconfig
```
cp /etc/defaults/slapd /backups/ldap/slapd
```
If you use an tls and use the same fqdn for your new server 
you must backup you ca an yout server certificates!!!

### ldap account manager
Goto /usr/share/ldap-account/manager/config
and backup your files
```
tar cvfz lam.backup-$(date +%y%m%d).tar.gz *.conf config.cfg pdf/ profiles/
```
```
cp lam.backup-$(date +%y%m%d).tar.gz /backups/ldap/
```

## Restore

### OpenLdap

Disable your ldap server
```
systemctl stop slapd
```

Restore your server configuration
```
cp /backups/ldap/slapd /etc/default/slapd
```

Backup your default files
```
cp -r /etc/ldap/slapd.d /etc/ldap/slapd.d.original
```
```
cp -r /var/lib/ldap /var/lib/ldap.original
```

Clear your old configuration
```
rm -rf  /etc/ldap/slapd.d/*
```
```
rm -rf /var/lib/ldap/*
```

Restore your config
```
slapadd -n 0 -F /etc/ldap/slapd.d/ -l /backups/config.example.net-$(date +%y%m%d).ldif
```
```
chown -R openldap: /etc/ldap/slapd.d
```

Restore your data
```
slapadd -n 1 -F /etc/ldap/slapd.d/ -l /backups/data.example.net-$(date +%y%m%d).ldif
```
```
chown -R openldap: /var/lib/ldap
```

If you use an tls encryption, you must restore your certificates an make sure the are readable by openldap user and group.


Start your openldap daemon
```
systemctl start slapd
```

### Restore ldap account manager
Goto /usr/share/ldap-account/manager/ and backup the config folder
```
cp -r config config.original
```
go into config folder and copy your tar.gz file into it
```
cp /backups/lam.backup-$(date +%y%m%d).tar.gz
```
extract
```
tar xvfz lam.backup-$(date +%y%m%d).tar.gz
```
restart apache2
```
systemctl restart apache2
```
