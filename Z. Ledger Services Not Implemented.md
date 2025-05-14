# Administrator Service (Command Group 10)
 Administrator services allow the Token Admins to manage users and groups.
 
 Command Code | Service
 ---|---
 100 | [Create User](#new_user)
 101 | [Read User](#read_user)
 102 | [Update User](#update_user)
 103 | [Delete User](#delete_user)
 104 | [Create Group](#new_user)
 105 | [Read Group](#read_user)
 106 | [Update Group](#update_user)
 107 | [Delete Group](#delete_user)
108 | [Create Group_User](#new_group_user)
109 | [Delete Group_User](#delete_group_user)

 

### Client Actions
Here are the steps that the client should do to send tokens to a locker:
1. Calculate if you need to make change (break tokens into smaller denominations).
2. Call "Get Available Change SNs" if you need change.
3. Call "Break" using the SNs if you need change.
4. Fix any known fracks according to the "pown" string on the file name.
5. Call Store Sum on the tokens that will be sent. 
7. Find Any Lost tokens if there is no response. 
8. Fix any Fracked tokens.
9. Tell the receiver the tokens have been sent and give them the code. 
10. Perhaps make change for the next transaction. (Change Management)


Step 5 in detail:
1. Generate a seven character seed. This seed should be random and use only 31 characters. [abcdefghjkmnpqrstuvwxyz23456789] 
2. Put a RAIDA ID in front of each seed and so that there are now 25 seeds. If the seed was YU78HNP then you would have 0YU78HNP, 1YU78HNP, 2YU78HNP..24YU78HNP 
3. Create an MD5 hash of each seed. One for each RAIDA. 
4. Change the last four byte of the MD5 hash so that they are all 0xFF.
5. Add up the sum of all the ANs of the tokens you want to put in the locker. The totals will when they get too big.
6. Pick a token randomly from the bank to use for encryption. 
7. Make the "Store Sum" request. 

### Receiver Actions
10. Get the code from the sender. 
11. Call peek and get all the serial numbers and denominations
12. Calculate the total there. 
13. POWN the tokens
14. Find Lost
15. Fix Fracked. 
16. Give change by calling Store Sum and giving the change to the user. 


## PUT

Store is the exact same thing as POWN SUM but instead of adding a sum, a single PAN is used for all the tokens. This PAN becomes the "Locker Number". However, to make it clear that the PAN is a locker number, the last four bytes must be set to all ones (0xFF) by the client. 



Sample Store Request:
```c
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN  SN SN SN SN 
DN  SN SN SN SN  
DN  SN SN SN SN  
DN  SN SN SN SN  
SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU SU //The sum of all the ANs of the SNs. See POWN SUM. 
PN PN PN PN PN PN PN PN PN PN PN PN FF FF FF FF  //This is the PAN for all the tokens. The last four bytes must be set to binary ones (0xFF) 
3E 3E  //Not Encrypted
```

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

Returns all the Serial Numbers that were changed. 

Sample Response
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
MS //Mixed content bitfield. Maybe many bytes. One byte for 8 tokens. 0 is fail, 1 is pass. 
3E 3E  //Not Encrypted
```



