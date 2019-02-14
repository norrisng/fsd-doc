# Preface #

## Preamble ##

FSD is an open-source protocol first written in the 90s by Marty Bochane for SATCO (now VATSIM). Today's online networks, VATSIM and IVAO, have since made their own modifications to it. Owing to their shared history, many of them are shared, though it seems that VATSIM has made more. In any case, neither have publicly shared them, as they are locked down by NDAs imposed by the respective networks.

The actual protocol in use by both networks seems to depart quite a bit from the original documentation. It is unclear as to why basic functions (e.g. adding/removing clients) have been completely redefined using completely different "keywords".

There are various sections involving what appears to be two-way authentication. The mechanism behind them is unknown, and is therefore undocumented below. It is likely that such sections are the main focus of network-specific NDAs.

For all intents and purposes, IVAO's FSD implementation can be considered a subset of VATSIM's. VATSIM-unique implementations are indicated as appropriate.

This document is licensed under the GPL.

*Disclaimer:*

This is an unofficial document. No NDAs were signed in during its making. The protocol was reverse-engineered by inspecting incoming/outgoing packets during the course of a regular network connection. No security measures were broken; the protocol is in plaintext, and the details of authentication measures are not documented (nor were the measures themselves reverse engineered).



## A technical rant ##

> *As for FSD, it's a pile of shit...it's basically a really REALLY badly implemented ircd.*
>
> \- Chris Collins, XSquawkbox developer

Perhaps this is a bit of rant on the protocol itself (but not the clients), but I still think it's probably important to note the following.

There are multiple areas where the FSD protocol (both original and VATSIM/IVAO-modified) breaks modern software engineering paradigms:
* Packets are in what is essentially an inconsistent comma-separated format.
* Packet definitions are inconsistent. Their use is also inconsistent.
* Lack of a separation of concerns. Much of the information managed by FSD could be better served over a RESTful API and/or stored in a RDBMS, including:
  * Flight plans
  * ATIS information
  * METARs
  * User registration
* Passwords are stored in a text file, in plain text, on the server named `cert.txt` (it is not a relational database). According to VATSIM staff, their implementation (which they call CERT) is encrypted, but not hashed.
* Irrelevant packets are sent to clients
  * ATC stations receive aircraft state data (e.g. state of control surfaces, landing gear etc.)

  * Packets not requiring client response (e.g. addressed to other clients) are sent by the server




## License ##

This work is licensed under the GPL.



# Introduction #

## Definitions ##

| Term                            | Definition                                                   |
| ------------------------------- | ------------------------------------------------------------ |
| Callsign                        | The unique identifier that a client is logged in as; e.g. `ACA8`, `EGLL_TWR`, `XX_SUP`, etc. |
| Client                          | A pilot or ATC station. For the purposes of FSD, observers are considered to be ATC stations. |
| Client string                   | A string identifying the client software. Think of it as the FSD equivalent of user agent strings for browsers. Examples include `Euroscope 3.2`, `vPilot`, etc. |
| Frequency (freq)                | The network analogue to VHF frequencies used for aircraft radio communication. For the purposes of FSD, observers are always tuned to **199.998 MHz**. |
| Latitude (lat), longitude (lon) | Latitude and longitude. They have a precision of 5 decimal places. |
| Network ID                      | A unique ID used by the network. For VATSIM, it is the CID; for IVAO, the VID. |
| Rating                          | The ATC or pilot rating, as assigned by the network. This is stored as an integer; the lowest rating is `1`. Examples: S1/C2/C3, AS3/ADC, FS3/SPP, etc. |
| Server                          | A FSD server.                                                |
| Packet                          |                                                              |

Placeholders are indicated using brackets; e.g. `(client string)`.



### Ratings ###

Each user has a pilot and ATC rating. This is stored as an integer inside `cert.txt` (see "A technical rant"), but is usually advertised under a different name:

**ATC ratings**

| `cert.txt` value | VATSIM rating | IVAO rating |
| ---------------- | ------------- | ----------- |
| 1                | OBS           | AS1         |
| 2                | S1            |             |
| 3                | S3            |             |
| 5                | SUP           |             |

**Pilot ratings**

| `cert.txt` value | VATSIM rating | IVAO rating |
| ---------------- | ------------- | ----------- |
| 1                | No rating     |             |
| 11               | P1            |             |
|                  |               |             |





## High-level overview ##

The protocol essentially follows this workflow:

1. Connect to the FSD server. 
2. If it is a legitimate* server, provide login details.
3. Provide position updates every 5 seconds.
4. Respond to any requests for information. Requests may come from the server itself, or from other online clients.
5. Request information as needed.
6. Receive information on other clients, even if it's not needed.
7. Repeat (3) through (6) *ad infinitum*.
8. Disconnect by telling the server that you're disconnecting.



