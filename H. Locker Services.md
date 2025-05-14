# Locker Service (Command Group 8)
 Locker services allow people to put many tokens into a RAIDA "Locker" and receive a locker key. This locker key can then be given to another person who then can use the key to download the coins. The locker key is a code specified by the client. 
 
 
## Sugested Client Use
1. The client creates a seed code. The seed code consists of 7 characters. The seven characters are randomly picked from 31 different symbols. The symbols are [ABCDEFGHJKMNPQRSTUVWXYZ23456789] The capital letters of the alphabet, all numbers. However, the letters oh, and el (O,L,I o,l,i) are excluded to reduce confusion. The numbers zero (0) and one (1) are also excluded. Note that the client can create any code but a seven letter code will be standard.

2. User puts the RAIDA number in front of the code and create a hash from it using MD5. These become the locker numbers and each RAIDA has a different numer. Any messages, emails, attachments etc are also sent.

3. The user sends the seven digit code to the money receiver. Along with any other data the receiver needs. 

4. The receiver puts a number infront of each code and creates the locker numbers. 

5. The receiver uses the keys to withdraw the tokens. 

 
 Command Code | Service | Note
 ---|---|---
82| [Put](#put)| Puts coins into a locker. 🔴Needs to support multi locker creation at one time
83| [Peek](#peek) | Allows the locker key holder to see what is in inside of it 
84 | [Remove](#remove) | Allows the locker key holder to emplty out some or all of the contents
? | [Dump](#dump) | Allows the locker key holder to empty locker without calling PEEK

<!--
85 | [Create Depository](create-depository) | Depository is a locker that only the KYC an create
86 | [Log Deposit](#pay) | Puts coins into a depository. Includes transaction information
87| [Confirm Payment](#put-named-locker)| Allows the depository to check if it has received payment for a service
88 | Depository Balance | Tells the caller to see how much WE$T is in a depository
89 | See Transaction Log | 
-->
### Client Actions
Here are the steps that the client should do to send tokens to a locker:
1. Calculate if you need to make change (break tokens into smaller denominations).
2. Call "Get Available Change SNs" if you need change.
3. Call "Break" using the SNs if you need change.
4. Fix any known fracks according to the "pown" string on the file name.
5. Call Store Sum on the tokens that will be sent. 
7. Find Any Lost tokens if there is no response. 
8. Fix any Fracked tokens.
9. Tell the receiver the tokens have been sent and give them the code. 
10. Perhaps make change for the next transaction. (Change Management)


Step 5 in detail:
1. Generate a seven character seed. This seed should be random and use only 31 characters. [abcdefghjkmnpqrstuvwxyz23456789] 
2. Put a RAIDA ID in front of each seed and so that there are now 25 seeds. If the seed was YU78HNP then you would have 0YU78HNP, 1YU78HNP, 2YU78HNP..24YU78HNP 
3. Create an MD5 hash of each seed. One for each RAIDA. 
4. Change the last four byte of the MD5 hash so that they are all 0xFF.
5. Add up the sum of all the ANs of the tokens you want to put in the locker. The totals will when they get too big.
6. Pick a token randomly from the bank to use for encryption. 
7. Make the "Store Sum" request. 

### Receiver Actions
10. Get the code from the sender. 
11. Call peek and get all the serial numbers and denominations
12. Calculate the total there. 
13. POWN the tokens
14. Find Lost
15. Fix Fracked. 
16. Give change by calling Store Sum and giving the change to the user. 


## PUT

Store is the exact same thing as POWN SUM but instead of adding a sum, a single PAN is used for all the tokens. This PAN becomes the "Locker Number". However, to make it clear that the PAN is a locker number, the last four bytes must be set to all ones (0xFF) by the client. If a client is generating a locker key based on a hash of something, they should create the whole hash and then write all ones over the last four bytes. This is done so that the RAIDA can index the lockercodes easier and determine a coin is in a locker or not without having to have a seperate table or seperate index. It also allows the caller to create more than one locker key at the same time. 


The sum is calculated as per these steps:
1. Eech AN of every coin is converted to four 32 integers from a byte array
```c
i0 = an[0] | an[1]<<8 | an[2]<<16 | an[3]<<24
i1 = an[4] | an[5]<<8 | an[6]<<16 | an[7]<<24
i2 = an[8] | an[9]<<8 | an[10]<<16 | an[11]<<24
i3 = an[12] | an[13]<<8 | an[14]<<16 | an[15]<<24
```
2. The integers are XOR-ed to the accumulator (XOR-ed sum). The initial value of the XOR-ed sum is zeroes
```c
sum[0] ^= i0
sum[1] ^= i1
sum[2] ^= i2
sum[3] ^= i3
```
3. Steps #1 and #2 are repeated for every coin
4. The resulting sum is converted to a byte array (SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU)
```c
SU = sum[0]
SU = sum[0] >> 8
SU = sum[0] >> 16
SU = sum[0] >> 24
SU = sum[1]
SU = sum[1] >> 8
SU = sum[1] >> 16
SU = sum[1] >> 24
SU = sum[2]
SU = sum[2] >> 8
SU = sum[2] >> 16
SU = sum[2] >> 24
SU = sum[3]
SU = sum[3] >> 8
SU = sum[3] >> 16
SU = sum[3] >> 24
```


Sample Put Request:
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN 
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU //The sum of all the ANs of the SNs. See POWN SUM. 
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  //This is the PAN for all the tokens. The last four bytes must be set to binary ones (0xFF) 
3E 3E  //Not Encrypted
```

### Put with multiple lockers:🔴 // Support for the creation of many lockers has not yet been implemented
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN   // Locker with two coins put in it
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU 
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  
DN  SN SN SN SN  // Locker with six coins put in it
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU 
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF 
DN  SN SN SN SN  // Locker with four coins put in it
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  
3E 3E 
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
 

## PEEK
Allows the caller to see all the serial numbers in a RAIDA Locker. This must be called before the REMOVE command can be called. It can also be used to find out how many coins are in the locker without removing them. 

### Encryption if locker key is the user's first coins
If the client is using CloudCoin for the first time, then the locker key must be used to encrypt this request. You will need to use Encryption type 2 in the header. 

The header: 
They Encryption Type (byte 16) will be '2'. Bytes 17-22 (DE SN SN SN SN ) are not used to specify the encryption token but intead are the first five bytes of the locker code in clear text. 
The RAIDA will use these five bytes to find the locker key needed for decryption.
If no locker key is found with matching five bytes, the command returns an error. 

### After Decryption of the Body
After decrypting the body, the RAIDA will use the locker key (LK) to find all the coins that are part of the locker. 

Sample Call
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH 
LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK 
3E 3E

```

Sample Response
```
DN  SN SN SN SN   
DN  SN SN SN SN   
DN  SN SN SN SN  
...
DN  SN SN SN SN  
3E 3E  //Not Encrypted
```

## REMOVE
This command is just like POWN except there is only one AN. 

To Remove tokens from the locker, the user must first call the Peek, then it will know all the serial numbers and all the ANs for the serial numbers. Note that this allows the user to fix fracked easilty.

Sample Call to Remove tokens

```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN
DN  SN SN SN SN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
DN  SN SN SN SN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
DN  SN SN SN SN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN 
DN  SN SN SN SN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
3E 3E  //Not Encrypted
```



## DUMP
This command is just like REMOVE except it returns the serial numbers so that the client does not have to do PEEK first. 

Sample Call to Remove coins:
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN // Lockere Key Number with the last bytes needing to be something
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
3E 3E  //Not Encrypted
```

Sample Response (just like PEEK) Returns all the denomination and Serial Numbers of the coins in the wallet:

```
DN  SN SN SN SN   
DN  SN SN SN SN   
DN  SN SN SN SN  
...
DN  SN SN SN SN  
3E 3E  //Not Encrypted
```

<!--
Response Status | Code
---|---
All Pass | 241
All Fail | 242
Mixed | 243

Response Body if All Pass or All Faile
```
3E 3E
```

Response body only present if there is mixed content
```
MS //Mixed content bitfield. Maybe many bytes. One byte for 8 tokens. 0 is fail, 1 is pass. 
3E 3E  //Not Encrypted
```

## Name Locker
Gives a Locker an easy to remember name and setsup a hash of it's locker key so that the locker can recieve coins without the sender knowing what the locker key is.
Sample Call to Remove tokens
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK // Locker Key to Name
NM NM NM NM NM NM NM NM NM NM NM NM NM NM NM NM // MD5 Hash of Name
3E 3E  //Not Encrypted
```

Sample Response
Response Status | Code
---|---
Link Made| ??
Locker Does not Exist | ??
MD5 Hash Already in Use | ??

Response Body:
```
3E 3E
```

## Transfer
Moves coins from one locker to another locker
Sample Call to Remove tokens
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK // Locker Key to Name
NM NM NM NM NM NM NM NM NM NM NM NM NM NM NM NM // MD5 Hash of Name
$$ $$ $$ // Amount of coins to transfer
3E 3E  //Not Encrypted
```
Sample Response
Response Status | Code
---|---
Link Made| ??
From Locker Does not Exist | ??
To MD5 Hash does not exist | ??
Insufficient funds | ??

Response Body:
```
3E 3E
```


