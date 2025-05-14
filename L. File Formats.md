# Binary Files
WEST Tokens are designed to be "files within files" so they can be inserted into any file format. This allows us to embed WEST into PNG, JPG, MP3, DOCX, TXT. PDF and and just about 
any other format. However, the native WEST file format is binary and has a ".bin" file extension. 

Binary files with the ".bin" extention may hold many tokens.
Each file has one file header. Each token inside the file has one token header. 

There are four standard file formats of the .bin files but many more can be added later if needed: 

## File Format Overview

Format Code | Bytes per token | Purpose | Description
---|---|---|---
8 | 807 | Debugging | Software will use this format if they have made a pown request but there was no response from the RAIDA. In this case it is unkonw if the AN or the PAN is in the RAIDA. Files using this format can be created and put in the "Limbo" folder until they are fixed using the "Find" service.  
9 | 416 | Most Secure | This format allows for twentyfive 16-byte Authenticity Numbers that are not related to each other. 
A | 21 | Compressed | This format allows for each token to have a unique seed that is used to generate all the authenticity numbers for that token. 
B | 7 | Ultra Compressed | This format allows all the tokens in the file to use the same seed.

## File Nameing Convention For Single tokens Inside Software:
We are accurate to one satoshi which represents a decimal, seven zeros, and a 1.

