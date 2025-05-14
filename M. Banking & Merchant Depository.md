# Creating depositories for Banks and merchants to dollarize
The goals of these services are to allow people to deposit money into banks.
And allow people to prove that the deposit took place. 


## Command Codes
Anyone can put coins in. Only the owner of the account can move the out to a locker. 
Command Code | Service | Description
--- | --- | :---: 
86 | 🔴[Payment](#) | People put money into a special depository & receive an invoice. 
87 | 🔴[See Invoice](#) | This receipt id allows anyone to see an invoice about what happened. 
88 | 🔴[Confirm Payment](#confirm-payment) | Confirms that a payment has been made and marks the payment as confirmed.
89 | 🔴[Show Statement](#show-statement) | Shows the owner of the Depository all the history of transactions. 
89 | 🔴[Move to Locker](#move-to-locker) | Orders tokens to be destroyed. 

## PAYMENT 

This is the exact same thing as the Locker service PUT but it:
1.  Deposits coins into a Depository and not a locker.
2.  Includes a transaction statement.
3.  Includes a transaction GUID.
4.  Includes the senders "Owner's Key" which must match should match the owner's key used to encrypt the message unless the depository allows it.
5.  The PAN is actually the Depository Number of the merchant. The client must learn about this on its own unless we create a resolution service.
6.  The last four bytes of the PAN will be slightly differt. Instead of FF FF FF FF. It will be FF FF FF F0.

See PUT about how to sum the numbers. 

Sample Put Request
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN 
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU //The sum of all the ANs of the SNs. See POWN SUM. 
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  //This is the PAN for all the tokens. The last four bytes must be set to binary ones (0xFF)
SD SS SS SS SS //  Owners Key. one byte for the denomination and four bytes for the serial number. 
ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID //GUID of the transaction. Sent by the client but the client may get it from a merchant. 
YY MM DD HH mm SS //Client's time stamp. Must be in UTC. 
TF //Transaction meta data format. 0 means stiped on 25 no fault tollerance. 1 means horizontal, vertical parity.  
PC //Privacy code. 0 means cannot be read by anyone but the participantes. 1 Means everyone can read it. 
ST ST ST ST.....//Meta Data Stripe up to 100 bytes. The client may use this to describe the transaction by striping it using a standard format.  
3E 3E  //Not Encrypted
```
This will create a transaction record that can just be stored on the RAIDA hard drive. Each Depository will have its own folder. The Depositories'
folders will have sub folders for "Senders Key" written in hexidecimal. Then each senders folder will have two sub folders called "Confirmed" and "Unconfirmed" and "Invoices" .bin files named after the transaction
GUIDs with an underscore with the amount then the new balance in the Depository then the transaction type (zero for payment), then timestamp. Inside the .bin file is the memo in binary form. Sample files are: 

File names: Transaction GUID, amount, New Balance, meta data format, time stamp and .bin file extention.  
```file
Data/
├─ /
├─ Statements/
│  ├─ AA9AE39A07B145DA938B7113FFFFFFF0/  (Depository ID)
│  ├─ A085C87642B14841A856E69EFFFFFFF0/
│  ├─ EC1D7C261DB54EA8B52D8046FFFFFFF0/
│  │  ├─ 46DE996BC5/   (Sender's key)
│  │  ├─ 13199AD076/
│  │  │  ├─Confirmed
│  │  │  │  ├─ E7FAF12810FD4BABB0E2657C16DEF8E0_3.19032_9000_0_1723449490.bin (Transaction file)
│  │  │  │  ├─ A085C87642B14841A856E69E1CE4112C_140_9150_0_1723449450.bin
│  │  │  │  ├─ AA9AE39A07B145DA938B7113199AD076_.0098_9140.0098_0_172346666.bin
│  │  │  ├─Unconfirmed
│  │  │  ├─Invoices
│  │  │  │  ├─ E7FAF12810FD4BABB0E2657C16DEF8E0_1_1.bin (Invoice file. Transaction GUID along with the invoice format code, privacy code)
│  │  │  │  ├─ A085C87642B14841A856E69E1CE4112C_1_1.bin
│  │  │  │  ├─ AA9AE39A07B145DA938B7113199AD076_1-0.bin

```
Note that Timestamps are in UTC time. 


### PAYMENT with multiple lockers:🔴 // Support for the creation of many lockers has not yet been implemented. THis is for Phase II. 
```c
Same thing as above but there may be multiple depositories specified 
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
