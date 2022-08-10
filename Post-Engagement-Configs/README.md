# Documentation Post Engagement

***After initial engagement, stakeholder expressed concerns about managing multiple acccounts. A LDAP solution was deteermined to be a viable option***


## OpenLDAP Installation

Reference: [LDAP Configuration](https://www.thegeekstuff.com/2015/01/openldap-linux/)
***not the other resourced used to configure***

### OpenLDAP Configuration

### LDAP Configuration

```bash

dnf install openldap-servers
dnf install openldap-clients

vi /etc/hosts
# add ldap server to the /etc/hosts file

cat /usr/lib/systemd/system/slapd.service
# check services
# ExecStart=/usr/sbin/slapd -u ldap -h "ldap:/// ldaps:/// ldapi:///"


vi /etc/openldap/ldap.conf

# add certificates to /etc/openldap/certs/
cd /etc/openldap/slapd.d/cn=config
vi 'olcDatabase={2}mdb.ldif'

# changed all for the my-domain to juniata
# changed all of the com to edu
olcRootDN: cn=Manager,dc=juniata,dc=edu

# Make necessary changes
vi /etc/openldap/slapd.d/cn=config.ldif
vi /etc/openldap/slapd.d/cn=config/olcDatabase={2}bdb.ldif

# Update schema
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif


```
#### Change the password

```bash
slappasswd
```
***modify password ldif - changepassword.ldif***
```
#  changepassword.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {ADD_SLAPPASSWD_HASH}
```
#### Testing and Start-Stop-restart

```bash
slaptest -u

service slapd start
systemctl start openldap
systemctl stop openldap
systemctl restart openldap

```
#### Initial search

```bash

ldapsearch -x -b "dc=juniata,dc=edu"

ldapsearch -x -W -D "cn=Manager,dc=juniata,dc=edu" -b "dc=juniata,dc=edu" "(objectclass=*)"

```

#### update_SSL.ldif

```
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/{CERTCA}.crt
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/s{DOMAINCERT}.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/{DOMAINKEY}.key
```
#### update SSL

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f updateSSSL.ldif
```
#### Commands to update LDAP server

```bash
ldapadd -x -W -D "cn=Manager,dc=juniata,dc=edu" -f {FILENAME}
```

#### base.ldif (sample)

```
dn: dc=juniata,dc=edu
objectClass: dcObject
objectClass: organization
o: juniata.edu
dc: juniata

dn: o=juniata.edu,dc=juniata,dc=edu
objectClass: organization
objectClass: top
o: juniata.edu

dn: ou=People,o=juniata.edu,dc=juniata,dc=edu
objectClass: organizationalUnit
objectClass: top
ou: People

dn: ou=Groups,o=juniata.edu,dc=juniata,dc=edu
objectClass: organizationalUnit
objectClass: top
ou: Groups

```

#### user.ldif (sample)

```
dn: uid=joeuser,ou=People,o=juniata.edu,dc=juniata,dc=edu
uid: joeuser
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
sn: User
givenName: Joe
cn: Joe User
userPassword: CrazyPassword
uidNumber: 100
gidNumber: 1700
loginShell: /bin/bash
homeDirectory: /home/joeuser

```

### Test OpenLDAP Server

```bash

netstat -plane |grep ":389"
netstat -plane |grep ":636"

firewall-cmd --permanent --add-port=389/tcp
firewall-cmd --add-service={ldap,ldaps}
firewall-cmd --runtime-to-permanent

```

### Python Script

[Simple csv to LDAP upload script](python-script/README.md)


## Zeppelin Configuration

### Removed from the Shiro Configuration
Shiro would not allow for multi-authentication
Defaulted to ldap only over configuration

```
# RE-ENABLE - SHIRO in the config
# username01 = {PASSWORD}, admin
# username02 = {PASSWORD}, admin
# username03 = {PASSWORD}, admin
# username04 = {PASSWORD}, admin
# username05 = {PASSWORD}

```
### Current Configuration

found in the - /etc/zeppelin/conf/shiro.ini file

```
### A sample for configuring LDAP Directory Realm

ldapRealm = org.apache.zeppelin.realm.LdapGroupRealm
## search base for ldap groups (only relevant for LdapGroupRealm):

ldapRealm.contextFactory.environment[ldap.searchBase] = dc=juniata,dc=edu
#dc=COMPANY,dc=COM

ldapRealm.contextFactory.url = ldap://jc-hadoop.juniata.edu:389
#ldap.test.com:389
# Used 389 since it was on localhost  - use 636 for machines outside /32

ldapRealm.userDnTemplate = uid={0},ou=People,o=juniata.edu,dc=juniata,dc=edu
#uid={0},ou=Users,dc=COMPANY,dc=COM

ldapRealm.contextFactory.authenticationMechanism = simple
ldapRealm.contextFactory.systemUsername = cn=Manager,dc=juniata,dc=edu
ldapRealm.contextFactory.systemPassword = {SECRET_PASSWORD}
```


### Management Tools

* Apache Directory Server was tested against the installation - successful
