![Healing Services](zips/fix.png)
# RAIDA Healing Service (Command Group 2)

These services allow the RAIDA to heal when a token becomes counterfeit on one RAIDA and authentic on others. 


Command Code | Service 
--- | --- 
40 | [Get Ticket](#get-ticket)
41 | [Get Ticket By Sum](#get-ticket-by-sum)
50 | [Validate Ticket](#validate-ticket)
60 | [Find](#find)
80 | [Fix](#fix) 

# Get Ticket
The GET TICKET service is nearly identical to DETECT accept a Ticket is returned. 
A ticket is a 4-byte random number that the RAIDA gives to the caller as proof that the tokens are authentic. 
This ticket can be used by the client to prove its authenticity to other RAIDA. 
The Ticket can be claimed one time by each RAIDA server. After the ticket grows old it times out.

The client gets a one ticket from a RAIDA and can send that ticket to 24 other RAIDA. 

Example Request Body with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN  SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
3E 3E //Not Encryption
```

Response Status | Code
---|---
All Pass | 241
All Fail | 242
Mixed | 243



Example Response if all pass :
```hex
MT MT MT MT  //Master Ticket
3E 3E //Not Encryption
```
Example Response if all mixed :
```hex
MS MS MS MS MS MT MT MT MT  //Master Ticket 
3E 3E 
```

# GET TICKET BY SUM
Like DETECT SUM, except it returns a ticket. The ANs are added up and put in one number. 
This service can be much faster than GET TICKET, especially if there are lots of tokens being detected. 
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

Example Response if pass :
```hex
MT MT MT MT  //Master Ticket
3E 3E //Not Encryption
```
Example Response if fail :
```hex
3E 3E 
```


# VALIDATE TICKET
The Validate Ticket protocol allows the client to see if a ticket belongs to a serial number.
It is used to prove that a person owns a token without them having to show the token. 
This is useful if you are using WEST Tokens to authenticate a person or thing in your own application. 

Validate ticket is not encrypted.

EXAMPLE BODY OF REQUEST
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
RI
TK TK TK TK 
3E 3E //Not Encryption
```

EXAMPLE REPLY: Shows seven serial numbers returned. The number is divisable by 5 bytes. 
```
DN  SN SN SN SN 
DN  SN SN SN SN 
DN  SN SN SN SN 
DN  SN SN SN SN 
DN  SN SN SN SN 
DN  SN SN SN SN 
DN  SN SN SN SN 
3E 3E 
```


# FIND 

A token is in Limbo when the client makes a request to POWN but there is no response from the RAIDA. 
The client is uncertain if the RAIDA got the message and turned the AN to the PAN or if the RAIDA did not get the message and the AN is unchanged. 

To get the token out of Limbo the client sends a Find rquest and sends the AN and the PAN to the RAIDA and asks which one is it's AN. 
The RAIDA will tell the Client if the AN is authentic, the PAN is authentic or if neither is authentic. 

The Fix Lost service allows the caller to send both AN and PAN to see if they are authentic. 
The service then responds with the correct answer. If neither are correct, it will respond with a status of "Neither".  '0x00'= Neither. "0x01" = AN and "0x02"=PAN. 

Example with four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
DN SN SN SN SN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  
3E 3E //Not Encryption
```

Response Statuses

Code | Status | Description
---|---|---
208 | Find All Neither | Neither the ANs or the PANs on any of the tokens were authentic 
209 | Find All AN | The ANs of all of the tokens were authentic 
210 | Find All PAN | The PANs of all of the tokens were authentic
211 | Find mixed | The tokens had a mix of ANs good, PANs good and Neither good. 


Mixed Results Codes

Hex Code | Meaning
---|---
0x00 | Nether the AN or the PAN presented by the Client was authentic. 
0x01 | The AN is athentic and the RAIDA has it in its database.  
0x02 | The PAN is authentic and the RAIDA has the PAN as in its database. 

Response Body if Results are Mixed for four tokens: One byte per token
```
01 
01 
00 
02   
//1st & 2nd tokens the ANs matched. 3rd neither matched. 4th the PAN matched. 
```

# Fix

All 25 RAIDA should agree that a token is authentic.  
When all RAIDA are not in agreement about the authenticity of a token, we call the token "Fractured" or "Fract" for short. 
The Fix protocol allows for Fract tokens to become whole. 

The fix procol must look at two config files to determine which other RAIDA to trust and how many of the RAIDA need to be in agreement to switch the AN to the client's request.
All of the file's data are located in the file's name and the file is located in the Config folder

Example config file that tells this RAIDA can accept tickets from all 25 RAIDA and it needs 13 good tickets to turn the AN. 
```
trusted.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.1.13.config
```
This example file name shows that the RAIDA is to trust no other RAIDA except for RAIDA 0 and that it must have one good ticket. 
```
trusted.1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.1.config
```
This example show that it can get tickets from any RAIDA except 10 and that it needs 13 good tickets. 
```
trusted.1.1.1.1.1.1.1.1.1.1.0.1.1.1.1.1.1.1.1.1.1.1.1.1.1.13.config
```
Each RAIDA Trusts the majority of other RAIDA. The client can get 'Tickets' from RAIDA that think the token is authentic and give them to the fract raida. 
The RAIDA requires 13 of the 25 RAIDA to change its AN to the AN that the client specifies. 

The fract RAIDA will always receive 25 tickts although they will all be zeros unless the actaully contain a ticket. So the majority of the tickets are just place holders. 

The Fix protocl may recieve empty tickets with all "00000". In this case there is no ticket. 

### Client Steps
1. Client receves a "fail" from one or more of the RAIDA (up to 12).
2. Client requests tickets from 13 or more RAIDA that are good.
3. Client sends all the tickets to all the RAIDA that are fracked (recieved a fail).
4. RAIDA checks the tickets. If the tickets are good then the fracked RAIDA set the AN that the clients specifies.  


### RAIDA Steps
1. The fracted RAIDA recieves Tickets with the Serial Numbers of the tokens that are fract and the Proposed Authenticity Numbers (or Proposed Authenticty Number Generator)  that client wants to give them.
2. The fracted RAIDA sends the Tickets to all the trused RAIDA in parallel using the VALIDATE TICKE protocol. 
3. Trusted RAIDA respond with the serial numbers associated with they ticket.
7. Fract RAIDA compares the Serial Numbers returned by the Good RAIDA with the Serial Numbers specified by the Client. 
8. If there are 13 Good RAIDAs that agree with what the Serial numbers that the client sent, the Authenticty Numbers on the Fract RAIDA are changed. 
9. All Pass is returned if all the token were fixed. All fail if none of the tokens were fixed. Mixed if some of the tokens were fixed. 

### Fix Algorithm

Fix allows you to fix multiple tokens using tickets that it got from the Get_Ticket service. 
Each ticket can only be used one time per RAIDA. 
Each RAIDA can use the same ticket to fix. 
This allows the client to use all the tickets returned from a Get_Ticket request to fix all the fracked tokens on all the RAIDA without needing to go and get more tickets.  The Fix then calls the "Validate Ticket" service and sees what SNs are returned. 
If 13 out of 25 RAIDA return the same SN, that SN's ANs are changed to the PAN supplied by the Client.


Version 0 Example Request Body with four tokens:
```dif
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN SN SN SN SN 
DN SN SN SN SN 
DN SN SN SN SN 
DN SN SN SN SN 
PG PG PG PG PG PG PG PG PG PG PG PG PG PG PG PG  //PAN Generator that will be used to generate PANs.  
TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  //Tickts for RAIDA 0 through RAIDA 4. 
TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  //Tickts for RAIDA 5 through RAIDA 9. 
TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  //Tickts for RAIDA 10 through RAIDA 14. 
TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  //Tickts for RAIDA 15 through RAIDA 19. 
TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  TK TK TK TK  //Tickts for RAIDA 20 through RAIDA 24. 
3E 3E //Not Encryption
```
The PAN is determined by concatinating the RAIDA ID, Serial Number and the PG. Like this

```diff
- WARNING: Client should send a different PG to each RAIDA otherwise the token will be stolen by any other RAIDA admin. 
```

1. The Client will generate a 16 byte random seed.
2. The Client will send this seed to the RAIDA. This seed is called a "PAN Generator".
3. The Client and the RAIDA will use an algorithm to figure out what the Authenticity numbers will be for each serial number that fixes. 
4. The Autheticty numbers for each serial number will be calculated by putting together the RAIDA Number (one byte), the token's denomination (one byte) and serial Number (four bytes) and the PG (16 bytes) t. 

```
1 + 0xFE + 1632435 + 9195C7A60CF0471BBE9B66144F5681C5 and then running an MD5 hash agains that.
```
Then the concatenated string is: 
```
PAN = MD5( "01FE18E8B39195C7A60CF0471BBE9B66144F5681C5"); 
```
So the AN will be:
```
fd1495be732ab2f6959559f3041c72c0
```

Example Request Body With Actual Bytes for an attempt to fix RAIDA 12 for four tokens:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
00 00 01  
05 00 23   
05 00 24  
05 00 25  
13 0A B9 AD 18 5E 45 1C 8E F4 11 13 74 95 65 FB
00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  //NO Tickts for RAIDA 0 through RAIDA 4. 
8A F4 55 E7  00 00 00 00  A4 BB 4A BF  B6 5C 74 87  00 00 00 00  //Tickts for RAIDA 5, 7 and 8. RAIDA 6 and 9 are empty. 
00 00 00 00  08 4C 38 8B  00 00 00 00  00 00 00 00  00 00 00 00  //Tickts for RAIDA 11.  RAIDA 10,12, 13 and 14 are empty.  
00 00 00 00  75 E5 6A A4  00 00 00 00  56 3C 57 72  00 00 00 00  //Tickts for RAIDA 16 and 18. RAIDA 15,17 and 19 are empty.  
00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00  //NO Tickts for RAIDA 20 through RAIDA 24. 
3E 3E //Not Encryption
```

Example Respons: 
There will only be a response body if there is mixed results. 
```
MS MS MS MS MS MT MT MT MT  //Master Ticket
3E 3E 
```