\* "Legitimate" refers to the network that the client is meant to connect to. For instance, a client meant for VATSIM (e.g. vPilot) will not connect to an IVAO FSD server, and vice versa.



# Packet format #

Information is sent and received using FSD-specific packets. Most packets roughly follow the format below:
```
command : destination : source : data <CRLF>
```
`command`
The name of the command. See each section for the commands.

Each command is prefixed with one of the following:
Symbol | Details
---|---
`$` | Requests and responses 
`#` | Adding/removing clients, text messages 
`%` | ATC update 
`@` | Aircraft update 


`destination`
The destination of the packet. This may be any one of the following. Case does not seem to matter:
* DATA
* SERVER
* FP
* The callsign of an online station / aircraft

The first three appear to be VATSIM-specific.

`source`
The sender of the packet. See `destination` for valid formats.

`data`
The content of the packet. See further for valid formats.

**Note:**
* Spaces are shown for clarity. They are not present in the actual packets, unless they are part of a user-entered field (e.g. flightplans, PMs).

* `<CRLF>` is a Windows-style line break (`\r\n`).

  





# Initial connection #

## Identification ##

After the initial ACK handshake, the client and server identify themselves using the `$DI` and `$ID` prefix respectively. The server identifies itself first.

Each server appears to contain a list of clients that are allowed to connect.

**Server:**

```
$DISERVER:CLIENT:VATSIM FSD V3.13:(token)
```
`(token)` is a 22-character hexadecimal string that changes on every login. The mechanism behind its generation is unknown.

**Client:**
```
$ID(callsign):SERVER:(token):(client string):3:2:(network ID):(num)
```
`(token)` is a 4-character hexadecimal string. The mechanism behind its generation is unknown.

`(num)` is a 9-digit number. Each user has a unique value. It is unclear if it is linked to the client's hardware.



## Login ##

First, login information is presented to the server:

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
It also requests the client's capabilities...
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



# Server heartbeat #

The server will broadcast a heartbeat to all connected clients every 30 seconds:

```
#DLSERVER:*:0:0
```





# Challenge-response authentication (VATSIM) #

The server will, at unknown intervals, sends an authentication challenge (`$ZC`):
```
$ZCSERVER:(callsign):ddf2fff485f5d24feab4
```
The client must respond with the expected response (`$ZR`), within a predefined amount of time. Clients are booted from the network if this time limit is exceeded:
```
$ZR(callsign):SERVER:fb70e2a04f648555fcb47ddb3b98694a
```
The mechanism of this authentication (i.e. the secret and/or hash function) is unknown.

According to VATSIM forum threads, it appears that this feature may have been added in 2006.

IVAO servers do not appear to utilize this form of authentication.



# Adding and removing clients #

While the presence (or absence) of client update packets (see "Positon updates" under "Requesting and sending information") will signify the presence (or lack thereof) of a client, the server will also send a special packet signaling the addition/removal of said client, whether this is due to client disconnecting or falling out of visibility range.



## Adding clients ##

It is unknown as to what `100` represents, but it appears that all clients share the same value.

### ATC ###

```
#AA(callsign):SERVER:(real name):(network ID)::(rating):100
```

### Pilots ###

```
#AP(callsign):SERVER:(network ID)::1:100:(rating):(real name ICAO)
```

It is unclear as to what the `1` represents. It could possibly represent

## Removing clients ##

### ATC ###

```
#DA(callsign):(network ID)
```

### Pilots ###

```
#DP(callsign):(network ID)
```



# Requesting and sending information #

## METAR - `$AX` ##

Requests are prefixed with `$AX`:
```
$AX(callsign):SERVER:METAR:(ICAO airport code)
```
### Server response ###
```
$ARserver:(callsign):METAR:(METAR contents)
```
Note: METAR requests are handled by the server. They are not sent to other clients.



## Other online users - `$CQ`, `$CR` ##

All requests are prefaced with `$CQ`.  A client may send or receive requests:
```
$CQ(requester):(requestee):(command)
```
Responses follow the format below:
```
$CR(requestee):(requester):(command):(data)
```



(unless otherwise specified, the formatted fields below contain only the `(data)` field)



### Commands  ###

#### `ACC` - Aircraft configuration ####

`ACC` data comes in for all aircraft within visibility range.

Configuration data is formatted in JSON.

**Request:**

```json
{"request":"full"} 
```

*All `ACC` requests contain the exact JSON string specified above.*



**Response:**

For some reason, `ACC` responses are prefixed with `$CQ`, and *not* `$CR` (as would be expected for a response):

```json
{
    "config":{
        "is_full_data":true,
        "lights":{
            "strobe_on":false,
            "landing_on":false,
            "taxi_on":true,
            "beacon_on":true,
            "nav_on":true,
            "logo_on":false
        },
        "engines":{
            "1":{
                "on":true
            },
            "2":{
                "on":true
            }
        },
        "gear_down":false,
        "flaps_pct":0,
        "spoilers_out":false,
        "on_ground":true
    }
}
```

