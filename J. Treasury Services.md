# Treasury Services (Command  Group: 3)

Used by the Treasury to create and destroy tokens. Also used to manage multi-chain liqudity.

[Command Codes](#command-codes)

[Command Code Meanings](#command-code-meanings)

[Access Keys](#access-keys)


## Command Codes
Note: The Tresury may manage coins in the West Token Conversion Wallet zero by using the Lock Services. 
Command Code | Service | Description
--- | --- | :---: 
120 | [Get Available SNs](#get-available-sns) | Tells the Treasury what serial numbers it can use to create tokens. 
130 | [Create Tokens](#create-tokens) | Orders that tokens be created
140 | [Delete Tokens](#delete-tokens) | Orders tokens to be destroyed. 
141 | [Freeze Tokens](#freeze-tokens) | Stops money from being transacted until unfrozen.  
142 | [Unfreeze Tokens](#unfreeze-tokens) | Returns the control of the coins to the user.  
143 | [Seize Tokens](#seize-tokens) | Orders tokens to be put into a locker so that the treasure can download it. 
150 | [Release Lock](#release-lock)|  Tells RAIDA to release lock on reserved SNs.
160 | [Get All SNs](#get-all-sns)|  Returns all the serial numbers that the RAIDA has minted.
161 | [Create_Depository](create-depository) | Creates a Depository for an exchange, merchant of bank
161 | [Stop_Depository](stop) | Suspect Depository
? | [🔴Set Fee For Swaps](#set-fee-for-swaps) | The conversion fee set in percentages. 


## Command Code Meanings
Code | Meaning | Sample
---|---|---
RD | Random | 6
DN | Denomination | This denomination will not have a random first four bits. Hex between 0x00 and 0x0F 
SN | Serial Number | 16777215
AN | Authenticity Number | F5DD153926DA42E8A062A915AD763DF0
AU | Authorization Number | 03C7C0ACE395d80182DB07AE2C30F034
CH | Challenge | A random number that they RAIDA must 
NR | Number of SN ranges | 10
NS | Number of single SN | 5
SI | Session ID | FF01AABB
RS | Range Start | A four byte serial number that is the first available SN in a range
RE | Range End | A four byte serial number that is the last available SN in a range
PG | PAN Generator | The Proposed Authenticy Number Generator is a seed the the RAIDA will use to create authenticy numbers. This is explained below.

# Access Keys
Some denominations are used as Access Keys and allow their holders to have rights on the RAIDA.

There are four access key types implemented in the RAIDA Protocol.

Every packet sent to a RAIDA server is encrypted with a key of a certain type. Besides its primary encryption function the key also provides authorization.

A RAIDA server decrypts a packet and identifies the user and permissions granted to this user.

The keys implemented as coins with higher denominations.

The key types are:
🔴 WARNING: Admins should not be able to do everything except use any Treasury Services.

🔴 WARNING: Treasury keys should be able to do everything.

🔴 WARNING: RAIDA Keys should be able to call services that sync if we have that.

🔴 WARNING: KYC uploads that do not have a user connected to them should be deleted after a month. The user can start again if they need to.
 
Decimal Value | Hex Value | Name | Function 
---|---|---|---
7 | 0x7 | User Key | Users are not allowed to use the Wyoming Stable Token unless they have passed KYC and AML requirments. Users who meet the KYC and AML requirments will be issued an Access Key by the Access Key Administrator. Almost all services require this key: pown, pown_sum, detect, detect_sum, store_sum, store, remove_locket, fix, get_ticket. Echo and Version calls do not require any key.
8 | 0x8 | Tresury Keys |These keys are used to mint coins and manage liquidity pools. Whomever posseses these keys can create and destroy coins (get_available_sns, create_coins, delete_coins)
9 | 0x9 | Admin Keys | Admins can create all key types and mint coins. Also admins can query stat services (get_stat, audit). Admins can call ANY service on the RAIDA.
10 | 0xA | RAIDA Keys | Used for inner-RAIDA communication. Only one service (validate_ticket) checks these keys
11 | 0xB | KYC Keys | Anyone can upload KYC documents to the RAIDA filesystem. (create_folder, delete_folder, upload_file, delete_file)
12 | 0xC | KYC Admin Keys | These keys are used to read ALL uploaded KYC documents


# Get Available SNs
This method is the first part of the two-step token creation process. The client sends four-byte SessionID and the RAIDA server locks serial numbers
using the Session ID. After that, the next step (Create Tokens) downloads the tokens using the same Session ID.

This service returns an array of token serial numbers that are available to be used in the creation command. The returned item can be either an individual serial number or a range.

The ranges come first prepended by the number of them. The individual serial numbers come after them.


Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
SI SI SI SI
DN DN DN DN DN DN DN DN DN DN DN DN DN DN DN DN DN DN DN DN //One byte for each denomination. Request for 0-255 tokens of that denomination. 
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Failure | 251


Sample Returns for three denominations. The first has three ranges and two singles. The second has 1 range and 1 single. The third has 2 ranges and 3 singles. 
```
DN    //First denomination code (See denomination codes in the Header format)
NR NS  // NR (Number of ranges 0 to 255). NS (Number of single serial numbers 0 to 255). 
RS RS RS RS  RE RE RE RE // RS (starting serial number in range) RE (last serial number in range, inclusive)
RS RS RS RS  RE RE RE RE
RS RS RS RS  RE RE RE RE  
SN SN SN SN // first single serial number
SN SN SN SN
DN     //Second denomination
NR NS 
RS RS RS RS  RE RE RE RE 
SN SN SN SN
DN  // Third denomination
NR NS 
RS RS RS RS  RE RE RE RE
RS RS RS RS  RE RE RE RE  
SN SN SN SN
SN SN SN SN
SN SN SN SN
3E 3E  //Not Encrypted
```

# Release Lock 
🔴 WARNING: This service has been implmented but is being considerd for deletion.

The service deletes coins that were previously reserved by Get Available SNs call.  
Example Request Body with four coins:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
SI SI SI SI
DN SN SN SN SN
DN SN SN SN SN
DN SN SN SN SN
DN SN SN SN SN
3E 3E
```


Response Status | Code
---|---
Success | 250


# Get All SNs
The service returns all serial number for a denomination (or all denominations).
The returned byte array is packed into a bitmap sequence where "1" stands for "minted", "0" is "available"


Example Request Body: 
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  
3E 3E  //Not Encrypted
```


Response Status | Code
---|---
Success | 250


```
11110000 111111000
```
sn 0 (lowest bit of the first byte) available
sn 1 available
sn 2 available
sn 3 available
sn 4 minted
sn 5 minted
sn 6 minted
sn 7 minted
sn 8 (lowest bit of the second byte) available
sn 9 available
sn 10 available
sn 11 minted
sn 12 minted
sn 13 minted
sn 14 minted
sn 15 minted

Response Body 
```
DN
NR NR NR NR
SB SB SB SB SB ...
...
E3 E3
```
DN - denomination (0x99 for all of them)
NR - number of returned bytes for a denomination
SB - serial number byte (one bit per serial number)



# Create tokens

The service creates tokens that were previously reserved by "Get Available SNs" call. Create tokens must be called within a few seconds after
the initial Get Available SNs call before the session expires. The same Session ID must be used for both services.

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
SI SI SI SI
PG PG PG PG PG PG PG PG PG PG PG PG PG PG PG PG // see instructions below
DN SN SN SN SN
DN SN SN SN SN
DN SN SN SN SN
DN SN SN SN SN
3E 3E //Not Encrypted
```


Response Status | Code
---|---
Success | 250


Response Body Will be empty except for status. 
```
E3 E3 //Not Encrypted
```


The PAN is determined by concatinating the RAIDA ID, Serial Number and the PG. Like this

```diff
🔴 WARNING: Client should send a different PG to each RAIDA otherwise the coin will be stolen by any other RAIDA admin. 
```

1. The Client will generate a 16 byte random seed.
2. The Client will send this seed to the RAIDA. This seed is called a "PAN Generator".
3. The Client and the RAIDA will use an algorithm to figure out what the Authenticity numbers will be for each serial number that fixes. 
4. The Autheticty numbers for each serial number will be calculated by concatenating the RAIDA Number (String), the Coin's Serial Number (string) and the Seed that will be (converted into a 32 character string) together. 

```
1 + 1632435 + 9195C7A60CF0471BBE9B66144F5681C5 and then running an MD5 hash agains that.
```
Then the concatenated string is: 
```
PAN = MD5( "116324359195C7A60CF0471BBE9B66144F5681C5"); 
```
So the AN will be:
```
fd1495be732ab2f6959559f3041c72c0
```
# Create Tokens In Market Locker
This is exactly like the Create Tokens command but with a different commmand number. 

The PG is not needed and will be ignored. Random numbers should be put there instead. 

The difference on the RAIDA is that these coins will be put in a system locker with a fixed Key ID. The key ID is: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
No locker service can call this special locker key and this cannot be used by the public as their locker. 

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
SI SI SI SI
RD RD RD RD RD RD RD RD RD RD RD RD RD RD RD RD // Random Numbers
DN SN SN SN SN
DN SN SN SN SN
DN SN SN SN SN
DN SN SN SN SN
3E 3E //Not Encrypted
```

Response Status | Code
---|---
Success | 250
Coins Not Available | ??

Response Body Will be empty except for status. 
```
E3 E3 //Not Encrypted
```


# Delete tokens
The sender sends authentic tokens. Those tokens are then deleted from the system and the Months From Start Byte is changed to zero. 

Example Request Body with four tokens: 
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN 
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN 
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN 
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN 
3E 3E  //Not Encrypted
```

Response Status | Code
---|---
All Pass | 241
All Fail | 242
Mixed | 243

Response Body 
```
E3 E3 //Not Encrypted
```

# Freeze Tokens
This stops the token from being able to be authenticated. 

```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN //coins to freeze
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
FK FK FK FK FK FK FK FK FK FK FK FK FK FK FK FK   //Freeze key (guid). This same key is needed to unfreeze and is added to the AN
3E 3E  //Not Encrypted
```
Response Status | Code
---|---
All Pass | Coins found and unfrozen
All Fail | None Unfrozen
Mixed | Mixed Lockers 

MT MT MT MT MS  //The mixed bit field. zeros for pass, ones for failed. 

# Unreeze Tokens
Makes it so that the coins can be used again.  
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN //coins to freeze
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
FK FK FK FK FK FK FK FK FK FK FK FK FK FK FK FK   //Freeze key (guid). This same key is needed to freeze and is subtracted from the existing AN
3E 3E  //Not Encrypted
```
Response Status | Code
---|---
All Coins found and unfrozen | 241
No Coins found and unfrozen | None Unfrozen
Mixed | Mixed Lockers 

//The mixed bit field. zeros for pass, ones for failed. 

# Seize Tokens
This will take the tokens specified and place them into a locker that the Treasure can then put in his bank. 
This service is almost identical to the PUT locker code service except there is no authentication of tokens. 

Sample Seize Request
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN 
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  //This is the locker code. The last four bytes must be set to binary ones (0xFF) 
3E 3E  //Not Encrypted
```
### Response Status Table

Status| Response Body Contents
---|---
All Lockers Passed | Will not include a response body | Will not include a response body | 
All Lockers Failed | Will not include a response body
Mixed Lockers | MT MT MT MT MS  //The MT are just zeros. 0x00 00 00 00. They are for future use. The number of MS bytes depends on the number of tokens p'owned. 

MS means Mixed Status but here we are talking about a mixed group of lockers, not coins. Each bit returned represents the status of one locker. If the bit is a zero then that locker has failed. If the bit is a 1 then that locker is authentic. 

Response Status | Code
---|---
All Pass | 241
All Fail | 242
Mixed | 243


# Set Fee For Swaps
Everytime someone converst coins, a fee will be charged to the account of the West. This
fee is set by the Treasure. In the future, the fee may be either fixed or variable and may have a complicated algorithm. Each currency may have a different fee associated with it. That is way there will be bytes reserved in the request body. For now this command just sets a percentage of the coins conversted. 

Suppose the conversion fee is set to 3%. If 100 West are converted to Bitcoin, the fee will be 3 West. Suppose a person wants to convert 1 Bitcoin to 100 West with the same 3% fee. The fee will be 3 West. The amount of fees is always calculated based on the value of the West tokens involved.  

The request body needs to have a percentage to charge as a fee. This percentage is expressed in two bytes. The first byte is the whole part of the percentage and ranges from 0 to 99. The second byte is the fraction part of the percentage and is valued from 0.99 too. So the least percentage that can be charged is 0.0% and the most is 99.99%

Example Request:
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
$$ $$ // Conversion Fee (percentage)
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS //160 bytes reserved for future expansion. 
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS 
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS
RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS RS 
3E 3E //Not Encrypted
```

Example Response:

Response Status | Code
---|---
Success | 250
Percentage is out of bounds| ?? 
The body is empty| ?? 
```
 //Empty
3E 3E //Not Encrypted
```
