# Install and configure SSSD on RHEL/CentOS Systems

One of the best and Recommended solution to pull users from AD/LDAP is to use tools like SSSD/Centrify/NSLCD/Winbind/SAMBA etc.

Follow below steps to setup sssd on Linux to get user info from AD using LDAP provider

#### Step 1: Install required sssd and openldap packages:

```
# yum install sssd sssd-clients -y
```

```
# yum install openldap  openldap-clients
```

#### Step 2: Configured sssd.conf with below config:
```
#vi /etc/sssd/sssd.conf
```

```
[sssd]
config_file_version = 2
reconnection_retries = 3
sbus_timeout = 30
services = nss, pam
domains = CLOUDERA.COM

[domain/CLOUDERA.COM]
description = CLOUDERA.COM
#debug_level = 2
min_id = 1000
id_provider = ldap
auth_provider = ldap
# auth_provider = krb5
ldap_schema = rfc2307bis
cache_credentials = true
enumerate = True   # without this `getent passwd` will show only local users

# krb5_server = adserver.dabsterinc.com
# krb5_realm = CLOUDERA.COM

###Update the AD server and other details ###
 
ldap_search_base = OU=hadoop,DC=CLOUDERA,DC=COM
ldap_uri = ldaps://adserver.cloudera.com
ldap_default_bind_dn = CN=Administrator,CN=Users,DC=CLOUDERA,DC=COM
ldap_default_authtok_type = password
ldap_default_authtok = Hadoop@1

ldap_id_use_start_tls = True
ldap_tls_reqcert = allow
ldap_tls_cacertdir = /etc/openldap/certs
ldap_user_object_class = user
ldap_user_name = sAMAccountName
ldap_user_principal = userPrincipalName
ldap_group_object_class = group
ldap_group_name = cn
ldap_force_upper_case_realm = True
 
##UID/GID Mapping config 
ldap_user_objectsid = objectSid
ldap_group_objectsid = objectSid
ldap_user_primary_group = primaryGroupID
case_sensitive = false
ldap_id_mapping = true
fallback_homedir = /home/%u
default_shell = /bin/bash
 
[nss]
reconnection_retries = 3
#debug_level = 2
 
[pam]
reconnection_retries = 3
#debug_level = 2
```

#### Step 3: Import SSL certs of AD server to the cacerts db under /etc/openldap/certs:
```
# echo | openssl s_client -connect adserver.cloudera.com:636 2>&1 | sed --quiet '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /var/tmp/ad.crt
# cd /etc/openldap/certs
# certutil -A -d . -n "AD cert" -t "C,," -i /var/tmp/ad.crt
```

#### Step 4: Restart SSSD and verify if “id –a shows the ad user info:
```
# chmod 0600 /etc/sssd/sssd.conf

# service sssd start
# id -a <AD-User>
# getent passwd <AD-User>
```

#### Step 5: Once verified, Configure PAM to allow AD users to login:
```
# vi /etc/pam.d/sshd
```
```
#%PAM-1.0
auth       required     pam_sepermit.so
auth       substack     password-auth
auth       include      postlogin
# Used with polkit to reauthorize users in remote sessions
-auth      optional     pam_reauthorize.so prepare
account    required     pam_nologin.so
account    include      password-auth
password   include      password-auth
# pam_selinux.so close should be the first session rule
session required pam_mkhomedir.so skel=/etc/skel/ umask=0022
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    include      password-auth
session    include      postlogin
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
```

```
# yum install authconfig
# authconfig --enablesssdauth --update
```

Login to the host with AD user account and verify the access to node. 
As sssd does caching of users on first start, sssd startup may take time as it has to connect to AD to get the user and cache it.


#### Step 6: Login Linux server with AD user
```
# ssh gulshad@`hostname -f`
```


#### (Optional) : SSSD mostly used commands
##### Start SSSD Service
`systemctl start sssd`

##### Stop SSSD Service
`systemctl start sssd`

##### Check Status
`systemctl status sssd`

##### Clear Cache
`systemctl stop sssd; rm -rf /var/log/sssd/* /var/lib/sss/{db,mc}/*; systemctl start sssd`

