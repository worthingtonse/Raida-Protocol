<!--![Reserve Locker](zips/convert.png)-->
# Selling CloudCoin Services

Allows a CloudCoin seller to post coins to sell. Allows CC Buyers to send crypto and then receive CloudCoins. 

## Commands

Command Code | Service | Description | Used By 
---|---|---|---
🔴85 | [Put For Sale](#put_for_sale)| Puts coins into a locker that are marked to be sold. | Seller


## PUT FOR SALE
This is exactly as the PUT command (from Locker Services) except the Authenticicy Number is different. Only the first eight bytes of the AN are random. The next five bytes reprednt the price that the coins are to be soldfor with the first two bytes being for the whole part of the number and the last three bytes being for the fraction part of the number. There is then one byte for the coin type and two bytes to mark it as a coin for sale. These bites are EE EE. 

Coin Types

Number | Coin
---|---
00 | CloudCoin
01 | BTC (Bitcoin)
02 | XRM (Monero)


The seller will post the price of each of their CloudCoins in the currency that is used to purchase it. 

The price is encoded in the first two bytes in the last two bytes of the AN (Locker Key). 

The smallest price that CloudCoins can be sold for is $.001. The most that CloudCoins can be sold for are $65.535 All sales must be in increments of $.001. The two bytes represent the number of $.001 times that value given. 

If a person wanted to sell their coins for $1.12 then the last four bytes of the AN would be "04 60 FF FF". 
This is derived by $1.12/$.001 = 1120. Converted to hex: "04 60" then add two 0xFF for a completed last four bytes as "04 60 FF FF". 

So if the first two bytes are "8D E1" (36,321 in Decimal), then the price that the coins are to be sold is $.001 * 36321 = $36.321 dollars.

Note that when a person wants to PEEK or REMOVE their coins that are for sale, they must also include the price. 

People can post many lockers for sale, each with a different price. 

Sample PUT FOR SALE Request:
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN 
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU //The sum of all the ANs of the SNs. See POWN SUM. 
LN LN LN LN LN LN LN LN $. $. .$ .$ .$ CT EE EE  //Locker number, whole price number, fraction price number, coin type, for sale marker
## ## // Number of bytes in the seller's address
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD  //Seller's crypto Wallet address. 128 Bytes fixed. 
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD  //The amount of these bytes used depends on the ## ## above.
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD  
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD  
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD
AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD AD 
3E 3E  //Not Encrypted
```
Response Status | Code
---|---
All Pass | 241
All Fail | 242

<!--
There will need to be a table to track the AN/Cryptocurrency/price/address. 

Coins For Sale Table: 

Symbol | Column Name | Bytes | Description
---|---|---|---
CT | Cryptocurrency Code | 2 | From the coin table. Like BTC or Monero
PN | AuthenticityNumber |  16 | The PN included in the request used as the locker ID.
TC | Total Amount of Coins | 4 | The total amount of coins in the locker*
$$ | Price in Dollars | 4 Float | Smallest allowed is $.001
AD | Seller's Cryptocurrency Address | 128 | Fixed size but use will vary according to coin type. BTC is around 64?
LK | Not locked = 00, Locked before sale = 01, Locked after sale = 02 // Used to lock the locker so that it cannot be viewed or sold. 
LT | Locked Time | 4 | the date-time that the record was locked. If the lock time is too long, the record should be unlocked. 


*Unlike other lockers, sales lockers must have all the coins removed at the same time. Users cannot just remove a few coins from a sales locker but must remove all of them at once time. Also, users are not able to add more coins to the forsale locker once it is created.

When records are deleted from this table, they will be moved to the "Past Transactions Table" that can be written to the hard drive. 
-->

