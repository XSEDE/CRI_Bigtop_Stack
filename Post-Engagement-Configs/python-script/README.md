# Supporting Python Script

```python
import sys
import ldap
import ldap.modlist as modlist
import csv

# ========================================
# USAGE NOTE
# CSV Format (note: no quotation marks and the CRLF
# =========================================
# first_name, last_name, user_name, password
# Bugs, Bunny, bbugs, Welcome2022a
# Mickey, Mouse, mmouse, Welcome2022b
# Daffy, Duck, dduck, Welcome2022c
# =========================================

# =========================================
# CHANGE THESE PARAMETERS
# =========================================
# Academic Year
ay = "2023"

# Semester
# Fall - 01
# Winter - 02
# Spring - 03
# Summer - 04
semester = "01"

# increment by 1
loadset = "10"
# ===========================================


# LDAP Process
# https://www.python-ldap.org/en/python-ldap-3.4.2/reference/ldap-dn.html

#  SERVER CONFIGS - DO NOT EDIT
conn = ldap.initialize("ldaps://jc-hadoop.juniata.edu:636/")
bdn = "cn=Manager,dc=juniata,dc=edu"
pwd = "{SECRET_PASSWORD}"
conn.simple_bind_s(bdn,pwd)
attrs = {}

# CSV Function
csvfs = open('newusers.csv')
csvreader = csv.reader(csvfs)

header = []
header = next(csvreader)
rows = []
# print("LDIF Safety Net\n")
cnt = int(loadset)
for row in csvreader:
   cnt = cnt + 1
   uidNumber  = ay + semester + str(cnt)
   uid = row[2].strip()
   givenName = row[0].strip()
   sn = row[1].strip()
   passwd = row[3].strip()
   # print(f'username {uid} - firstname  {givenName}')
   dn = "uid=" + uid + ",ou=People,o=juniata.edu,dc=juniata,dc=edu"
   homedir = "/home/" + uid
   cn = givenName + " " + sn
#  Section used to screen print the LDIF file
   ldq = f'dn: uid={uid},ou=People,o=juniata.edu,dc=juniata,dc=edu' + "\n"
   ldq = ldq + f'uid: {uid}' + "\n"
   ldq = ldq + f'objectClass: person' + "\n"
   ldq = ldq + f'objectClass: organizationalPerson' + "\n"
   ldq = ldq + f'objectClass: inetOrgPerson' + "\n"
   ldq = ldq + f'objectClass: posixAccount' + "\n"
   ldq = ldq + f'objectClass: shadowAccount' + "\n"
   ldq = ldq + f'sn: {sn}' + "\n"
   ldq = ldq + f'givenName: {givenName}' + "\n"
   ldq = ldq + f'cn: {givenName} {sn}' + "\n"
   ldq = ldq + f'userPassword: {passwd}' + "\n"
   ldq = ldq + f'uidNumber: '  + uidNumber + "\n"
   ldq = ldq + f'gidNumber: 1700' + "\n"
   ldq = ldq + f'loginShell: /bin/bash' + "\n"
   ldq = ldq + f'homeDirectory: /home/{uid}' + "\n"
   ldq = ldq + "\n"
   ldq = ldq + "-" + "\n"
   attrs['objectClass'] =['person'.encode('utf-8'),'organizationalPerson'.encode('utf-8'),'inetOrgPerson'.encode('utf-8'),'posixAccount'.encode('utf-8'),'shadowAccount'.encode('utf-8')]
#   attrs['dn'] = [str(dn).encode('utf-8')]
   attrs['uid'] = [str(uid).encode('utf-8')]
   attrs['sn'] = [str(sn).encode('utf-8')]
   attrs['givenName'] = [str(givenName).encode('utf-8')]
   attrs['cn'] = [str(cn).encode('utf-8')]
   attrs['userPassword'] = [str(passwd).encode('utf-8')]
   attrs['uidNumber'] = [str(uidNumber).encode('utf-8')]
   attrs['gidNumber'] = ['1700'.encode('utf-8')]
   attrs['loginShell'] = ['/bin/bash'.encode('utf-8')]
   attrs['homeDirectory'] = [str(homedir).encode('utf-8')]
   ldif = modlist.addModlist(attrs)

# Enable for LDIF print out
#  print("LDIF....")
#  print(ldq)

# Enable encoded LDIF dump
#  print("Python UTF-8 encodeed LDIF")
#  print(ldif)

   try:
      conn.add_s(dn,ldif)
   except conn.LDAPError as e:
       pass
conn.unbind_s()
csvfs.close()


```
