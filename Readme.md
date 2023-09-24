# Cloud exploitation

## Table of contents

##### ➤ AZURE CSP

Public enumeration

* [1. Enumerate storage account](#enumerate-storage-account)


Authenticated enumeration

* [1. Gathering an inventory of ressources (from Azure portal)](#gathering-an-inventory-of-ressources-from-Azure-portal)
* [2. Gathering information using PowerZure](#gathering-information-using-powerzure)
* [3. Enumerating subscription information with MicroBurst](#enumerating-subscription-information-with-microburst)


Password attacks

* [1. Guessing Azure AD credentials using MSOLSpray](#Guessing-Azure-AD-credentials-using-MSOLSpray)
* [2. Identify conditional access policy bypass using MFASweep](#Identify-conditional-access-policy-bypass-using-MFASweep)


Exploitation

* [1. Exploiting reader permissions](#exploiting-reader-permissions)
* [2. Exploiting contributor permissions on IaaS services](#exploiting-contributor-permissions-on-iaas-services)
* [3. Exploiting contributor permissions on PaaS services](#exploiting-contributor-permissions-on-paas-services)
* [4. Exploiting owner and privileged Azure AD role permissions](#exploiting-owner-and-privileged-Azure-AD-role-permissions)


##### ➤ AWS CSP



#
# ⭕ Azure CSP - Public enumeration


## 🔻Enumerate storage account

A storage account is way of hadnling data in the cloud and are compose to the following sub-services :

| sub-services | Description | DNS records |
| :---: | --- | --- |
| Blob service | General HTTP/HTTPS file hosting | xxx.blob.core.windows.net |
| File service | Attached data storage (SMB/NFS) | xxx.file.core.windows.net |
| Table service | NoSQL semi-strctured datasets | xxx.table.core.windows.net |
| Queue service | HTTP/HTTPS message storage | xxx.queue.core.windows.net |


##### Blob storage

The blob storage is a containers/folders that can be used to store data/object and can be accssed through HTTP/HTTPS endpoint. The URL address is a combination of the storage account name, the conatiner and the object name
```
#Storage account name : kiosecstorage
#Container name : public
#File : readme.txt

https://kiosecstorage.blob;core.windows.net/public/readme.txt
```

The containers can have different public access level configuration :
- **Private** : Does not allow anonymous access. Access needs to be authenticated.
- **Blob** : Allow anonymous access to a know object (file into the container). In this case, an attacker needs to know the full filenames (IDOR attack requiered or fuzzing)
- **Container/Public** : Allows the container file to be listed and accessed anonymously.

***Important note :*** The storage account have also a public access level. If the container public access is public but the storage account have a non-public access, the container cannot be accessed anonymously.

#####  ➤  Detect file in container having "blob" public access level

```
gobuster dir -u https://kiosecstorage.blob.core.windows.net/public/ -w /usr/share/wordlists/dirb/big.txt -t 30 -e -k -x .html,.php,.asp,.aspx,.htm,.xml,.json,.jsp,.pl,.txt,.docs,.docx,.yaml
```

#####  ➤  Identify misconfigured blob containers using MicroBurst (Powershell)

```
# Import module
Import-Module .\MicroBurst.psm1

# Detect storage misconfigured with the specific storage account name kiosecstorage
Invoke-EnumerateAzure Blobs -Base kiosecstorage

# Detect storage misconfigured with custom container list (customcontainer.txt) and storage account name (kiosecstorage)
Invoke-EnumerateAzure Blobs -Base kiosecstorage -Folders .\customcontainer.txt

# Download the contents of the objects detected (ex: credentials.txt)
Invoke-WebRequest -Uri "https://kiosecstorage.blob.core.windows.net/public/credentials.txt" --OutFile "credentials.txt"
```

#####  ➤  Read container having "container/public" public access level

Add "/?restype=container&comp=list"

```
https://kiosecstorage.blob.core.windows.net/public/?restype=container&comp=list
```



#
# ⭕ Azure CSP - Authenticated enumeration


## 🔻Gathering an inventory of ressources (from Azure portal)

#####  ➤ From the Azure portal

Navigator **all ressources**, then click to Export to CSV.

***Important note :*** The extract is limited to the 5K first ressources.

![image](https://github.com/Kiosec/Cloud-security/assets/100965892/fd83ecd8-16f4-41fa-8a2e-6f2d8633c58c)


## 🔻 Gathering information using PowerZure

```
#Download PowerZure in an Az Powershell module-authenticated powershell session
PS C:\> cd C:\Users\$env:USERNAME
PS C:\> git clone https://github.com/hausec/PowerZure.git

#Import the Powerzure module into the powershell session
PS C:\> cd .\PowerZure\
PS C:\> Import-Module .\PowerZure.ps1

#If you need to installed the Azure AD Module, answer yes then reopen a new powershell session
PS C:\> cd C:\Users\$env:USERNAME\PowerZure
PS C:\> Import-Module .\PowerZure.ps1

#Execute a command like : determining the actual access that a credential has and its level of access (read/write/execute).
PS C:\> Get-AzureTargets
```

The list of all PowerZure availiable functions can be listed with **'PowerZure -h'** command

![image](https://github.com/Kiosec/Cloud-security/assets/100965892/87f382f2-4603-4bfa-b003-57cbb5421d4a)

***Important note :*** The noting that there are default detections in the Azure Defender-enabled verion of Azure Security Center for PowerZure.


## 🔻Enumerating subscription information with MicroBurst

```
#Import MicroBurst module in an Az Powershell module-authenticated powershell session
PS C:\> cd C:\Users\$env:USERNAME\MicroBurst
PS C:\> Import-Module .\MicroBurst.psm1

#Create a folder to store the output of the function
PS C:\> New-Item -Name "microburst-output" -ItemType "directory"

#Run the MicroBurst function to enumerate the Azure subscription
PS C:\> Get-AzDomainInfo -Verbose -Folder microburstoutput
```

![image](https://github.com/Kiosec/Cloud-security/assets/100965892/4e792bef-81d0-4db0-9b4b-381acf68e3f1)

***Important note :*** The noting that there are default detections in the Azure Defender-enabled verion of Azure Security Center for MicroBurst.



#
# ⭕ Azure CSP - Password attacks


## 🔻Guessing Azure AD credentials using MSOLSpray

MSOLSpray takes a list of user and account a list of password and try them against the Azure authentication portal. 

##### Install MSOLSpray
```
git clone https://github.com/dafthack/MSOLSpray.git
```

##### Execute MSOLSpray
```
cd .\MSOLSpray\
Import-Module .\MSOLSpray.ps1

Invoke-MSOLSray -UserList .\userlist.txt -Password myweakpassword123 
```


## 🔻Identify conditional access policy bypass using MFASweep

MFASweep allows to detect the conditional access policy bypasses against a compromised user accounts.

##### Install MFASweep
```
git clone https://github.com/dafhazck/MFASweep.git
```

##### Execute MFASweep
```
cd .\MFASweep\
Import-Module .\MFASweep.ps1

Invoke-MFASweep -Username kiosec@myazurelab.com -Password myweakpassword123
```



#
# ⭕ Azure CSP - Exploitation


## 🔻Exploiting reader permissions

Review of common areas within Azure that are available to the Reader role where cleartext passwords can be stored. These may be intentional cleartext passwords, but for the most part, these data stores will contain credentials that are accidentally exposed.

***Important note :*** One important thing to note is that some credentials are meant to be in cleartext. There are specific services in Azure where cleartext passwords are expected and utilized as part of the service. This may seem like a dangerous practice, and it is certainly something that we will make use of as an attacker, but with proper authorization controls around the credentials, they can be safely used by some services.



## 🔻Exploiting contributor permissions on IaaS services


## 🔻Exploiting contributor permissions on PaaS services


## 🔻Exploiting owner and privileged Azure AD role permissions
