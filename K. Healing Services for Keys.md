# Healing Services For Keys (Command Group 4)

These services allow the RAIDA Client to fix encryption keys that are fracked with a RAIDA to support quantum safe encryption. (Fracked means that a minority of RAIDA think the key is counterfeit)


Command Code | Service 
--- | --- 
44 | [Get Encryption Ticket](#post-an-part) 
45 | [Fix Encryption](#post-an-parts) 




## GET ENCRYPTION TICKET

The two services "Get Encryption TIcket" and "Fix Encryption" are used to exchange keys between clients who have shared secrets with RAIDA servers but not a specific
RAIDA server. This will establish a shared secret with a fracked RAIDA so fixes can be done. 

These services are only needed if there are no tokens that can be used for encryption between a RAIDA and client but the same token does work with at least two other RAIDA. 

Then the fix keys process can run. This process uses the two services in this "RAIDA Key Service" that are designed just for this purpose. 

The client uses the "Encrypt RAIDA Key" service and sends key parts to two "helper" RAIDA that are not fracked (They can be successfully encrypted). The helper RAIDA are told which RAIDA the key parts are for so they can encrypt the key parts using a common secret between the Helper RAIDA and the Fracked RAIDA. 

For example, if the key is fracked on RAIDA 5, the client would randomly choose a key 5000 through 5999 and ask the helper RAIDA to use that. If the key was fracked on RAIDA 23, the client would pick a number between 23000 and 23999.

The key parts are actually the AN of the fracked token. 

The client will recieve two encrypted key parts from the two different RAIDA. The client sent an 8 byte key but receives a 16 byte key in the response. 

The the client then uses these two key parts and calls the "Post RAIDA Key" service. The call to the Post RAIDA Keys is unencrypted. But the key parts embedded in the request are encrypted by the other RAIDAs so called "id keys". 

The Fracked RAIDA can then decrypt the key parts using the common secret between them and the helping RAIDA. The key is actually the AN of the key ID that they client provides and so the AN of that token is changed accordinly so now encryption will work with the token provided. 


Client wants to talk to RAIDA 11. However, RAIDA 11 lost all its data and the Client and RAIDA 11 have no shared secret.  RAIDA 5 is "Helper RAIDA 1" and RAIDA 12 is "Helper RAIDA 2". The Client has shared secrets with Helper RAIDA 1 and 2 (tokens). 

Step | Actor | Action | Notes
---|---|---|---
1 | Client | Gets the AN from the token that has failed the validation challenge. This is the AN that the Fracked RAIDA should have but does not. | This is a GUID
2 | Client | Chooses 2 random RAIDA (who do pass the validation challenge) to talk to | In this case Helper RAIDA 1 and Helper RAIDA 2 are chosen.
3 | Client | Client breaks the AN from the fracked token into 2 parts (Each 8 bytes long) | Parts are indexed as part 0 and part 1
4 | Client | Encrypts the request including the DN, SN and AN of the token on Fracked RAIDA to be fixed along with part 0 usings Helper RAIDA 1's key  | This is the AN for the token that we want to fix on Fracked RAIDA.
5 | Client | Encrypts part 1 using Helper RAIDA 2's key | Again, We will use the encrypt DN SN SN SN SN of the token we will fix on Fracked RAIDA.
6 | Client | Calls the [Encrypt Key](RAIDA%20Key%20Tickets.md#post-raida-key) service on Helper RAIDA 1 and Helper RAIDA 2 | Tells them to encrypt using Fracked RAIDA's key
7 | Helper RAIDA 1 and 2 | They unencrypt the request using Client's shared secret | This is the token in the request header. We want to fix this on Fracked RAIDA.  
8 | Helper RAIDA 1 and 2 | Add padding and encrypt the part using Fracked RAIDA's key | Five bytes are from the token in the request header's encryption part. 
9 | Helper RAIDA 1 and 2 | Encrypt the response using Client's key and return to Client | Now Client has unreadable "tickets"
10 | Client | Does not encrypt the body of the [Post Key](RAIDA%20Key%20Services.md#post-raida-key) request to Fracked RAIDA | Denomination and Serial Numbers are clear text
11 | Fracked RAIDA | Fracked RAIDA decrypts the messages using Helper RAIDA 1's and Helper RAIDA 2's Key | Must already have shared secrets with other RAIDA
12 | Fracked RAIDA | Puts the key parts together in the order the were received | First keypart is part 0. Second is part 1
13 | Fracked RAIDA | Fracked RAIDA puts this key into the AN of the token specified | Now Client and the fracked RAIDA have at least one shared secret
14 | Fracked RAIDA | Fracked RAIDA uses the key to encrypt the response | Responds to Client
15 | Client | Client decrypts the response and checks the challenge | 
16 | Client | If the challenge is good, Alice changes the AN on the token she specifies to the key  | Not neccessar if the AN has not changed. Now Client has a common secret with the fracked RAIDA
17 | Client | Client can now make any request she likes to Bob using this token | Key exchange done
18 🔴| Fracked RAIDA | For additional security, the Fracked RAIDA should tell its token Manager to pown the key that was used in the encryption. 


Note: Clients know that the each RAIDA has 🔴 1000 tokens on their 0 network and the client can pick one at random. 
Sample Request for Get Encryption Ticket:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH  //Challenge
DN SN SN SN SN //The ID of who the key is for. // The Denomination, The Serial Number of the Bob's encryption token (1-25,999) depending on Bob's RAIDA ID. 
KY KY KY KY KY KY KY KY //Eight byte key part. 
DN SN SN SN SN //The serial number of the token that will be used to create a shared secret between Alice and Bob (Fracked token on Bob)
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN //The AN of the token used to create a shared secret between Alica and Bob (Fracked token on Bob)
E3 E3  //Not Encrypted

```

The RKE will take 8 byte key parts and then add 5 bytes of the token to be fixed (DN SN SN SN SN), 2 random bytes to the end. 
Then it adds one know byte to the end: 0xFF. 
This creates a 16 byte block that can be encrypted effectivly and padds it as needed. 
Then, the RKE server will use the encryption key that the client specified to encrypt the 16 bytes. 
The RKE returns the encrypted 16 bytes to the user.

The nonce for encryption is the same as the nonce sent in the request header.

Response Status Codes
Code | Status | Details
---|---|---
0 | Fail | Failed but no details provided
1| Success | Key ticket returned successfuly
11 | Offline | Service temporarilty offline 

Sample Response Body
```
D798010F14C64102AC3EB6D728166054
E3 E3 //Not Encrypted
```

## FIX ENCRYPTION
The "Post Key" service recieves a group of tickets encrypted by other RAIDAs. It uses its Authenticity Numbers of its own ID keys to decrypt them. Then it assembles the keys that it is able to encrypt a response to the caller. It checks that each key ends with a "0xFF"and removes the last eight bytes that the client does not know about. It Ignores keys it is unable to encrypt and then respond to the caller with:

* Which key parts it used to create a master key.

* 🔴A SHA-256 hash of the key.

* Seems the Nonce in the header should be the same as the ones used in the "Encrypt Key" requests.

Sample Request for Fix Encryption. Note: This is all unencrypted:
```hex
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH  //Challenge
DN SN SN SN SN //The serial number of the fracked token. 
CO CO SP DA SH DN SN SN SN SN KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY //token to encrypt and key part. 
CO CO SP DA SH DN SN SN SN SN KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY //token to encrypt and key part.  
.....
CO CO SP DA SH DN SN SN SN SN KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY KY //token to encrypt and key part. 
E3 E3  //Not Encrypted
```

Meaning of Codes:
Code | Meaning
---|---
CH | Challenge
CO | token ID
SP | Split ID (Zero if no splits)
DA | Detection Agent (RAIDA ID)
SH | Shard ID
DN | Denomination 
SN | Serial Number
KY | Key
E3 | Ending bytes


The service will then decypt the keys using the keys that it owns. 
It looks for the 0xFF at the end to make sure the token is whole and true. 
It removes the token DN and SN SN SN SN bytes and make sure they are the same for both key parts. 

Then it responds to the client with a hash of the key it has created. 

RAIDA Changes the AN in its ANs table for the token specified in the key parts. 

Sample Respons:
```hex
HS HS HS HS HS HS HS HS HS HS HS HS HS HS HS HS //SHA-256 hash of the Master key or is it SHA-128?
KA //Key accepted status. Either 0x00 for not accepted or 0x01 for accepted. 
KA
---
KA //one for every key the client proposed
E3 E3  //Not Encrypted
```
