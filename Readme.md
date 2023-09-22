# Cloud exploitation

## Table of contents

##### ➤ AZURE CSP

Public enumeration

* [1. Enumerate storage account](#enumerate-storage-account)


##### ➤ AWS CSP



``` ```
``` ```
## Enumerate storage account

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

#####  ➤  Read container having "container/public" public access level

Add "/?restype=container&comp=list"

```
https://kiosecstorage.blob.core.windows.net/public/?restype=container&comp=list
```
