# Connecting to a FSD server 

## Identification (VATSIM only) ##

After the initial ACK handshake, the client and server identify themselves using the `$DI` and `$ID` prefix respectively. The server identifies itself first.

Each server appears to contain a list of clients that are allowed to connect.

**Note:** IVAO does not implement such a form of identification. The client initiates communication after the TCP connection is established (see the "Login" section).



### Server ###

On initial connection, the server presents the client with the following:

```
$DISERVER:CLIENT:VATSIM FSD V3.13:(token)
```

`(token)` is a 22-character hexadecimal string that changes on every login. The mechanism behind its generation is unknown.

### Client ###

To login to the FSD server, send the following packet:

```
$ID(callsign):SERVER:(client id):(client string):3:2:(network ID):(num)
```

| Field             | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `(callsign)`      | The callsign to log in as.                                   |
| `(client id)`     | A 4-character hexadecimal string that identifies the client. See next subsection for details. |
| `(client string)` | A human-readable string that identifies the client in use.   |
| `(network ID)`    | The user's network ID (i.e. IVAO VID or VATSIM CID)          |
| `(num)`           | A user-unique 9-digit number. It is unclear if it is linked to the client's hardware. |

#### Client ID and string ####

The server contains a whitelist of client IDs. Connections using non-whitelisted client IDs will be closed by the server:

```
$ERserver:unknown:016::Unauthorized client software
```

The client string is *not* used by the server to identify the client software. It is merely a human-readable string for readability purposes.

The following are a list of known client IDs:

| Network | Client ID | Client string |
| ------- | --------- | ------------- |
| VATSIM  | `69d7`    | EuroScope 3.2 |
| VATSIM  | `de1e`    | VRC           |





## Login ##

First, login information is presented to the server. The format varies slightly between pilots and ATC:

**ATC:**

```
#AA(callsign):SERVER:(full name):(ID):(password):1:100 
```

The client then provides its first position update:

```
%(callsign):(frequency):0:100:1:(lat):(lon):0 
```

It also makes the following information requests:

```
$CQ(callsign):SERVER:ATC:(callsign) 
$CQ(callsign):SERVER:CAPS 
$CQ(callsign):SERVER:IP
```

`ATC:(callsign)` *may* represent a request asking if the specified latter callsign is an ATC position.



**Pilot:**

```
#AP(callsign):SERVER:(network ID):(password):1:100:(rating):(full name ICAO)
```

The client then provides its first position update:

```
@S(callsign):(frequency):0:100:1:(lat):(lon):0 
```

&nbsp;

On successful login, the server returns a pre-defined login message as a series of PMs sent from `SERVER`:

```
#TMserver:(callsign):By using your VATSIM assigned identification number on this server you 
#TMserver:(callsign):hereby agree to the terms of the VATSIM Code of Regulations and the 
#TMserver:(callsign):VATSIM User Agreement and the VATSIM Code of Conduct which may be viewed 
#TMserver:(callsign):at http://www.vatsim.net/network/docs/ 
#TMserver:(callsign):All logins are tracked and identification numbers are recorded. 
#TMserver:(callsign):Users must enter their real full first names and surnames when logging 
#TMserver:(callsign):onto any of the VATSIM.net servers. 
```



### ATC-specific ###

The server then requests the client's capabilities...

```
$CQSERVER:(callsign):CAPS
```

...and responds to the client requests from earlier:

```
$CRSERVER:(callsign):ATC:N:(callsign) 
$CRSERVER:(callsign):CAPS:ATCINFO=1:SECPOS=1 
$CRSERVER:(callsign):IP:(client IP address)
```

`ATC:N:(callsign) ` *probably* means something along the lines of "`callsign` is not an ATC position" (the `N` probably means "no").



The client responds to the request for capabilities data:

```
$CR(callsign):SERVER:CAPS:ATCINFO=1:SECPOS=1:MODELDESC=1:ONGOINGCOORD=1
```

