# CHAT
Same as: [https://gitlab.shurafom.eu/worthingtonse/super-client/-/edit/master/chat.md](https://gitlab.shurafom.eu/worthingtonse/super-client/-/edit/master/chat.md)

The RAIDA will need to have three services added: POST MESSAGE, GET MESSAGE, 
POLL MESSAGES. 

Command Code | Service 
--- | --- 
173 | [POST MESSAGE](#post-message)
174 | [GET MESSAGE](#get-message)
172 | [POLL MESSAGE](#poll-message)

The request headers will be exactly the same as is used today. And so too is the response headers. 
The bodies will be different. It is assumed (at the moment) 
that both sender and receiver are identified by their IdCoins. 

## POST MESSAGE 
(Command Code - 173 (0xAD))

POST Body Structure is a fixed fields:

NOTE: IX (Index of stripe) - The client breaks the data into many stipes and each stripe is labled so the receiver knows how to put it back together

```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN SN SN SN SN // SN of the sender 🔴
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN // AN of the ID coin
DN SN TO TO TO // SN of the receiver 🔴
ID ID ID ID // Message ID
IX IX // Index of stripe, server keeps and returns this but does not process
TS TS TS TS // Timestamp (UNIX Epoch, seconds)
MS MS MS .. // Message body (variable length, up to 1400 bytes)
E3 E3  // EOM Marker

```
Response codes: "Success" (250) or "Message ID already exists" (171). The latter means that it not possible to send same Message ID to the same receiver twice or more.

## GET MESSAGE

(Command Code - 174 (0xAE))
Downloads up to 1400 bytes of messages.

Body Structure for three messages
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN OW OW OW OW // SN of the receiver (changed) 🔴
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN  // AN of the ID coin
ID ID ID ID // First message ID
ID ID ID ID // Second message ID
ID ID ID ID // Third message ID
        // === These fields may refer to Get_V2 (future use) ===
        // RW // Number of rows * 100. So 0x01 means return 100 rows. 00 Means max rows. (future use)
        // YR // The Year  00 = 2000. 255 = 2255. Time must be in UTC (future use)
        // MM // Month (future use)
        // DY // Day of Month (future use)
        // RT // Return Code. 00=stripe 01=mirror, 11 = 2nd mirror, FF Stripe, Mirror and 2nd Mirror (future use)
        // ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID ID // Reserved for Future Use. Put random numbers. (future use)
E3 E3  // EOM Marker

```
The service will respond with up to 1400 bytes of data. Messages are packed into a single request without any separators. Each response is prepended with a block of fixed-length headers (13 bytes - SN(3) + ID(4) + TS(4) + IX(2)). Length of each message body is known to client from the POLL request. 

Messages are returned in the same order as they were requested.

If client requests a handful of messages which do not fit into a single frame, "Chat: Requested Messages Too Long" (172) is returned. For three messages it means that sum of bodies lengths should not be greater than 1400 - 13 * 3 = 1361 bytes

If message with given Message ID could not be retrieved (ex. it has been already downloaded), it is skipped.

```
NM NM // Number of messages
SN SN SN ID ID ID ID TS TS TS TS IX IX // SN of the sender ("FROM") + Message ID + TimeStamp + Index of stripe
SN SN SN ID ID ID ID TS TS TS TS IX IX // Second message header
SN SN SN ID ID ID ID TS TS TS TS IX IX // Third message header
MS MS MS .. // First message body (variable length)
MS MS MS .. // Second message body
MS MS MS .. // Third message body
E3 E3  // EOM Marker
```
Response codes: "AllPass" (241) if all requested messages were sent, "AllFail" (242) if none (no response body as well), "Mixed" (243) if only some. For "AllPass" and "Mixed" total number of actually returned messages is returned in bytes 0-1 (big-endian) of response body.


## POLL MESSAGS
(Command Code - 172 (0xAC))

This service does need authentication, user must not look into other's users inboxes

Request Header:
```
CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH CH
DN TO TO TO TO // SN of the receiver (Changed) 🔴
AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN AN // AN of the ID coin
```
Response codes: "Success" (250) if there are messages, "Fail" (251) if there are no messages, "FailToAuth" (64) if receiver's IdCoin is counterfeit

"Success" response body for three pending messages (MAX - 100). Message IDs are sorted by timestamp in ascending order (oldest first) 
```
ID ID ID ID FR FR FR ML ML // First message ID + Sender + Message Length
ID ID ID ID FR FR FR ML ML // Second message ID + Sender + length
ID ID ID ID FR FR FR ML ML // Third message ID + Sender + length
E3 E3  // EOM Marker
```
