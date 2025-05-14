# RAIDA Filesystem Service (Command Group 10)

These services allow the RAIDA to create and modify folders and data objects. 
It is up to the client to stripe/mirror and sum objects.


Command Code | Service 
--- | --- 
101 | [Create Folder](#create-folder)
102 | [Show Folder Contents](#show-folder-contents)
103 | [Remove Folder](#remove-folder)
104 | [Put Object](#put-object)
105 | [Get Object](#get-object)
106 | [Remove Object](#remove-object)
107 | [Show Any Folder Contents](#show-any-folder-contents)
108 | [Get Any Object](#get-any-object)
109 | [Send Object](#send-object)
110 | [Set ACL](#set-acl)
111 | [Get ACL](#get-acl)
112 | [Show Versions](#show-versions)
112 | [Get Version](#get-versions)
112 | [Set Version ](#set-version)

# Codes: 
Code | Description
---|---
PT | is no more than 255 bytes. The full path of the folder The path must be NULL-terminated. If the path starts with $ then the folder is the public folder and can be accessed by everyone. 
PL | chunk palyoad
NO | Number of objects
TY | Object Type. 1 - Folder, 2 - Data
SZ | Object size in bytes (just the slice for this RAIDAX)
FZ | Object name length
ON | Object name
KY | is KYC Key ID
FN | Friendly Name This is used to address people like "sean@worthington.net"
PC | Protocol Code. These are the names of folders on the RAIDA for the user's emails, instant messages and others. 


# Create Folder
The Create Folder service creates a folder on the RAIDA filesystem.
The Root Path for the folder is located in the user's directory Folders/<UserID>. The user ID equals to the ID of the KYC key.

The maximum path length is 255 charaters.

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT // Path to target 
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Error Already Exists | 201


# Remove Folder
Recursively deletes a folder and all of its contents.
Requires KYC permission.

```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT //path to target
3E 3E //Not Encrypted
```


Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Does not exist | 202



# Show Folder Contents
Displays content of a user folder.
An object in the folder can be either another folder or a data file.
Requires KYC permission.


```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT //Path to taget
3E 3E //Not Encrypted
```

Response

```hex
NO NO
TY SZ SZ SZ SZ FZ ON ON ON ON ON ON ON ON ON ON
TY SZ SZ SZ SZ FZ ON ON ON ON ON ON ON ON ON ON
TY SZ SZ SZ SZ FZ ON ON ON ON ON ON ON ON ON ON
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Does not exist | 202


# Show Any Folder Contents
Displays content of any folder of any user.
The caller must specify the userid (KYC key number)
Requires admin permission

```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
KY KY KY KY
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT //Path to folder
3E 3E //Not Encrypted
```


Response

```hex
NO NO
TY SZ SZ SZ SZ FZ ON ON ON ON ON ON ON ON ON ON
TY SZ SZ SZ SZ FZ ON ON ON ON ON ON ON ON ON ON
TY SZ SZ SZ SZ FZ ON ON ON ON ON ON ON ON ON ON
3E 3E //Not Encrypted
```


Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Does not exist | 202



# Put Object
The service stores a binary chunk of any size below 4G in a RAIDA folder.
The format of the chunk is up to the Client software.

The Client needs to provdide the chunk size

The maximum path length is 255 charaters.

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
SZ SZ SZ SZ
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT //Path to folder
PL PL PL ... PL
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Error Already Exists | 201


# Get Object
The service retreives a binary chunk from the RAIDA

The maximum path length is 255 charaters.

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT
3E 3E //Not Encrypted
```

PT - is no more than 255 bytes. The full path of the folder
The path must be NULL-terminated

Response

```hex
PL PL PL ... PL
3E 3E //Not Encrypted
```

PL - chunk payload

Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Does not Exist | 202



# Get Any Object
The service retreives a binary chunk from the RAIDA. The chunk can belong to any user.

The maximum path length is 255 charaters.

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
KY KY KY KY
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT
3E 3E //Not Encrypted
```

PT - is no more than 255 bytes. The full path of the folder
The path must be NULL-terminated

KY - is KYC user key

Response

```hex
PL PL PL ... PL
3E 3E //Not Encrypted
```

PL - chunk payload

Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Does not Exist | 202


# Remove Object
The service remove a binary chunk from the RAIDA

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT //file path
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Filesystem Error | 200
Does not Exist | 202


# Send Object
The service allows users to put files into the folders of other users. But, 
there are only a few folders the user can put files into. The folders
that users can put files into are specified by the PC (Protocol Code). There will be folders like the .MIME that allow people to send emails to each other. A binary chunk of any size below 4G in another user's RAIDA folder.

The Client needs to provdide the chunk size

The maximum path length is 255 charaters and must we use path codes that are four letters. 

PC (Protocol Codes)
Code | Description
---|---
MIME | Email files with a .eml extension
MQTT | Message Queuing Telemetry Transport
AMQP | Advanced Message Queuing Protocol
DDS0 |Data Distribution Service
XMPP | Extensible Messaging and Presence Protocol
IRC0 | Internet Relay Chat

Note, that each RAIDA must be able to do a DNS lookup to find the SN of the user. Each user will need to have
an 'A' record in their domain. So Sean@RaidaTech.com would be sean 0.0.78.230. The DN (Denomination Number) can be assumed to be a user account, admin accout, etc. 


Example Request to send and email\message or file attachment to another person. 
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
SZ SZ SZ SZ
PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT PT .. PT PT PT //For future use. Can be left blank
PL PL PL ... PL //Payload. the file. Generally, emails should be txt or email file.
PC PC PC PC // Protocol Code. Four bytes fixed readable text. Such as "MAIL", "MQTT"
FN FN FN FN FN FN .... FN ( 255 characters fixed ). This is the friendly name such as sean@raidatech.com 
🔴 DN SN SN SN SN // This is optional but should be used if it is here. 
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Undeliverable, Specified User did not exist | ?
Filesystem Error | 200
Error Already Exists | 201


