# Create AD accounts using OpenLdap utilities


### 1. Set Variables:
```
#ldap details
ARG_LDAPURI="ldaps://adserver.us-west1-b.c.lithe-grid-257118.internal:636"
ARG_DOMAIN="HWX.COM"
ARG_BINDDN="CN=hdpadmin,OU=Users,OU=Hadoop,DC=HWX,DC=COM"
ARG_USERPSWD="Hadoop@123"
ARG_USER_BASE="OU=CLD_HDP,DC=HWX,DC=COM"

#User to add
FIRSTNAME="adnan"
LASTNAME="shaikh"
ARG_NewUserPass="Welcome@123"
```

ARG_NewUserPass_unicodePwd

### 2. Create LDIF file ad_user.ldif
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
unicodePwd:: ${ARG_NewUserPass_unicodePwd}

dn: CN=$FIRSTNAME $LASTNAME,${ARG_USER_BASE}
changetype: modify
replace: userAccountControl
userAccountControl: 512
EOFILE
```

__Make sure there are no spaces at the ends of each of these lines or you will get an error like the following:__

```
ldap_add: No such attribute (16)
      additional info: 00000057: LdapErr: DSID-0C090D8A, comment: Error in attribute conversion operation, data 0, v2580
```

