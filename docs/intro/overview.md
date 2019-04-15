# A high-level overview #



## The purpose of packets ##

Information is transmitted between clients and the server by means of **packets**. A packet is a single line of specially-formatted text that conforms to the FSD protocol.

It serves a similar purpose to XML or JSON, in the sense that they also involve data and/or state transfer. However, keep in mind that FSD precedes either of these standards.



## Client workflow ##

### Connecting to the server ###

1. Connect to the server.
2. If the server is legitimate, the client identifies itself to the server.*
3. Provide login details.

\* *Only applies to VATSIM. "Legitimate" refers to the network that the client is meant to connect to. For instance, a client meant for VATSIM (e.g. vPilot) will not connect to a non-VATSIM FSD server.*



### Send and receive data ###

* Provide position updates every 5 seconds.
* Receive packets, from either the server or other clients.
* If received packets indicate a request, respond to them.



### Disconnecting ###

1. The client informs the server over FSD that it's disconnecting.
2. Close the connection.