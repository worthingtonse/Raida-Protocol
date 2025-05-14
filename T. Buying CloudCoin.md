
# Buying CloudCoin Services

### Steps to buy CloudCoins
1. LIST: The buyer will look at the coins that are for sale by calling the List Lockers For Sale command.
2. Send BTC: The buyer then sends BTC to the RAIDA's wallet for the amount of the locker they will purchase.
3. Wait: The buyer waits for the transaction to clear.
4. BUY: The buyer sends the transaction id and their BTC address to the RAIDA using the "Buy" service.
5. PEEK: The buyer now has the locker key with his coins. He callse the PEEK service.
6. REMOVE: The buyer then calls the REMOVE service and then has his coins. 

## Commands

Command Code | Service | Description
---|---|---
🔴86 | [LIST LOCKERS](#list-lockers)| Returns a list of lockers that are for sale.  
🔴110 | [BUY LOCKER](#buy-locker) | Allows the buyer to put the CC he purchased into his locker. 
🔴111 | [SEND CRYPTO](#send-crypto) | Allows the RAIDAs to tell the BTC RAIDA to send coins. 

## LIST LOCKERS
This returns a list of For Sale lockers of the coin type (BTC, Monero, etc) specified. 
It returns the total coins in the locker and the price of the locker. 

The list is ordered by price assending. If there are more than one locker with the same price, it will show the lockers with the least
amount of coins in them first. 

Sample Request:
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
CT CT CT // Coin Type such as BTC, XRM, ETC..
## // Number of records to return (up to 256)
3E 3E
```

Sample Response:
```c
CT CT TC TC TC TC $$ $$ $$ $$ //Coin Type, Total coins in locker, price.  
CT CT TC TC TC TC $$ $$ $$ $$
CT CT TC TC TC TC $$ $$ $$ $$ 
3E 3E
```

<!--
![Reserve Locker](zips/lockers.png) -->
## BUY LOCKER
Look at the steps that the buyer must make:  [Steps to buy CloudCoins](#steps-to-buy-cloudcoins)

After receiving the BUY LOCKER command the RAIDA will:
1. Check the Transaction Log to make sure this request has not been made before.
2. Look at the senders Bitcoin address and transaction number and check the blockchain explorer to see if the transaction is there. 
3. Check to see if there is a For Sale Locker with the correct coin type, price and total coins.
4. Make sure that the amount the buyer sent matches the amount in the locker (Ok to round up or down)
5. Change the ANs of the coins in the For Sale Locker to the PN specified by the Buyer.
6. Send a response back to the buyer. 
7. Calculate fees that the seller must pay. 
8. Order the Python server to send BTC minus the fee to CC Seller's crypto address.
9. Record the transaction.
10. Make a record of the fees collected.
11. Figure out how to get the fees to the RAIDA Admins. 


Note: The RAIDA must use the CD CD CD, $$ $$ $$ $$ and TO TO TO TO to find the "For Sale Locker" that the buyer wants to buy.  

Sample Request
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  //The locker key the buyer wants to use for their locker
CD CD CD //currency code to look out for. ASCII like BTC or XMR
$$ $$ $$ $$ TO TO TO TO  // the per CC price in Bitcon or Monetor or other crypto. And the total coins to be purchased 
AS // The Bitcoin Address that the buyer used to send BTC to the RAIDA. 
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD // 32-35 bytes. Buyer's cryptocurrency address used to send
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD // Crypto to the RAIDA's bitcoin address
AD AD AD
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //The buyers transaction ID created by sending BTC to the RAIDA.(32 bytes)
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID  
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME //Memo should contain identifyable tansaction information 
...
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME //Helps buyer recover coins if something goes wrong.
3E 3E //Not Encrypted
```

Sample Response: 
```
3E 3E //Not Encrypted
```
Response Status | Code
---|---
? | ?

## SEND CRYPTO
This is a service that exists on the BTC Raida. It allows all the other RAIDA to command it to send coins. 

```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
CT CT CT//Coin type
LG LG //Length of the private key part that is stored in the private key aarea below. 
TX TX TX TX TX TX TX TX TX TX TX TX TX TX TX TX  //Transaction number
TX TX TX TX TX TX TX TX TX TX TX TX TX TX TX TX
AM AM AM AM AM AM AM AM AM AM AM AM AM AM AM AM  //The amount of BTC that is expected to fidd
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD // 32-35 bytes. seller's cryptocurrency address to send BTC to.
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD
KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY // RAIDA's part of the private key. The size is 92 bytes fixed but the length of usable bytes is set by the (LG LG) above.
KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY 
KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY
KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME //Memo - 80 Characcters fixed 
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME 
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME
3E 3E //Not Encrypted
```

Sample Response: 
```
3E 3E //Not Encrypted
```
Response Status | Code
---|---
? | Command recieved

<!--![Convert](zips/exchange.png) 

## CHECK FOR PAYMENT
* The RAIDA will make a call to a block explore for the transaction supplied by the client
* RAIDA checks the data of the transactoin. If it is too old it is rejected.
* RAIDA checks the list of recetn transactions and makes sure tokens have not been isssued yet.
* RAIDA moves coins from the sellers For Sale locker into the client's locker. 

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
Not enough market coins| 182
Price differnt more than 1% | 181
Address did not fit allowable format | 198
```
 //Empty 
3E 3E 
```


# Withdraw from Depository
* The user must first put the coins that they want to sell into a locker.
* The client must check with the exchange rate web API and decide when to convert. 
* The client will tell the RAIDA what price it would like to buy at (based on the exchange rate)
* The RAIDA will then check the price with the same API, if the prices are within 1% of each other, the transaction shall be made. 
* The Administrators must have a market maker account with enough currency in it to handle the transaction
* The RAIDA will tell the Crypto Wallet (Multisig) to transfer the money to the account.
* First come first serve with the market account to keep it simple. If the market account runs out conversion stops.
* There must be some way to roll back the transaction if they crypto part fails. 

The user sends:
* The cryptocurrency-code that they want to convert into (See table of crypto currencies)
* The converstion cost they expect to pay
* Their wallet address
* The receipt number (not required )
* The memo (up to 1300 bytes allows for 20KB on 16 RAIDAs)

Sample Request
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK LK //Locker key with CloudCoins
CD CD CD //currency code to convert to
$$ $$ $$ $$ $$ $$ $$ $$ $$//Converstion cost expected
AS // Address size
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD //Target cryptocurrency address
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD
AD AD AD
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID  //The receipt ID 
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME
...
ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME ME //up to 1300 bytes of memo data. Optional.
3E 3E //Not Encrypted
```


Response Status | Code
---|---
Success | 250
Not enough market coins| 182
Price differnt more than 1% | 181
Address did not fit allowable format | 198

```
PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN PN  // P

 //Empty 
3E 3E 

Command Code | Type | Swap | Used By | Command | Link
---|---|---|---|---|---
113 | PythonRA | BTC To WEST | Buyer | [TriggerTransaction](#trigger-transaction) | Requests a RAIDA server to send a crypto transaction to a remote wallet
114 | PythonRA | BTC To WEST | Buyer | [GetRate](#get-rate) | Gets exchange rate for the client (This has been moved to the RAIDAX Proxy on the Treasurer's Workstation). RAIDAX calls service #114 on PythonRAIDA
115 | PythonRA | BTC To WEST | Buyer | [WatchForTransaction](#watch-for-transaction) | Checks if a transaction is confirmed on the Blockchain(This has been moved to the RAIDAX Proxy on the Treasurer's Workstation).


## REST Services Running on an Inforation Server
Some services do not need Data Supremacy and are located on tradition servers. They are acccessed using cutomary REST calls. These REST services are provided by a thrid part. The API can be found at [Postman](https://documenter.getpostman.com/view/16362858/UVXokDS6)

Method | Name | Description
---|---|---
GET | Get supported currencies | Returns a list of currencies that can be swapped
GET | Get rate | Shows the exchange rate the user must take. 
GET | Get transaction status | Tells the user what is happening with their transaction
GET | Get Transactions list | Shows a list of transactions that the user has performed
GET | Validate address | Allows the user to make sure the address is good
-->