File Name Part | Description
---|---
Whole Number | The number infront of the decimal formated with commas between 3 numbers
Fraction | The numbers after the decimal point serperated by underscores. 
Token Symbol | WST (Wyoming Stable token
P'Own String (optional) | 25 (one per RAIDA) letters showing the results of the last pown. p=passed, f=failed, u=untried, e=error, n=no response, k=encrypton error
Serial Number | The serial number of the token in the file
Memo (optional) | Can be used for any purpose a software would like or left empty. Commonly created by the user.
price (optional) | How much in dollars the token should be sold for if for sale. 
Token Group (option)| Allows software to group tokens such as "ForSale", "ForGroceries", "Savings" "invoice_98473233".
Extension | Always ".bin"

Although some items are optional, the dots are not. 

Samples:
```c
0.00_000_001.WEST.ppppfpppuppppppppppupppep.488773920.From Bill.20..bin
0.00_100_000.WEST.ppppupppeppfpppuppppppppp.1122320.GAI.30.ForSale.bin
0.00_000_001.WEST..223920....bin
1.00_000_000.WEST.ppppupppeppfpppuppppppppp.94887720.Bonus.50..bin
1000.00_000_000.WEST.ppppupppeppfpppuppppppppp.3920.extra.1000.All people.bin

```

## File Nameing Convention For Files with Multiple tokens:
If there are more than 1 token in the file, the sum of all the tokens will be in the name. The extension maybe .bin or .png. 

File Name Part | Description
---|---
Whole Number | The number infront of the decimal formated with commas between 3 numbers
Fraction | The numbers after the decimal point serperated by underscores. 
Token Symbol | WST (Wyoming Stable token
Memo (optional) | Can be used for any purpose a software would like or left empty. Commonly created by the user.
Extension | Always ".bin"

Although some items are optional, the dots are not. 

Samples:
```c
0.08_362_541.For Billy.bin
0.50_167_000..bin
0.00_400_099.Taylor.bin
1.00_000_340.transport.bin
11268.69_894_485.CUBBY.bin
2556.58_000_000.For PGR.bin

```


## File Header For All tokens (32 Bytes Fixed)
```
FT CL ID ID SP EN CC HS HS HS HS HS HS HS HS FL
RC RC RC RC RC RC RC RC RC RC RC RC RC RC RC RC 
```
## File Header Codes:
Code | Bytes |  Name | Description
---|---|---|---
FT | 1 |  Format Type | '8', '9', 'A' or 'B'.
CL |  1 | Cloud ID | 1 for theRAIDAX network
ID |  2 |token ID 1 | ID tokens are 0, Cloudtoken is 1, NFTs are 2  10 =  BTC
RE |  1 |Reserved| Reserved
EN |  1 |Encryption Type | 0 for no encryption
CC |  2 |token Count | How many note are in the file. Not the tota value but total count. 
HS |  7 | MD5 Hash of encryption password | First 7 bytes of a hash of the encryption key used to encrypt the token seeds. (if used). Otherwise zeros if no encyption. This is used to tell if the password that the user used was correct. 
FL |  1 |Flags | Represents 8 bitfields. 00000000 means no flags. 00000001 means encoded with email recovery. Custom for applications. 
RC |  16 | Receipt Number | A transaction's GUID to help software track transactions internally. Can be all random or zeros. But can be used by applications for anything. However, if there is no Receipt number, then the following hex should be used "4C 69 76 65 20 46 72 65 65 20 4F 72 20 44 69 65". This transalates to "Live Free or Die" in ASCII.

## DENOMINATIONS

Except for Idtoken and possible future extensions of this table decimal values should be treated as powers of ten

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
`0x07` | 7 | `0` (fake money used to strengthen encryption)



# FORMAT '8'
This requires 807 bytes per token. Format 8 should be readable by all software but is not the perfered file format. Format 'A' should be used instead. 
This format is for internal use only. It is used for tokens that are stored in the Limbo folder

Split ID (1 byte) | Shard ID (1 byte) | Denomination (1 Byte) | Serial Number (4 Bytes) | Authenticity Numbers (400 Bytes) | Proposed ANs (400 Bytes)
---|---|---|---|---|---
SP | SH | DN | SN SN SN SN  |  16 byte AN x 25 | 16 byte PAN x 25


## token Body Codes for Format 8
16 byte AN x 25 (The original ANs before the call)
16 byte PAN x 25 (The PANs that were sent to the RAIDA but were not confirmed)

Sample File with 2 token in it
```c
SP SH DN SN SN SN SN 
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN   
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN

SP SH DN SN SN SN SN 
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN   
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  

```



# FORMAT '9'
This requires 407 bytes per token. Format 9 should be readable by all software but is not the perfered file format. Format 'A' should be used instead. 

Split ID (1 byte) | Shard ID (1 byte) | Denomination (1 Byte) | Serial Number (4 Bytes) | Authenticity Numbers (400 Bytes) | Proposed ANs (400 Bytes)
---|---|---|---|---|---
SP | SH | DN | SN SN SN SN  |  16 byte AN x 25 | 16 byte PAN x 25


## token Body Codes for Format 9
16 byte AN x 25 (The original ANs before the call)

Sample token in format 9
```c
SP SH DN SN SN SN SN 
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN   
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  

```

# Human Readable Fileformat for Internal Use

```ini

[General]
Status = Authentic
Receipt_ID = c3f294064fb9c2eeb37df8cde81f3cf9
Serial_Number = 6765324
Denomination = .1
coinID = 0
coinName = CloudCoin 
Stripe = n/a
Shard = n/a
Date_Created = January 15th, 2024
[IP:Port]
RAIDA 00 = 34.234.92.98:4593
RAIDA 01 = 34.234.92.98:4593
RAIDA 02 = 34.234.92.98:4593
RAIDA 03 = 34.234.92.98:4593
RAIDA 04 = 34.234.92.98:4593
RAIDA 05 = 34.234.92.98:4593
RAIDA 06 = 34.234.92.98:4593
RAIDA 07 = 34.234.92.98:4593
RAIDA 08 = 34.234.92.98:4593
RAIDA 09 = 34.234.92.98:4593
RAIDA 10 = 34.234.92.98:4593
RAIDA 11 = 34.234.92.98:4593
RAIDA 12 = 34.234.92.98:4593
RAIDA 13 = 34.234.92.98:4593
RAIDA 14 = 34.234.92.98:4593
RAIDA 15 = 34.234.92.98:4593
RAIDA 16 = 34.234.92.98:4593
RAIDA 17 = 34.234.92.98:4593
RAIDA 18 = 34.234.92.98:4593
RAIDA 19 = 34.234.92.98:4593
RAIDA 20 = 34.234.92.98:4593
RAIDA 21 = 34.234.92.98:4593
RAIDA 22 = 34.234.92.98:4593
RAIDA 23 = 34.234.92.98:4593
RAIDA 24 = 34.234.92.98:4593
[Last_Detection_Status]
RAIDA 00 = Passed 
RAIDA 01 = Failed
RAIDA 02 = Untried
RAIDA 03 = Error Returned From RAIDA: Unauthorized amount
RAIDA 04 = Encryption Failed
RAIDA 05 = Passed
RAIDA 06 = Passed
RAIDA 07 = Passed
RAIDA 08 = Passed
RAIDA 09 = Passed
RAIDA 10 = Passed
RAIDA 11 = Passed
RAIDA 12 = Passed
RAIDA 13 = Passed
RAIDA 14 = Passed
RAIDA 15 = Passed
RAIDA 16 = Passed
RAIDA 17 = Passed
RAIDA 18 = Passed
RAIDA 19 = Passed
RAIDA 20 = Passed
RAIDA 21 = Passed
RAIDA 22 = Passed
RAIDA 23 = Passed
RAIDA 24 = Passed
[Starting Authenticity Numbers]
RAIDA 00 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 01 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 02 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 03 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 04 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 05 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 06 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 07 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 08 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 09 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 10 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 11 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 12 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 13 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 14 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 15 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 16 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 17 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 18 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 19 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 20 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 21 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 22 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 23 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 24 = 32cd23416f5f6d7a5d059065897bab05
[Proposed Authenticity Numbers]
RAIDA 00 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 01 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 02 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 03 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 04 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 05 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 06 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 07 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 08 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 09 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 10 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 11 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 12 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 13 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 14 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 15 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 16 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 17 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 18 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 19 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 20 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 21 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 22 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 23 = 32cd23416f5f6d7a5d059065897bab05
RAIDA 24 = 32cd23416f5f6d7a5d059065897bab05
```
<!--
# Text Based Files
These files are designed for people to read them. They are used when the primary concern is the user. They use the .html file format with one data point per line. 

```ini
<!DOCTYPE>
<head>
<title>Wyoming Extensible Stable Token</title>
</head>
<body>
<!-- Format like: “YYYY-MM-DD HH:MM:SS” 
<h2 id="updated">Last Updated = "2022-06-16 23:15:34"</h2>
<h2 id="den">Denomination = Hundred Thousand </h2>
<h2 id="sn">Serial Number = 558554 </h2>
<h3 id ="s01">Secret 01 = </h3> 
 </body>
</html>



```
Names of sixteen denominations: 
Million
Hundred Thousand
Ten Thousand
One Thousand
One Hundred
Ten
one
zero
Dime
Cent



<!--


# FORMAT 'A'
## token Header Codes for Format 'A'
Every token within a file has its own header. For file format 'A', we just have 20 bytes per token because all the ANs are created by one hash seed. 

Shard | Denomination 1 Byte | Serial Number (4 Bytes) 
---|---|---
SD| ST | DN | SN SN SN SN  

## token Body Codes for Format 'A'

 11 Byte Seed (SE ED) | 3 Bytes Reserved (RE)
---|---
 SE ED SE ED SE ED SE ED SE ED SE | RE RE RE


##  Sample token with Header and Body Format 'A'
This has 7 tokens in it. Each token takes up 20 bytes. 
```c
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE//First token
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE//Second token
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE
SD ST DN  SN SN SN SN  SE ED SE ED SE ED SE ED SE ED SE  RE RE RE//7th token
```

To Generate the 25 seeds for each RAIDA:

RAIDA ID | DN |SN | Seed | Concatenated Hex Strings | SHA-384 Hash. Truncated to 16 bytes. 
---|---|---|---|---|---
0x00 | 0xFD | 0x009EF945  | 0xD427AC7B9EF945768E435E | 0x00FD009EF945D427AC7B9EF945768E435E | D9F0CC6D564E8425651BBC6925B0062E
0x01 | 0xFD | 0x009EF945  | 0xD427AC7B9EF945768E435E | 0x01FD009EF945D427AC7B9EF945768E435E | E32D6DC6B7A0A3272E2D8444BF9CDE1C
0x02 | 0xFD | 0x009EF945  | 0xD427AC7B9EF945768E435E | 0x02FD009EF945D427AC7B9EF945768E435E | C6C4D93385C22287C7647F6F2EFA9BE9

1. The software will take all the binary numbers (RAIDA ID, denomination, Serial Number, 11 byte seed ) and turn them into hexidecimal string that uses all uppercase characters. 
2. The resulting number 34 character string will generate a SHA-384 Hash.
3. The hash is trucated to 16 bytes and that becomes the binary AN number. 

-->
# FORMAT 'B'
## token Header Codes for Format 'B'
For this standard, there is one seed for all the tokens. Each token is just the split number, Denomination number and the serial number. This gives us 6 bytes per token. 

After the header, there is an 11 byte seed. THis keeps the same seed size as with format 'A':
 11 Byte Seed (SE ED) 
|---|
| SE ED SE ED SE ED SE ED SE ED SE | 

Then there are six bytes for every token. 

Shard | Denomination 1 Byte | Serial Number (4 Bytes) 
---|---|---
SD | ST | DN | SN SN SN SN  



##  Sample file body in format 'B'
This has 7 tokens in it. Each token takes up 20 bytes. 
```c
CC CC CC //token Count (How many tokens are in the file)
SE ED SE ED SE ED SE ED SE ED SE //Seed
SD ST DN  SN SN SN SN  // First token
SD ST DN  SN SN SN SN  
SD ST DN  SN SN SN SN 
SD ST DN  SN SN SN SN  
SD ST DN  SN SN SN SN 
SD ST DN  SN SN SN SN  6th token
```
<!--
To Generate the 25 seeds for each RAIDA (Same as with 'A'):

RAIDA ID | DN |SN | Seed | Concatenated Hex Strings | SHA-384 Hash. Truncated to 16 bytes. 
---|---|---|---|---|---
0x00 | 0xFD | 0x009EF945  | 0xD427AC7B9EF945768E435E | 0x00FD009EF945D427AC7B9EF945768E435E | D9F0CC6D564E8425651BBC6925B0062E
0x01 | 0xFD | 0x009EF945  | 0xD427AC7B9EF945768E435E | 0x01FD009EF945D427AC7B9EF945768E435E | E32D6DC6B7A0A3272E2D8444BF9CDE1C
0x02 | 0xFD | 0x009EF945  | 0xD427AC7B9EF945768E435E | 0x02FD009EF945D427AC7B9EF945768E435E | C6C4D93385C22287C7647F6F2EFA9BE9

1. The software will take all the binary numbers (RAIDA ID, denomination, Serial Number, 11 byte seed ) and turn them into hexidecimal string that uses all uppercase characters. 
2. The resulting number 34 character string will generate a SHA-384 Hash.
3. The hash is trucated to 16 bytes and that becomes the binary AN number. 

-->
