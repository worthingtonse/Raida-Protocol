# Instant messaging via the RAIDAX
For Phase I, we will use the Matrix messaging protocol. The Matrix server will be on one of the RAIDAX servers. 
We may have the Matrix service use a hard drive that rediredts to the RAIDAX storage so that the saved content is 
spread over all the RAIDAX servers and secure. The RAIDAX protocol will be used like a VPN to provide a quantum safe transport to the Matrix Service
located on a RAIDAX server. 

We would load balance the messaging requests. Every RAIDAX would have a Matrix service running on it. 
Everytime the Client sends a message, they will send it to a different RAIDA. The client could figure out the closest RAIDAX (in miliseconds) 
to use for speed sake. The client would then need to listen for all 25 RAIDAX to send it messages. 
This would balance the load and make it more secure. 

For Phase II we could get into the Matix's protocol, extract the message and stripe it to all 25 RAIDAX. 


## RAIDAX Header Requests:
To make this work, the RAIDAX Header Request would need to use some bytes that we have not used yet. 
1. The two "Port Number" bytes (also called the Application Bytes index 8 and 9) would have a code for INSTANT_MESSAGE.
   
## Software Components
1. The client will be a standard Matrix client that will send and receive messages throught the Go Client.
2. The Server will be a standard Matrix server that will send and receive messages to the RAIDAX server. 
3. The RAIDAX protcol will be used to provide a VPN through the Go client and RAIDAX server. 

## Workflow: 
1. The Matrix client sends a REST request to the local Go Client (or perhaps a new Python Client).
2. We would use HTTP instead of HTTPS.
3. The Go Client calculates the nearest RAIDAX in ms, encapulates the Matrix request in the body and sends it.
4. The RAIDAX server receives the request, sees that it is an instant message and unencapsolates the body.
5. RAIDAX looks at it's configutation file to see what port the Matrix Server is listing on.
6. The RAIDAX server sends the body to the Matrix server.
7. The RAIDAX must listens for requests from the Matrix server that will be sent to the client.
8. The RAIDAX then encapsulates the Matrix server's message and sends it to the client.
9. The client gets the response or a new message and unencrypts and unencapulates it.
10. The Go Client puts the message together and sends it to the Matix client on the same machine. 


## Sample encoding for Phase II
We would need to explore if there is a way to compress the matrix request.

