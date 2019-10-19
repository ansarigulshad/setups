# # Configure OpenLDAP Server on RHEL/CentOS


### Step 1: Install LDAP packages on LDAP server
```
# yum -y install openldap-servers openldap-clients
# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG 
# chown ldap. /var/lib/ldap/DB_CONFIG 
```

### Step 2: Start & verify LDAP server
```
# systemctl start slapd 
# systemctl enable slapd


# netstat -antup | grep -i 389
```

### Step 3: Run below command to create an LDAP root password. (Here we have used "hadoop123!")
```
# slappasswd -s hadoop123!
{SSHA}7CF6juAoKseni3iy4FVrI+Lnh5dXuA4/
```
### Step 4: Create chrootpw.ldif file and specify the password generated above for "olcRootPW" section
```
# vi chrootpw.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}7CF6juAoKseni3iy4FVrI+Lnh5dXuA4/
```

```
# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif 
```

### Step 5: Import basic Schemas
```
[root@hortonworks ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif 
[root@hortonworks ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif 
[root@hortonworks ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif 
```

### Step 6:	Setup domain name on LDAP DB

##### 6.1 - Generate directory manager's password
```
# slappasswd -s hadoop123!
{SSHA}zoOyuzjjy3TPLhen6LVWDCcRLER/Nsh4
```
##### 6.2 - Create chdomain ldif file
###### replace the password generated above for "olcRootPW" section
```
# vi chdomain.ldif
++++++++++++++++++++++++++++++++++++++++++++++++
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth"
  read by dn.base="cn=Manager,dc=hortonworks,dc=com" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=hortonworks,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=Manager,dc=hortonworks,dc=com

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}zoOyuzjjy3TPLhen6LVWDCcRLER/Nsh4

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by
  dn="cn=Manager,dc=hortonworks,dc=com" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=hortonworks,dc=com" write by * read
```

##### 6.3 - Send the configuration to the LDAP server
```
# ldapmodify -Y EXTERNAL -H ldapi:/// -f chdomain.ldif 
```

##### 6.4 - Create base domain ldif file
```
# vi basedomain.ldif
```

```
dn: dc=hortonworks,dc=com
objectClass: top
objectClass: dcObject
objectclass: organization
o: hortonworks
dc: hortonworks

dn: cn=Manager,dc=hortonworks,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

dn: ou=People,dc=hortonworks,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Group,dc=hortonworks,dc=com
objectClass: organizationalUnit
ou: Group

dn: ou=Users,dc=hortonworks,dc=com
objectClass: organizationalUnit
ou: Users

dn: ou=Hadoop,dc=hortonworks,dc=com
objectClass: organizationalUnit
ou: Hadoop
description: organizationalUnit for BigData & Hadoop
```
##### 6.5 - Send the configuration to the LDAP server
```
# ldapadd -x -D cn=Manager,dc=hortonworks,dc=com -w 'hadoop123!' -f basedomain.ldif
```


### Step 7: 
```
# ldapsearch -x -H ldap://localhost:389 -D 'cn=Manager,dc=hortonworks,dc=com' -w 'hadoop123!' -b 'dc=hortonworks,dc=com'
```

------------------------------------------------------------------------------------------------------------------------------

## Some Usefull links

[Add users and groups trough CLI](https://github.com/dabsterindia/LABs/blob/master/Active%20Directory/openLdap%20-%20Commands.md "Most Useful commands in openLdap")


[Install and Setup phpLDAPAdmin](https://github.com/dabsterindia/LABs/blob/master/Active%20Directory/Install%20phpLDAPadmin.md "")

[Setup SSL/TLS security for OpenLDAP Server](https://github.com/dabsterindia/LABs/blob/master/Active%20Directory/OpenLDAP%20over%20SSL.md "")
