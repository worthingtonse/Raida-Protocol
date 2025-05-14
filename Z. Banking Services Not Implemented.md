# Banking Services
Allows a user to control their account. The user must already have been given an account but the network administrator.

Command Number | Command
---|---
110 | [Balance](#balance)
111 | [Transfer](#transfer)
112 | [Read Statement](#read_statement)
113 | [Confirm Payment](#confirm_payment)

## Balance
Returns that number of Tokens a user has in their account.
Data Type of the total: DECIMAL(27,18) 12 bytes. 4 bytes for each 9 digits. 


Example Request Body:
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
00 00 00 00 //Account Number
AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU //Account Key
3E 3E
```

Response
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
TO TO TO TO TO TO TO TO TO TO TO TO  //12 Byte total amount of coins in the account
3E 3E
```

## Transfer
Moves coins from owner's account to another

Example Request Body:
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
00 00 00 00 //Account Number
AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU AU //Account Key
00 00 00 00 //To account number
TO TO TO TO TO TO TO TO TO TO TO TO  //12 Byte total amount of coins to transfer
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //GUID of the transaction
YY MM DD HH MM SS //Client's time stamp Must be in UTC
TY //There will be two transactions created. One for the sender and one for the receiver. The sender has 03 and the receiver has 02.
ST ST ST ST.....//Fix length Meta Data Stripe up to 50 bytes.
3E 3E
```

Response
```
// No body, just a status of All Pass or a status of "failed to authenticate"
```


## Statement
Allows the user to read statements. The user can specify how many rows they want to download. They can also specify the date that should start to download.

Request
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
OW OW OW
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN
RW  //Number of rows * 100. So 0x01 means return 100 rows. 00 Means max rows.
YR //The Year  00 = 2000. 255 = 2255. Time must be in UTC
MM //month
DY //Day of the Month
RT //Return Code. 00=stripe 01=mirror, 11 = 2nd mirror, FF Stripe, Miorror and 2nd Mirror
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //Reserved for Future Use. Put random numbers. 
E3 E3

```
Reponse body for three Records:
```
CB CB CB CB //Current Balance
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //GUID
TT //Transaction type
AM AM AM AM //Amount
BL BL BL BL //Balance
TS TS TS TS TS TS //Time Stamp
ME ME ME ME ME ME ... ME ME // Fixed 50 bytes for memo.
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //GUID
TT //Transaction type
AM AM AM AM //Amount
BL BL BL BL //Balance
TS TS TS TS TS TS//Time Stamp
ME ME ME ME ME ME ... ME ME // Fixed 50 bytes for memo.
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //GUID
TT //Transaction type
AM AM AM AM //Amount
BL BL BL BL //Balance
TS TS TS TS TS TS//Time Stamp
ME ME ME ME ME ME ... ME ME // Fixed 50 bytes for memo.
3E 3E
```
Reponse body if there are no statements:
```
Show Statements: No Statements found. Staus Code: 120

//Nothing in body if there is no statements.
```

## Confirm Payment

Allows anyone with a statement's GUID to read about that transaction.

Request Body:
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
RR RR RR RR RR RR RR RR // eight bytes of random numbers
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID  //GUID
RR RR RR RR RR RR RR RR  //eight more bytes of random numbers
3E 3E
```

Reponse body for four coins:
```
TT //Transaction type
AM AM AM AM AM AM AM AM AM AM AM AM //Amount
TS TS TS TS //Time Stamp
OW OW OW OW  //Account that received the transfer (got paid)
SE SE SE SE  //Sender
ME ME ME ME ME ME 00 00 ... //Memo ends with two null characters.
3E 3E
```
