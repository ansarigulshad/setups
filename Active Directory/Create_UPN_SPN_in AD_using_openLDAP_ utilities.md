# Create AD accounts using OpenLdap utilities

## 1. ADD USER ACCOUNT

### Step 1: Set Variables:
```
#ldap details
ARG_LDAPURI="ldaps://adserver.us-west1-b.c.lithe-grid-257118.internal:636"
ARG_DOMAIN="HWX.COM"
ARG_BINDDN="CN=hdpadmin,OU=Users,OU=Hadoop,DC=HWX,DC=COM"
ARG_USERPSWD="Hadoop@123"
ARG_USER_BASE="OU=CLD_HDP,DC=HWX,DC=COM"

#User to add
FIRSTNAME="gulshad"
LASTNAME="ansari"
```

```
LDAPTLS_REQCERT=never ldapsearch -x -H "${ARG_LDAPURI}" -D "${ARG_BINDDN}" -w "${ARG_USERPSWD}" -b "${ARG_USER_BASE}" 
```

### Step 2: Create unicode Password for the above ad user with the password Welcome@123
```
ARG_NewUserPass=`echo -n '"Welcome@123"' | iconv -f UTF8 -t UTF16LE | base64 -w 0`
```

```
echo $ARG_NewUserPass
```

### Step 3: Create LDIF file
```
cat > /tmp/$FIRSTNAME.ldif <<EOFILE
dn: CN=$FIRSTNAME $LASTNAME,${ARG_USER_BASE}
changetype: add
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
distinguishedName: CN=$FIRSTNAME $LASTNAME,${ARG_USER_BASE}
cn: $FIRSTNAME $LASTNAME
sn: $FIRSTNAME$LASTNAME
givenName: $FIRSTNAME
displayName: $FIRSTNAME $LASTNAME
name: $FIRSTNAME $LASTNAME
accountExpires: 0
userAccountControl: 514
sAMAccountName: $FIRSTNAME$LASTNAME
userPrincipalName: $FIRSTNAME$LASTNAME@${ARG_DOMAIN}

dn: CN=$FIRSTNAME $LASTNAME,${ARG_USER_BASE}
changetype: modify
replace: unicodePwd
unicodePwd::${ARG_NewUserPass}

dn: CN=$FIRSTNAME $LASTNAME,${ARG_USER_BASE}
changetype: modify
replace: userAccountControl
userAccountControl: 512
EOFILE
```
```
cat /tmp/$FIRSTNAME.ldif
```
___Make sure there are no spaces at the ends of each of these lines or you will get an error like the following:___

```
ldap_add: No such attribute (16)
      additional info: 00000057: LdapErr: DSID-0C090D8A, comment: Error in attribute conversion operation, data 0, v2580
```

### Step 4: Add user account to AD (user will be added under $ARG_USER_BASE)
```
LDAPTLS_REQCERT=never ldapadd -x -H "${ARG_LDAPURI}" -a -D "${ARG_BINDDN}" -f /tmp/$FIRSTNAME.ldif -w "${ARG_USERPSWD}" 
```

## 2. ADD SERVICE PRINCIPAL AND CREATE KEYTAB

### Step 1: Set Variables:
```
#ldap details
ARG_LDAPURI="ldaps://adserver.us-west1-b.c.lithe-grid-257118.internal:636"
ARG_DOMAIN="HWX.COM"
ARG_BINDDN="CN=hdpadmin,OU=Users,OU=Hadoop,DC=HWX,DC=COM"
ARG_USERPSWD="Hadoop@123"
ARG_USER_BASE="OU=CLD_HDP,DC=HWX,DC=COM"

#User to add
ARG_SPN="HTTP/myhost.hortonworks.com"
```

### Step 2: Create unicode Password for the above ad user with the password Welcome@123
```
ARG_NewUserPass=`echo -n '"Welcome@123"' | iconv -f UTF8 -t UTF16LE | base64 -w 0`
```

```
echo $ARG_NewUserPass
```

### Step 3: Create LDIF file
```
dn: CN=${ARG_SPN},${ARG_USER_BASE}
changetype: add
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
distinguishedName: CN=${ARG_SPN},${ARG_USER_BASE}
cn: ${ARG_SPN}
userAccountControl: 514
accountExpires: 0
userPrincipalName: ${ARG_SPN}@${ARG_DOMAIN}
servicePrincipalName: ${ARG_SPN}

dn: CN=${ARG_SPN},${ARG_USER_BASE}
changetype: modify
replace: unicodePwd
unicodePwd::${ARG_NewUserPass}

dn: CN=${ARG_SPN},${ARG_USER_BASE}
changetype: modify
replace: userAccountControl
userAccountControl: 66048
```

### Step 4: Add user account to AD (user will be added under $ARG_USER_BASE)
```
LDAPTLS_REQCERT=never ldapadd -x -H "${ARG_LDAPURI}" -a -D "${ARG_BINDDN}" -f /tmp/$FIRSTNAME.ldif -w "${ARG_USERPSWD}" 
```

### Step 5: Create Keytab (Login to linux server and run below commands)
```
# ktutil

ktutil:  add_entry -password -p HTTP/myhost.hortonworks.com@HWX.COM -k 1 -e aes128-cts-hmac-sha1-96
ktutil:  add_entry -password -p HTTP/myhost.hortonworks.com@HWX.COM -k 1 -e aes256-cts-hmac-sha1-96
ktutil:  add_entry -password -p HTTP/myhost.hortonworks.com@HWX.COM -k 1 -e arcfour-hmac-md5-exp
ktutil:  add_entry -password -p HTTP/myhost.hortonworks.com@HWX.COM -k 1 -e des3-cbc-sha1
ktutil:  add_entry -password -p HTTP/myhost.hortonworks.com@HWX.COM -k 1 -e des-cbc-md5

ktutil:  write_kt /var/tmp/spenego.service.keytab
ktutil:  exit
```

```
klist -ket /var/tmp/spenego.service.keytab
```
```
kinit -kt /var/tmp/spenego.service.keytab HTTP/myhost.hortonworks.com@HWX.COM
```
