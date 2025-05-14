# Special Purpose Workstations
<img align="left"  src="https://github.com/worthingtonse/WEST-Protocol/blob/main/zips/dashboard.png" style="border-radius: 5%" >
Special Purpose Workstations are computer designed to be used for a specific role. There are: Treasure, Auditor, Marketing and Performance Monitoring. These macines are designed to be physically located in the same buildings as West Nodes and allow authorized personel to see specific information that is neccessary for their job function. 

Server Owner | Location | Purpose | Notes
---|---|---|---
Auditor | State Auditor's Office | Looks at Token Usage | This is the only service that allows State of Wyoming Officials to understand the activities of customers. Here they can actaully see who is spending what money and what transactions are occuring.
Treasurer | State Treasure's Office | Looks at events dealing with token creation, management and deletion | All events having to do with the creation of tokens are recorded on this one network node
Performance | University of Wyoming's Center for Blockchain and Digital Inovation | Monitors the System's Performance | This shows how many transaction are being handled, wher transactions are comming from in the world, time to execute transactions and other performance indicators
Marketing | TBD | Monitor Activites Caused by Marketing | Allows marketers to use AI, BI to see the results of their marketing campaigns and to develop strategies to improve campaigns. 

Because of security concens, only the Node Located in the Auditor's Office should have auditor functionality. 

We can create a custom dashboard that will display the data as required. The raw data can be imported to a database such as MySQL or into a platform such as DataDog so that meaningful graphic reports can be generated for the purpose of marketing metrics, system tuning, planning and PR. These reports will be generated based on the "Standard Report Records" as shown below: 

## The Events Table: 
Records common events such as authentications, healing, locker services, 
event_id | date_time | ip_address | user_id | service_command_number| amount_processed | execution_time_nanoseconds | response_status
---|---|---|---|---|---|---|---
1 | 6/18/2024, 9:04:25 AM | 189.23.98.223 | e004af90-7a34-454d-af17-6aec64584fe7 | 8 | 10  | 5773 | 241
4 bytes | 4 Bytes | 16 Bytes | 16 Bytes | 2 Bytes | 4 Bytes | 4 Bytes | 2 Byte
Int | DataTime | bin | bin |Small Int | Small Int | Small Int | Small Int

* The first byte of the Service Details spefies is the Service Details are 8 bit or 5 bit. If the first byte is a zero, it uses 8 bit otherwise it uses 5 bit. See the 5 bit table below.

## RAIDA Error Table "raida_errors"
error_id | date_time | error_string
---|---|---
4 bytes | 4 bytes | 255 bytes


## Special command numbers that represent events. Auditors events start with 500. Treasurey events with 600 and Performance Events with 700.  
Event ID / Command Number | Event Title | Event Details | 
---|---|---
500 | KYC Account Created | Records everytime a user is created. User ID is the account creator. IP Address becomes the user ID created.
501 | KYC Account Deleted | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
502 | User Account Created | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
503 | User Account Deleted | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
504 | Auditor Account Created | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
505 | Auditor Account Deleted | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
506 | Tresurer Account Created | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
507 | Tresurer Account Deleted | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
508 | Auditor Account Created | Records everytime a user is created. User ID is the account creator. IP Address  becomes the user ID created.
509 | Admin Account Deleted | Records everytime a user is created. User ID is the account creator. IP Address becomes the user ID created.
600 | Admin Rate changed | Currency ID that was changed. Amount is the rate it was changed to. 
700 | Server Starting | everything else is zeros
701 | Page Written | User ID is blank. Page number goes in amoun
702 | Internal Error Recorded | post a detail to the RAIDA Error Table "raida_errors".
703 | Locker Index File Full | Amount is the amount of records that are full



## Event Monitoring Role Permissions 

Metric Being Recorded | Included in Auditor | Included in Treasure | Included in Performance | Marketing
---|---|---|---|---
Date Time | ✅  |✅| ✅|✅
IP Address | ✅  |✅| ✅|✅
User ID* | ✅  |✅| ❌|✅
Service Command Number | ✅  |✅| ✅|✅
Execution Time Nanoseconds | ✅  |✅| ✅|❌
Response Status | ✅  |✅| ✅|❌
Token Creation | ✅  |✅| ❌|✅
Token Deletion | ✅  |✅| ❌|✅
Set Conversion Fee | ✅  |✅| ❌|❌
User ID Enabling | ✅  |✅| ❌|❌
User ID Disabling | ✅  |✅| ❌|❌
KYC Document Access | ✅  |✅| ❌|❌
Joining User ID with True Identity | ✅ |❌| ❌|❌

The User ID is what allows activities to be traced to people. This is the biggest privacy concern. 


Data is collected on the following subjects: 
1. Chronology. Such as the day and time when the most authentication requests occure.
2. IP Address. Where in the world are authentications happening
3. Service Use. How many tokens were authenticated.
4. Customer ID Use. What did a specific user do. 
5. Technical. Transaction completion times, number of transactions per second, etc.
6. Administrative actions such as creating 


## Dashboards

### Auditor Dashboard
Top show all of the most important facts including the last 24 hours: Total trades, Total Tokens, Total Unique Individuals, Average Trade, Speed of trades, Biggest Trade. Averages, counts, max and mins, totals, uniques. Second row allows for the selection of common reports including biggest trades, volume over the day, AI enabled suspicius activity. See list below.  

### Performance Dashboard
Shows all eight columns. There is a filter by drop down on each column head. Events can be sorted by start and end date. They can choose just one command to look at. "All" is the default. Sort based on user ID is possible. Sort by ip address. Sort by nanoseconds taken. Sort by Respons code ('All' is default. There is a button "Search" that causes the query to execute. 

Reports Buttons
1. Search 
2. Show Averages
3. Show Counts
4. Show Max and Min
5. Show Totals
6. Show Uniques


<!--
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
