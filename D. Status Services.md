![Status Services](zips/status.png)

# Status Services (Command Group 0)

Explains that Status of the RAIDA.


Command Code | Service | Description
--- | --- | :---: 
00 | [Echo](#echo) | Sends a challenge handshake to test connection, encryption and mutual authenticate.
01 | [Version](#version)  | Returns the version of the protocol
02 | [Status](#status) | Returns status and performance indicators for diagnostics (Not Implemented)
03 | [Count Coins](#count-coins) | Returns the number of tokens on the RAIDA

Byte Code | Meaning
---|---
CH | Challenge. Client's encrypted challenge that the servers must decrypt. Used for mutual authentication.
3E | Ending bytes
VE | Version of the code
TO | Total coins
AU | Authentication (Password)(


# Echo  

Echo tests connection and the encryption to ensure it is working. This echo is more for debugging than production work.

Typically it is enough to send 18 bytes in the body. 
But echo can accept more bytes between CH and 3E if necessary. 
For example if we want to test big packets we may want to send a bigger packet.

Request Body (18 bytes):
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
3E 3E //Not Encrypted
```

Reponse will have no body. The status code in the response header will be:

0xFA  //Decimal code 250


# Version

Command Group: 0

Command Number: 1

Request (no parameters)
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
3E 3E
```
Response: Success Status 250

Version (8 bytes)
```
VE VE VE VE VE VE VE VE
3E 3E
```

# Count Coins
Returns the total number of coins the RAIDA has. This value is not updated in real time but only once every 20 minutes. This time can be changed in the future. 

## Example Request Body:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
00 00 00 00 //for future use 
AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU //Optional password. All zeros if not used. 
3E 3E
```
Response Status | Code
---|---
Success | 250
Failure | 251

Response:

## Response
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
TO TO TO TO TO TO  //Six Byte total amount of coins
3E 3E
```

# Status
We can create a custom dashboard that will display the data as required. The raw data can be imported to a database such as MySQL or into a platform such as DataDog so that meaningful graphic reports can be generated for the purpose of marketing metrics, system tuning, planning and PR. These reports will be generated based on the "Standard Report Records" as shown below: 

## Sample Standard Reporting Record: 

Date Time | IP Address | User Key | Service Command Number| Service Details | Execution Time Nanoseconds | Response Status
---|---|---|---|---|---|---
6/18/2024, 9:04:25 AM | 189.23.98.223 | e004af90-7a34-454d-af17-6aec64584fe7 | 8 | 10 TOKENS * | 5773 | 241
4 Bytes | 16 Bytes | 16 Bytes | 1 Byte | 23 Bytes | 2 Bytes | 2 Byte
* The first byte of the Service Details spefies is the Service Details are 8 bit or 5 bit. If the first byte is a zero, it uses 8 bit otherwise it uses 5 bit. See the 5 bit table below. 

Data is collected on the following subjects: 
1. Chronology. Such as the day and time when the most authentication requests occure.
2. IP Address. Where in the world are authentications happening
3. Service Use. How many tokens were authenticated.
4. Customer Key Use. What did a specific user do. 
5. Technical. Transaction completion times, number of transactions per second, etc. 

## Implementation
We have not implemented this yet because we do not know the specific needs for data. Each of the 25 RAIDA will be tasked with loggin different commands. There are about 28 commands and each RAIDA can monitor one or two of them. This is one of the few services where the RAIDA returns more data than it receives. Thus it could be a target for a DDOS attack. To prevent any DDOS attacks, this service can only be accessed by Treasures and Administrators.  

## Example Request Body:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
00 00 00 00 //Time to start the status return used in the C programming langage
00 00 00 00 // Time to end the status. Zero means until now.
00 00        // Service to return. Zero means all of them .
AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU //Authorization Password. User keys cannot access this service. 
3E 3E
```
Response Status | Code
---|---
Success | 250
Failure | 251

## Five Bit Table
Dec | Letter
---|---
0 | 0/O
1 | 1/I
2 | 2
3 | 3
4 | 5
5 | 5
6 | 6
7 | 7
8 | 8
9 | 9
10 | A
11 | B
12 | C
13 | D
14 | E
15 | F
16 | G
17 | H
18 | J
19 | K
20 | L
21 | M
22 | N
23 | P
24 | Q
25 | R
26 | S
25 | T
26 | U
27 | V
28 | W
29 | X
30 | Y
31 | Z


Response:

## Response
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
CT CT //Number of rows returned
RC RC .. //Standard response record #1. 64 byes fixed for each record.
RC RC .. //Standard response record #2. 64 byes fixed for each record.
...
RC RC .. //Standard response record #N. 64 byes fixed for each record. 
3E 3E
```



<!--
Code | Meaning
--|--
CH |  Challenge (16 byte randome number)
DU |  Days of uptime. How many days the RAIDA has been up since last failure .
NE | No Error Time. How many days since the RAIDA has had to change its SN table to correct errors. 
ER | Errors Last time. How many errors were found the last time it found errors.  
AU |  Authorizatio number (Secret between RAIDA servers like a password)
DN | Denomination of the serial numbers that the RAIDA is to return. 


# Performance (This service is not implemented yet)
The job of the performance service is to allow admins to evaluat the use of their RAIDA. 
The performance returns up to 255 performance indicators. 
The is done by key value pairs These keys and values are based on the Performance Table below. 

Returns: three ranges (contiguous serial numbers) and three single (incontiguous) serial numbers. It also shows the denomination but the client will probably 
rember what denomination they asked for. 
```
DU DU NE ER    //Discolosures
NR NS 
RR RR RR RR RR RR RR RR //Range: 4 bytes start, 4 bytes end. 
RR RR RR RR RR RR RR RR
RR RR RR RR RR RR RR RR 
SN SN SN SN
SN SN SN SN
SN SN SN SN 
```

Example Request Body:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
KY MX MX MX //Key Maxium Records to return
AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU //Authorization Password. Only RAIDA can use this service and a password is required
PR PR PR PR PR PR PR PR PR PR PR PR PR PR PR PR //Comand Parameters. Each one may have different meanings 
3E 3E
```

Sample Responses with four key/value pairs:
```
DT DT DT DT TT TT TT TT //Date (a day), Total calls for that day)   //Key Value
3E 3E
```

Number of times a service has been called

Amount handled per request. 
Key | Name | Parameters
---|---|---
0 | Echo | None
10 | Detect | 4 byte starting date. 4 byte ending date (optional)
20 | Pown | 4 byte starting date. 4 byte ending date (optional)
30 | Pown with Check Sum and Add | 4 byte starting date. 4 byte ending date (optional)
40 | Get Ticket | 4 byte starting date. 4 byte ending date (optional)
50 | Validate Ticket | 4 byte starting date. 4 byte ending date (optional)
60 | Find | 4 byte starting date. 4 byte ending date (optional)
80 | Fix | 4 byte starting date. 4 byte ending date (optional)
81 | Put in Locker | 4 byte starting date. 4 byte ending date (optional)
82 | Get From Locker | 4 byte starting date. 4 byte ending date (optional)
83 | Read Meta | 4 byte starting date. 4 byte ending date (optional)
90 | Fix with Remote Key | 4 byte starting date. 4 byte ending date (optional)
100 | Show Stats | 4 byte starting date. 4 byte ending date (optional)
120 | Get Available SNS | 4 byte starting date. 4 byte ending date (optional)
130 | 	Create Coins | 4 byte starting date. 4 byte ending date (optional)
140 | Delete Coins | 4 byte starting date. 4 byte ending date (optional)
150 | Audit Coins | 4 byte starting date. 4 byte ending date (optional)
200 | Put Key | 4 byte starting date. 4 byte ending date (optional)
210 | Get Key | 4 byte starting date. 4 byte ending date (optional)
220 | Exchange Key | 4 byte starting date. 4 byte ending date (optional)
-->


