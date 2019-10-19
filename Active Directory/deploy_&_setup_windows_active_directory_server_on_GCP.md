# Setup Windows Active Directory on Google Cloud


### Step 1: Launch Windows Server Instance on GCP
* Compute Engine
       --> Create Instance
* Deploy Instance with Windows server image

```
eg:
Windows Server 2012 R2 Datacenter
Server with Desktop Experience, x64 built on 20191008
```

#### Step 2: Generate username and password for firt time login

#### Step 3: Login AD server with same username/password we generated in step 2

#### Step 4: Setup password for 'Administrator' account
Administrator's credentials are required in further steps

#### Step 5: Once loged-in, Add following Roles and features
1) DNS Server
2) Active Directory Domain Services (AD DS)
3) Active Directory Certificate Service (AD CS)

##### a) Add `DNS Server` and `AD DS` Roles
* Promote this server to a domain controller

_AD Server will be rebooted automatically after this setup

##### b) Login with 'administrator' credentials

##### c) Add `AD CS` Role
* Configure Active Directory Certificate Service on the destination server

##### d) Reboot AD Server after setting up AD DS successfully

#### Step 6: 