*Line breaks have been inserted for readability. They are not present in the actual packets.*

JSON fields may be omitted as desired.



#### `ATIS` - ATIS

Each controller has an "ATIS" field associated with it. This field is essentially a user-defined information box. It does not contain the airfield ATIS, unless it belongs to an ATIS station (see below).

Actual airfield ATISes are implemented by means of an additional online station with the callsign `ICAO_ATIS`. Its ATIS field contains the actual ATIS. A computer-generated voice is broadcast on Its voice channel to simulate voice ATIS.

**Request:**

```
$CQ(requester):(requestee):ATIS
```

**Response:**

```
$CR(requestee):(requester):ATIS:V:(voice server address, without "http://")
<...>
$CR(requestee):(requester):ATIS:T:(line of ATIS text)
<...>
$CR(requestee):(requester):ATIS:E:(total number of lines, including this line)
```





#### `CAPS` - Client capabilities ####

This a




#### `RN`	 -  Real name ####

`Kim Jong-il ZKPY::1`



### Server-only commands ###

The following commands are only sent/received by the server. The sending/receiving station is represented as `DATA` or `SERVER`:



#### `INF` - Client system information ####

**Request:**

`$CQDATA:(recipient):INF `

**Response:**
```
#TM(callsign):DATA:(client string) PID=(CID) ((Real name ICAO)) IP=(IP address) SYS_UID=-270030784 FSVER=(sim)  LT=(lat) LO=(lon) AL=(alt) 
```
* `SYS_UID` is a 9-digit number. It appears that it can be a negative value.
* `FSVER` represents the flight simulator being used (e.g. `Prepar3dV3`). It is left blank (but not omitted) if the client is ATC.
* `LA`,`LO`, and `AL` appear to be floating-point numbers



### Text messages - `#TM` ###

All messages are prefixed with `#TM`:

```
#TM(sender):(recipient):(message contents)
```

If `(sender)` and `(recipient)` are online callsigns, then this command acts as a private message.

If `(recipient)` is of the format `@xxyyy`, then this command sends the message contents over the frequency **1xx.yyy MHz**.

VATSIM appears to correctly escape message contents, as including colons in message text does not seem to cause unexpected behaviour.

However, this does not seem to be the case in IVAO. To correctly escape colons, use `::` as the escape sequence.

If `(sender)` and `(recipient)` are not online callsigns, then the text message represents additional background functionality that the average user does not see or deal with. It's used by VATSIM to hack in additional functionality (see below), without making major modifications to the protocol.



#### Receipt of flight plan (VATSIM) ####

```
#TM(own callsign):FP:(flightplan callsign) GET
```

Once the client receives a flight plan, it shall immediately notify `FP` of such. The reason for this is unknown.

The server will then acknowledge with the following:

```
#PCserver:(own callsign):CCP:BC:(flightplan callsign):0
```





#### Server messages ####

```
#TMserver:(callsign):(message)
```

These are essentially text messages that are sent from a callsign named `server`.

These are usually seen on initial login.



### Flight plans - `$FP` ###

A pilot may file a flight plan as follows:

```
$FP(callsign):*A:(VFR/IFR):(acft type):(spd):(origin):(sched dep):(sched dep):(alt):(dest):(EET):(FOB):(alt airport):(remarks)
```

| Placeholder   | details                                                     |
| ------------- | ----------------------------------------------------------- |
| `(VFR/IFR)`   | `V` or `I`                                                  |
| `(acft type`) | includes FAA prefixes/suffixes, if present; e.g. `H/B77W/L` |
| `(spd)`       | Ground speed, in knots                                      |
| `(sched dep)` | Scheduled departure; e.g. ` 2215` or 10:15pm Z              |
| `(EET)`       | Estimated enroute time; e.g. `1:37`                         |
| `(FOB)`       | Fuel on board; e.g. `2:30`                                  |

*IVAO's format likely differs, but this has not been investigated in detail.*



## Position updates ##

Position updates are sent every 5 seconds.



### Aircraft - `@S` ###

Aircraft updates are prefixed with`@S`:

```
@S:(callsign):(squawk):(rating):(lat):(lon):9:0:4282385248:5 
```



### ATC - `%` ###

ATC updates are prefixed with `%`:

```
%(callsign):(freq):(altitude):100:1:(lat):(lon)
```

ATC stations and observers also have a location and altitude associated with them.



# Logging off #

To log off, send one of the following packets below.

## ATC - `#DA` ##

```
#DA(callsign):SERVER
```

## Aircraft - `#DP` ##

```
#DP(callsign):(network ID)
```

