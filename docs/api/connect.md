# Connecting to a FSD server

## Pre-login identification (VATSIM only) ##

**Note:** IVAO does not implement this form of identification. The client initiates communication after the TCP connection is established (see the "Login" section).

### Server ###

Once the connection is established, the server identifies itself using the `$DI` prefix:

```
$DISERVER:CLIENT:VATSIM FSD V3.13:(token)
```

`(token)` is a 22-character hexadecimal string that changes on every login. The mechanism behind its generation is unknown.



### Client ###

The client must then identify itself using the `$ID` prefix before continuing:

```
$ID(callsign):SERVER:(client id):(client string):3:2:(network ID):(num)
```

| Field             | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `(callsign)`      | The callsign to log in as.                                   |
| `(client id)`     | A 4-character hexadecimal string that identifies the client. See next subsection for details. |
| `(client string)` | A human-readable string that identifies the client in use.   |
| `(network ID)`    | VATSIM CID.                                                  |
| `(num)`           | A user-unique signed 9-digit number. It is unclear if it is linked to the client's hardware. |



### Client ID and string ###

VATSIM servers appear to contain a whitelist of client IDs. Connections using non-whitelisted client IDs will be closed by the server:

```
$ERserver:unknown:016::Unauthorized client software
```

The client string is not used by the server to identify the client software. It is merely a human-readable string for readability purposes.

The following are a list of known client IDs:

Client ID | Client string
--------- | -------------
`69d7` | EuroScope 3.2
`88e4` | vPilot
`48e2` | Swift
`de1e` | VRC
`d8f2` | xPilot



## Login ##

The format varies slightly between pilots and ATC:

### ATC ###

```
#AA(callsign):SERVER:(full name):(network ID):(password):(num1):(protocol version)
```

The meaning of the following fields is unclear:

| Field    | Examples  | Possible meaning |
| -------- | --------- | ---------------- |
| `(num1)` | `1`, `12` | ATC rating       |



### Pilot ###

```
#AP(callsign):SERVER:(network ID):(password):(num1):(protocol version):(num2):(full name ICAO)
```

The meaning of the following fields is unclear:

| Field    | Examples   | Possible meaning |
| -------- | ---------- | ---------------- |
| `(num1)` | `1`, `11`  | Pilot rating     |
| `(num2)` | `2`, `30`  | Unknown          |





## Post-login ##

### Message of the Day ###

If the login is successful, the server returns a pre-defined message, encoded as private messages sent from `SERVER`.

#### IVAO ####

```
#TMSERVER:(callsign):IVAO FSD Server B.06k - (c)1996-2016 Filip Jonckers, Kenny Moens, Ken Andries - limited license, usage granted to International Virtual Aviation Organisation VZW
#TMSERVER:(callsign):Welcome to the IVAO - Europe (a) - Network Server
#TMSERVER:(callsign):Please visit www.ivao.aero for further info.
#TMSERVER:(callsign):You are now online and connected, have fun.
#TMSERVER:(callsign):You are (b)
```

| Field | Information                                                  |
| ----- | ------------------------------------------------------------ |
| `(a)` | IVAO server number (e.g. `4` for EU4)                        |
| `(b)` | User's ATC or Pilot rating; e.g. `Pilot Rating FS3`, `ATC Rating AS3` |



#### VATSIM ####

```
#TMserver:(callsign):Welcome back to VATSIM! Our new voice technology is now live - if you have any questions or need support visit vats.im/audio for required downloads, instructions and tutorial videos. If you need further support visit the Audio for VATSIM forums at vats.im/afv
```



### Post-login setup ###

#### IVAO ####

The client then provides information to the server about itself:

```
!C(callsign):SERVER:(rating):(num1):(client):(client version):(num2)
```

| Field              | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `(callsign)`       | User's callsign.                                             |
| `(rating)`         | User's rating, as an integer (see "Terms Used" under "Introduction") |
| `(num1)`           | *Unknown.*                                                   |
| `(client)`         | Name of the client; e.g. `IvAp`.                             |
| `(client version)` | Client version; e.g. `2.0.2`                                 |
| `(num2)`           | *Unknown.*                                                   |

The server replies with some information about the client:

```
!RSERVER:(callsign):(rating):(num1):(rating):(num2):(client IP address)
```

The meaning of `(num1)` and `(num2)` is unclear. They are different from `(num1)` and `(num2)` in the packet sent before this response.



##### ATC

Prior to the above-mentioned `!C` packet, ATC clients provide an additional packet on the controller's airfield:

```
&D(callsign):SERVER:0:(airport ICAO)
```

##### Pilot #####

After the `!C` packet, the client sends some additional info:

```
$NV(callsign):SERVER:(callsign):1
-PD(callsign):SERVER:(MTL model name)
-MD(callsign):SERVER:13
&R(callsign):SERVER:0
-RF(callsign):SERVER:3:3
```

The meaning of the trailing numbers is unclear.



#### VATSIM ####

The server requests information on the client's capabilities:

##### ATC #####

Server request:

```
$CQSERVER:(callsign):CAPS
$CRSERVER:(callsign):ATC:N:(callsign)
$CRSERVER:(callsign):CAPS:ATCINFO=1:SECPOS=1
$CRSERVER:(callsign):IP:(client IP address)
```

The `N` on the second line *may* indicate the server indicating that the client is not providing ATC services.

Client response:

```
$CR(callsign):SERVER:CAPS:ATCINFO=1:SECPOS=1:MODELDESC=1:ONGOINGCOORD=1
```



##### Pilot #####

Server request:

```
$CQSERVER:(callsign):CAPS
$CRSERVER:(callsign):IP:(client IP address)
$ERserver:(callsign):008:(callsign):No flightplan
```

Client response:

```
$CR(callsign):SERVER:CAPS:VERSION=1:ATCINFO=1:MODELDESC=1:ACCONFIG=1
```
