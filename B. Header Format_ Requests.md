<!-- ![Request Headers](zips/header.png) -->

[Request Headers Overview](#-raida-request-headers)

[Byte by Byte Meaning](#request-header-byte-by-byte-code-meanings)

[COMMAND GROUPS](#command-groups)

[DENOMINATIONS](#denominations)

[ENCRYPTION](#encryption)

# RAIDA Request Headers
This prodocol allows clients to send requests to the  RAIDA. 
All requests made to the RAIDA must have a 32 byte header.
The header is fixed length and will always be 32 bytes. 
If the header does not contain 32 bytes, then the RAIDA will not respond. 
The Request Header is not encrypted. 



Headers are made up of four groups of eight bytes. The byte groups are named "Routing", "Presentation", "Encryption" and "Nounce".
This header is designed to allow  RAIDAs to be extensible and perfome other functions besides token authentication. (extensible)[#extensible].

All multibyte fields are considered big-endian. For example, the two "ID" byte located at 0x04 and 0x05, 0x04 is MSB and 0x05 is LSB

## RAIDA Protocol Header Byte Group View (32 Bytes Fixed):
```javascript
VR SP RI SH CG CM ID ID  // 8 Routing Bytes
VR AP AP CP TR AI RE RE  // 8 Presentation Bytes
EN DN SN SN SN SN BL BL  // 8 Encryption Bytes
NO NO NO NO NO NO EC EC  // 8 Nounce Bytes
```



## RAIDA Protocol Header Offset View:
Offset(h) |`0x00` | `0x01` | `0x02` | `0x03` |`0x04` |`0x05`|`0x06`|`0x07`|`0x08`|`0x09`|`0x0A`|`0x0B`|`0x0C`|`0x0D`|`0x0E`|`0x0F`
-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-
`0x00` | VR |SP |RI |SH| CG | CM |ID| ID| VR | AP| AP |CP | TR| AI| RE| RE 
`0x10` | EN | DN |SN |SN| SN | SN |BL| NO| NO| NO | NO| NO |NO | NO| EC| EC 

![Color Chart of Bytes](zips/request_header_format.png)

### Request Header Byte by Byte Code Meanings

Index | Group | Code | Name | Notes
---|---|---|---|---
00 | Routing | BF | Bitfield  | Describes if the eight Routing fields are just random. The first bit is always random. 
01 | Routing | SP | Split ID | For future use in case the token is to be split. Otherwise zero. 
02 | Routing | RI | RAIDA ID  |  Detection Agents called RAIDA Servers. Value must be 0-24. 
03 | Routing | SH | Shard ID  | For future use in case the token is to be sharded. For now, zero. 
04 | Routing | CG | [Command Group](#command-groups) | High order byte for command number
05 | Routing | CM | [Command](#commands)  | Low Number byte for command number
06 | Routing | ID  | RAIDA ID 0 | Always zero
07 | Routing | ID  | RAIDA ID | Always one
08 | Presentation | BF | Bitfield  | Describes if the eight Routing fields are just random. The first bit is always random. 
09  | Presentation | AP | Application 0| | Applications are generally numbered by the port they use
10 | Presentation | AP  | Application 1 |  0 means generic. 80 = HTTP, 25 =SMTP etc. 
11 | Presentation | CP | Compression | Future Use. 0 means none (Future Use)
12 | Presentation | TR | Translation | Future Use. 0 means none. Can be used to encode text-based protocols. 
13 | Presentation | AI  |AI Translation | Future Use. 0 means none. What type of AI should be used to translate.
14 | Presentation | ii |  i | The packet index number in the message array
15 | Presentation | LE |  AR | The length of packets in the message array.
16 | Encryption | EN  | [Encryption](#encryption) Type  |  0x00 means no encryption. See encryption codes table.
17 | Encryption | DE | [Denomination](#denominations) | Denomination of the token used to encrypt the request body.
18 | Encryption | SN | Encryption token SN 0| Serial Number of the token used to encrypt the body. HOB
19 | Encryption | SN |  Encryption token SN 1 | 2nd Highest Order Byte
20 | Encryption | SN |  Encryption token SN 2 |  
21 | Encryption | SN |  Encryption token SN 3| Lo Order Byte
22 | Encryption | BL u16| Body Length | Length in bytes of the entire body including the last 2 terminating bytes. 
23 | Encryption | BL u16| Body Length| LOB. if more than 65,535 bytes are sent, then bytes 22 and 23 will be FF FF and bytes 24, 25, 26 and 27 will be the body length. 
24 | Nonce | NO |  Nonce 0 / BL u32 | The nonce used in the encryption and should never be used twice. 
25 | Nonce | NO |  Nonce 1 / BL u32| 
26 | Nonce | NO |  Nonce 2 / BL u32|
27 | Nonce | NO |  Nonce 3 / BL u32| 
28 | Nonce | NO |  Nonce 4 | Always a random byte
29 | Nonce | NO |  Nonce 5 | Always a random byte
30 | Nonce | NO |  Nonce 6 / Echo 0 | Serves two purposes. These bytes are always echoed back to the client.
31 | Nonce | NO |  Nonce 7 / Echo 1 | 

* Nounce can do two jobs. Bytes 30, 31 are used as an Echo also.
* If EN (byte 16) is zero (no encryption) then bytes 17-31 may take any values. In 'no encryption' mode, these values will be ignored except for the two echo bytes. 
* The nounce is also a challenge. The RKE (RAIDA Key Exchange) server must decrypt this and place it in the response if RKE is enabled. 


## COMMAND GROUPS FOR APPLICATION CODE 0 (Taken from bytes 9 and 10)
Code | Name | Description
--- | --- | ---
0 | Status | Status, Errors that have nothing to do with commands but usually with headers
1 | Authentication | Services that authenticate tokens.
2 | Healing | Services that make all RAIDA agree on the authenticity of tokens.
3 | Admin | Services that add or subtrack tokens from the  RAIDA.
8 | Locker Services | Services that place and retrieve tokens from "RAIDA Lockers."
9 | Change | Breaks 100s into 10s or joins 10s to make 100s. Works with all denominations.


## COMMAND CODES

Group | Code (Decimal) | Command | Description
---|--- | --- | --- 
 Status |00 |  Echo | Sends a request for a response.
 Status |01 | Version | Returns the version of the RAIDA Software
 Status |02 | ShowStats | Shows information about requests to the RAIDA
 Status |03 | Count Coins | Shows how many coins are in the RAIDA
 Authentication | 10 |  Detect | Compares authenticity numbers but does not change them.
 Authentication | 11 |  Detect SUM | Compares authenticity numbers by adding them all together. Does not change them.
 Authentication | 20 | P'Own | Password Own. Changes ANs (Authenticity Numbers) with PANs (Proposed Authenticity Numbers). 
 Authentication | 21 | P'Own Check Sum | Checks the sums of all ANs sent at once and changes them if good.
Healing |  40 |Get Ticket | Returns proof that the tokens are good. 
Healing |  41 |Get Ticket by Sum | Returns proof that the tokens are good based on a sum of ANs. 
Healing |  44 |Get Encryption Ticket | Returns proof that the tokens are good. 
Healing |  45 |Fix Encryption | Returns proof that the tokens are good based on a sum of ANs. 
 Healing | 50 | Validate Ticket | Checks to see if a ticket is valid. 
 Healing | 60 |Find|  Checks last two ANs to see if there is a match. 
 Healing | 80 | Fix | Accepts a bunch of tickets to ensure they are good.
 Locker | 82 | Put | Puts token in a locker that can be opened by a key.
Locker | 83 | Peek | Just returns information about what is in the locker such as how many tokens are there. 
Locker | 84 | Remove | Removes tokens from locker and destroys locker.
 Change |91 | Get Available SNs | Asks the RAIDA what SNs are available for use. 
 Change |92 | Break | Breaks token into ten smaller tokens. 
 Change |93 | Join | Joins smaller tokens into one larger token. 
 Files | 101 | Create Folder | Creates a folder on the RAIDA
 Files | 102 | Show Folder Contents | Shows list of files and folders
 Files | 103 | Remove Folder | Removes a folder
 Files | 104 | Put Object | Uploads a file
 Files | 105 | Get Object | Downloads a file
 Files | 106 | Remove Object | Deletes a file
 Files | 107 | Show Any Folder Contents | Used by KYC officers to see list of files
 Files | 108 | Get Any Object | Used by KYC officers to see files
 
 Treasury | 120 | Get Available SNs| Shows available serial numbers. 
 Treasury |130 | Create tokens| Adds more tokens to the total tokens on the RAIDA.
 Treasury |140 | Delete tokens| Removes tokens from the RAIDA.
 Treasury |150 | Release Lock| Tells RAIDA to release lock on reserved SNs. 
 Treasury | 151 | Set Fee for Swaps | Allows Treasure to set the fee swap rate 
 Treasury |160 | Get All SNs | Allows RAIDA to syncronize their serial numbers. 
 
 Key Exchange |44 |Encrypt Ticket | Returns an encrypted key part that can be used as a shared secret. 
 Key Exchange |45 | Fix Encryption | Accepts tickets from many RAIDA to created a shared secret from many "key parts". 



## DENOMINATIONS
Except for ID Token and possible future extensions of this table decimal values should be treated as powers of ten.

Hex Code| Decimal Code | Denomination
--- | :---: | ---
`0xF8` | -8 | `.000 000 01` (1 Satoshi)
`0xF9` | -7 | `.000 000 1`
`0xFa` | -6 | `.000 001`
`0xFb` | -5 | `.000 01`
`0xFc` | -4 | `.000 1`
`0xFd` | -3 | `.001`
`0xFe` | -2 | `.01` (1 Cent token)
`0xFf` | -1 | `.1` (1 Dime token)
`0x00` | 0 | `1` (1 Whole token)
`0x01` | 1 | `10`
`0x02` | 2 | `100`
`0x03` | 3 | `1,000`
`0x04` | 4 | `10,000`
`0x05` | 5 | `100,000`
`0x06` | 6 | `1,000,000`
`0x07` | 7 | User Keys
`0x08` | 8 | Treasurer Keys
`0x09` | 9 | Account Admin Keys 
`0x0a` | 10 | RAIDA ID Keys
`0x0b` | 11 | KYC Keys


## ENCRYPTION
Code | Type | Description | Bytes after EN
---|---|-----|---
0 | No encryption | Clear Text | 5 ( All Zeros)
1 | 128 AES CRT | Shared Secret |  5 (1 DN + 4 SN)
2 | 128 AES CRT | Locker Key is used | This is used by the PEEK command if the user doesn not have any coins yet. See PEEK in Locker Services. 
3 | 128 AES CRT | Peer-to-Peer RKE | This is used when two computers who have used RKE to exhange keys are using byte 17 through 21 of the header as an ID for a key instead of a coin. 

### ENCRYPTION RULES All SERVICES

Rule | Description
---|---
 Request Headers | Not encrypted
 Request Bodies | Encrypted using the same encryption key as the Request header specifies. 
 "E3 E3" |  Terminating bytes at end of a request or a response not encrypted.
