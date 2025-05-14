# Remote Procedure Call Services
RPC Services allows clients to make simple calls to get simple information. 

Command Group Number is 12

This has not been developed yet. 

![Status Services](zips/gpg.png)

# nslookup (Command Group _)

Explains that Status of the RAIDA.


Command Code | Service | Description
--- | --- | :---: 
120 | [nslookup](#nslookup) | Looks up the IP address of a server
121 | [host](#host)  | Returns the Host file on the RAIDA. 
122 | [Get Exchange Rate](#get_exchange_rate) | Gets the exchange rate between West and another currency.
123 | [Ask AI](#ask_ai) | Asks AI a question
124 | [GET URL](#get-url) | Gets the return data from a URL call. 
125 | [Payment Sent Alert](#payment-sent-alert) | Gets the return data from a URL call. 
126 | Ghost Chats | Sends Messages to other users using quantum safe encryption. 

## Payment Sent Alert
* The client must check with the exchange rate web API and decide when to convert. 
* The client will not get to specify the price due to the slowness of crypto transactions.
* The RAIDA will then check the price with the same API, if the prices are within 1% of each other, the transaction shall be made. 
* The Administrators must have a market maker account with enough West tokens in it to handle the transaction.
* First come first serve with the market account to keep it simple. If the market account runs out conversion stops.
* The RAIDA will make a call to a block explore for the transaction supplied by the client
* RAIDA checks the data of the transactoin. If it is too old it is rejected.
* RAIDA checks the list of recetn transactions and makes sure tokens have not been isssued yet.
* RAIDA puts coins from the market locker into the client's locker. 

The user sends:
* The cryptocurrency-code they send coins to. 
* The transaction numbers.
* Their crypto wallet address that coins were sent from.
* The receipt number (not required )
* The memo (up to 1300 bytes allows for 20KB on 16 RAIDAs)

Sample Request
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK //Locker key that they want the coins to be put into.
CD CD CD // Currency Code that was sent
TR TR TR TR TR TR TR TR TR TR TR TR TR TR TR TR // Transaction ID
TR TR TR TR TR TR TR TR TR TR TR TR TR TR TR TR 
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID  //The receipt ID 
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME
...
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME //up to 1300 bytes of memo data. Optional.
3E 3E //Not Encrypted
```


Response Status | Code
---|---
Success | 250
Not enough market coins| ??
Price differnt more than 1% | ??
Address did not fit allowable format | ??
Memo too long | ??
Receipt ID in use| ?? 
Locker not found| ?? 

```
 //Empty 
3E 3E 

```



## nslookup 
Allow the caller to request the first IP returned from an nslookup called on the RAIDA. The 
fully qualified domain name requested can be up to 64 bytes. 

Request (no parameters)
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
RR
FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN //64 Bytes Fixed
FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN
FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN
FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN FQ DN 
3E 3E
```

RR - Record Type. 
A - IPv4 address = 0x1
SRV - ServiceRecord = 0x2


Response: Success Status 250
NXDOMAIN Status 185

Version (8 bytes)
```
IP V4 IP V4 // four byte IPV4 address.
PN PN // port number
3E 3E
```

