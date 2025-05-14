![Authentication Services](zips/authentication.png)
# Authentication Services (Command Group 1)

These services check the authenticty of tokens and allow change of ownership.

Key Terms: 

AN: Authenticity Number. A 16 byte password that they coin has. Usually shown as 32 hexidecimal characters. 

SN: Serial Number. Every coin has a unique 4 byte serial number .

PAN or PN: Proposed Authenticity Number. The new AN that the client wants to repace the old AN with. 

POWN: Password Own. To ask the RAIDA to change the AN to the PAN.

Fracked: When a coin has one or more RAIDA that think it is counterfeit but more than 13 that think it is authentic. 

Command Code | Service  | Description
--- | --- |---
10 | [Detect](#detect) | Tells if token is authentic without changing AN
11 | [Detect Sum](#detect-sum) | Uses sum of ANs to mass authenticate
20 | [Pown](#pown)| Changes AN 
21 | [Pown Sum](pown-sum)| Changes ANs based on sum of ANs
22 | [Hash PANG](hash-pang) | Changes lots of ANs a requires the least amount of badwidth. 

## Code meanings
Code | Meaning | Sample in HEX | Description
---|---|---|---
RD | Random | 6 | A random byte
DN | Denomination | 1 | A denomination code (See Request Header Denominations)
SN | Serial Number | 16777215 | Four bytes of serial numbers
AN | Authenticity Number | F5DD153926DA42E8A062A915AD763DF0 | The token's password
PN | Proposed Authenticity Number | 6A3C3E384566483A9EE629D6BB38BCB9 | The new password
CH | Challenge | 84566483A9EE629D6BB38BCB96A3C3E3 | A random number that they RAIDA unencrypt 


# DETECT 

DETECT does not require a PANs (Proposed Authenticty Number) and requires less bandwidth and proccessing power than P'OWN. 
This is used as a fast way of checking to see if tokens are authentic as in a "Check Health" function on a client.  
Use UDP if sending less than 65 notes and TCP if sending more. 
It is recomended that this servie only be used if you have already used the DETECT SUM service and it responded with a All Fail status. 

REQUEST: Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
3E 3E //Not Encrypted
```
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
MS //Mixed content bitfield. Maybe many bytes. One byte for 8 tokens. 0 is fail, 1 is pass. Will be padded with zero bits to the end of a byte. 
3E 3E  //Not Encrypted
```

# DETECT SUM
Like DETECT, but does not require ANs. Instead, the ANs are added up and put in one number. 
This service can be much faster than DETECT, especially if there are lots of tokens being detected. 
Use UDP if sending between 65 and 275 notes. 
Use TCP if sending more than 275 notes, you should use TCP and it is probably better to use regular detect in case one of the tokens is bad.  

The sum is calculated as per these steps:
1. Eech AN of every coin is converted to four 32 integers from a byte array
```
i0 = an[0] | an[1]<<8 | an[2]<<16 | an[3]<<24
i1 = an[4] | an[5]<<8 | an[6]<<16 | an[7]<<24
i2 = an[8] | an[9]<<8 | an[10]<<16 | an[11]<<24
i3 = an[12] | an[13]<<8 | an[14]<<16 | an[15]<<24
```
2. The integers are XOR-ed to the accumulator (XOR-ed sum). The initial value of the XOR-ed sum is zeroes
```
sum[0] ^= i0
sum[1] ^= i1
sum[2] ^= i2
sum[3] ^= i3
```
3. Steps #1 and #2 are repeated for every coin
4. The resulting sum is converted to a byte array (SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU)
```
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


REQUEST: Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN 
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU //Some of all Authenticity Numbers with the higest numbers using ADD and not carrying about the Carrie bit (on the processor flags)
3E 3E //Not Encrypted
```
Response Status | Code | Note
---|---|---
All Pass | 241 | All the tokens were authentic. 
All Fail | 242 //really means sum does not add up. So, the tokens may all be good but just one is bad so the DETECT service should be run to no exactly which tokens are bad. 


Response Body if All Pass or sum does not add up (All Fail). 
```
3E 3E
```

# P'OWN 
P'Own means "Password Own". The client sends the Serial Number, Authenticity Number and the Proposed Authenticity Number. The AN and PAN must be different or an error 
will be returned with a status that the ANs and PANs matched. The server checks to see if the AN that the client sent matches the AN in the table.
If it matches, the AN in the table is replaced with the PAN provided by the client and the "pass" status is returned. 
If multiple tokens are sent then the server may return "all pass" or "all fail" status. 
If some of the tokens are found authentic and some are found counterfeit, then the "mixed" status is returned along with a bitfield. 
The bitfield has one bit per token. A bit set to '1' is a pass and each bit set to '0' is a fail. Zeros are padded to the byte. 
Since the server only returns whole bytes, the client must know when to stop reading the bitfield based on how many tokens were sent.

Like all services, the p'own allows the client to send a 16 byte challenge. 
The server returns the first four bytes of an MD5 hash of this challenge in the response header. 
This allows the client to be certain that the response came from the RAIDA since only the RAIDA would be able to decrypt the challenge.

REQUEST: DN (Denomination) is one byte. SN(Serial Number) is 4 bytes
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN 
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN
3E 3E  //Not Encrypted
```
Reponse body if all passed | Reponse body if all failed: | Response body if mixed with passes and fails:
---|---|---
no response body | no response body | MT MT MT MT MS  //The MT are just zeros. 0x00 00 00 00. They are for future use in case we want to return master tickets.

MS means Mixed Status. Each bit returned represents the status of one token. If the bit is a zero then that token has failed. If the bit is a 1 then that token is authentic. 



Response Status | Code
---|---
All Pass | 241
All Fail | 242
Mixed | 243


# P'OWN SUM
The client must add up all of the AN numbers to get a sum. Then it includes all the SNs of the tokens that are part of the equation. 

Steps for calculating the XOR-ed sum are the same as for the DetectSum service.


```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN 
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU //The sum of all the ANs of the tokens added together (no carry)
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD //number to add to the current ANs to derive the PAN 
3E 3E  //Not Encrypted
```
You will need to calculate the new Authenticity Number by adding it to the AD. This is done by summing the AN with the AD and truncating the the number . 
so D4785BED + 48BD4125 = 11D359D12 but we drop the leading 1 so the new AN becomes 1D359D12.  


Status| Response Body Contents
---|---
All Passed | Will not include a response body | Will not include a response body | 
All Failed | Will not include a response body
Mixed | MT MT MT MT MS  //The MT are just zeros. 0x00 00 00 00. They are for future use. The number of MS bytes depends on the number of tokens p'owned. 

MS means Mixed Status. Each bit returned represents the status of one token. If the bit is a zero then that token has failed. If the bit is a 1 then that token is authentic. 


Response Status | Code
---|---
All Pass | 241
All Fail | 242
Mixed | 243
 

# HASH PANG 
Not Implemented Yet


The HASH PANG service allows for the includsion of Serial number ranges. Suppose that a user is changing the passwords for coins SN 3000 - 4000. Instead of needing to list every Serial number, the call just includes the range. This makes the service good for handling large requests.  The client must evaluate an authenticity number for a group of coins. Then the client sends the number to the RAIDA servers. The RAIDA servers in turn do the same evaluation for the serial numbers. If the resulting AN matches the AN sent by the Client the result is considered authentic. The client can send ranges of serial numbers along with individual numbers.

The algorithm used to evaluate the Group Hash is md5. It doesn't have to be strong because the RAIDA server is supposed to already know all the ANs of the original coins. The client iteratively adds each AN to the Group MD5 hash. In the end the Client gets the final MD5 hash that becomes the Group hash.

On each calculation the Serial Number is prepended to the AN

Example:
sn 10, an 3031900e6c47411da57192574faa61c3
sn 20, an 5ab8661c15c44bc995f5bda379efdaf4
sn 30, an ed797feb38b0416ba2bd1d7db29f68d7

// binary representation 
input0 {0x0a, 0x30, 0x31, 0x90, 0x0e, 0x6c, 0x47, 0x41, 0x1d, 0xa5, 0x71, 0x92, 0x57, 0x4f, 0xaa, 0x61, 0xc3}
input1 {0x14, 0x5a, 0xb8, 0x66, 0x1c, 0x15, 0xc4, 0x4b, 0xc9, 0x95, 0xf5, 0xbd, 0xa3, 0x79, 0xef, 0xda, 0xf4}
input2 {0x1e, 0xed, 0x79, 0x7f, 0xeb, 0x38, 0xb0, 0x41, 0x6b, 0xa2, 0xbd, 0x1d, 0x7d, 0xb2, 0x9f, 0x68, 0xd7}

	md5Init(&ctx);
	md5Update(&ctx, (uint8_t *)input0, strlen(input0));
	md5Update(&ctx, (uint8_t *)input1, strlen(input1));
	md5Update(&ctx, (uint8_t *)input2, strlen(input2));
	md5Finalize(&ctx);

The resulting Group hash is 21e46bbb18d8ce646a840ccda4602f56

The call supports sending a range of serial numbers.

The PAN number is calculated the same way as in the PANG call using the PG number

Example with three ranges and three single coins:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
NR NR  
DN RR RR RR DN RR RR RR // Starting and ending range
DN RR RR RR DN RR RR RR  
DN RR RR RR DN RR RR RR 
DN SN SN SN //Singles
DN SN SN SN
DN SN SN SN 
HA SH HA SH HA SH HA SH HA SH HA SH HA SH HA SH  
PG PG PG PG PG PG PG PG PG PG PG PG PG PG PG PG
3E 3E 
```

NR - number of ranges
DN RR RR RR DN RR RR RR - six-byte range 'from:to'
SN SN SN SN - individual serial number
HS - Group hash AN
PG - Pan Generator

Response Status | Code
---|---
All Pass | 241
All Fail | 242

