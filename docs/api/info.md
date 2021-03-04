# Sending and receiving information

All requests are prefaced with `$CQ`.  A client may send or receive requests:

```
$CQ(requester):(requestee):(command)
```

Responses follow the format below:

```
$CR(requestee):(requester):(command):(data)
```





## Commands ##

### `ACC` - Aircraft configuration (VATSIM only) ###

Returns the current configuration (i.e. flaps, landing gear, lights etc.) of the requested callsign's aircraft. Requests and responses are formatted in JSON.

`ACC` data is received for all aircraft within visibility range, even if not requested.

It may also be sent by a client uninitiated at any time (e.g. if the user extends/retracts flaps). In such a case, the `ACC` packet is addressed to `@94836`.

IVAO does not appear to have an equivalent command.

**Note:** the formatted fields below contain only the `(data)` field.



**Request:**

```json
{"request":"full"}
```

*All `ACC` requests from other users contain this exact JSON string .*



**Sample response:**

For some reason, `ACC` responses are prefixed with `$CQ`, and **not** `$CR` (as would be expected for a response):

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

If the `ACC` packet is being sent by the client uninitiated, then JSON fields may be omitted as desired.



### `ATIS` - ATIS ###

Returns the requested callsign's voice server URL and ATIS message. This is only applicable for ATC stations.

Each controller has an "ATIS" field associated with it. This field is essentially a user-defined information box. It does not contain the airfield ATIS, unless it belongs to an ATIS station (see below).

On VATSIM, actual airfield ATISes are implemented by means of an additional online station with the callsign `ICAO_ATIS`. Its ATIS field contains the actual ATIS, and a computer-generated voice is broadcast on Its voice channel to simulate voice ATIS.

**Request:**

```
$CQ(requester):(requestee):ATIS
```

**Response:**

```
$CR(requestee):(requester):ATIS:V:(voice server address, without "http://")
...
$CR(requestee):(requester):ATIS:T:(line of ATIS text)
$CR(requestee):(requester):ATIS:T:(line of ATIS text)
...
$CR(requestee):(requester):ATIS:E:(total number of lines, including this line)
```





### `C?` - Tuned frequency [IVAO only] ###

Returns the radio frequency the user is tuned to.

**Request:**

```
$CQ(requestee):(requester):C?
```

**Response:**

```
$CR(requestee):(requester):C?:(frequency)
```

The frequency is formatted as `xxx.yyy` (e.g. 122.800)



### `CAPS` - Client capabilities ###

Returns a list of the client's capabilities. It is unclear as to what these capabilities are, as pilots and ATC stations appear to return the exact same values.

Both clients and `SERVER` will request CAPS information.

**Request:**

```
$CQ(requestee):(requester):CAPS
```

**Response:**

```
$CR(requestee):(requester):CAPS:ATCINFO=1:SECPOS=1:MODELDESC=1:ONGOINGCOORD=1
```





### `RN`	 -  Real name ###

Returns the real name of the user logged in as the requested callsign.

**Request:**

```
$CQ(requester):(requestee):RN
```

**Response:**

ATC:

```
$CR(requestee):(requester):RN:(real name):(ATC sector file):(rating)
```

Pilot:

```
$CR(requestee):(requester):RN:(real name ICAO)::(rating)
```



## Server-only commands ##

The following commands are only sent/received by the server. The sending/receiving station is represented as `DATA` or `SERVER`:



### `INF` - Client system information ###

**Request:**

```
$CQDATA:(recipient):INF
```



**Response:**

```
#TM(callsign):DATA:(client string) PID=(CID) ((Real name ICAO)) IP=(IP address) SYS_UID=(uid) FSVER=(sim) LT=(lat) LO=(lon) AL=(alt)
```

* `SYS_UID` is a 9-digit number. It appears to be a negative value.
* `FSVER` represents the flight simulator being used (e.g. `Prepar3dV3`). It is left blank (but not omitted) if the client is ATC.
* `LA`,`LO`, and `AL` appear to be floating-point numbers



## Text messages ##

All messages are prefixed with `#TM`:

```
#TM(sender):(recipient):(message contents)
```

If `(sender)` and `(recipient)` are online callsigns, then this command acts as a private message.

### Special recipients ###

The following recipients are treated differently:

| Recipient | Meaning                                                      |
| --------- | ------------------------------------------------------------ |
| `@xxyyy`  | Sends the message over the frequency **1xx.yyy MHz**, instead of PM. |
| `*`       | Broadcast message, received by everyone on the network. <br /><br />IVAO and VATSIM restrict this functionality to users with the supervisor rating and above. When an underqualified user attempts to send a broadcast message, the server replies with a syntax error `$ER` packet. |

If `(sender)` and `(recipient)` are not online callsigns, then the text message represents additional background functionality that the average user does not see or deal with. It is used by VATSIM to hack in additional functionality (see below), without making major modifications to the protocol.



### Escaping of message contents ###

VATSIM appears to correctly escape message contents, as including colons in message text does not seem to cause unexpected behaviour.

However, this does not seem to be the case in IVAO. To correctly escape colons, use `::` as the escape sequence.





### Flight plan acknowledgement (VATSIM) ###

```
#TM(own callsign):FP:(flightplan callsign) GET
```

Once the client receives a flight plan, it shall acknowledge it using the above command. The reason for this is unknown.

The server will then acknowledge with the following:

```
#PCserver:(own callsign):CCP:BC:(flightplan callsign):0
```





### Server messages ###

```
#TMserver:(callsign):(message)
```

These are essentially text messages that are sent from a callsign named `server`.

These are usually seen on initial login.



## Pilot flight plans ##

A pilot may file a flight plan as follows:



### IVAO ###

IVAO's flightplan format is more detailed, and is based on the [ICAO international flight plan [PDF, page 2]](https://www.faa.gov/documentLibrary/media/Form/FAA_7233-4_PRA_07-31-2017.pdf).

Please refer to [IVAO documents [PDF]](https://www.ivao.aero/training/documentation/books/SPP_ADC_Flightplan_Understanding.pdf) on what to include in each field.

```
$FP(callsign):SERVER:(VFR/IFR):(acft):(spd):(origin):(sched dep):1000:(crz alt):(dest):(EET):(FOB):0:0::(remarks):(alt airport):S:0:
```

| Placeholder   | details                                                      |
| ------------- | ------------------------------------------------------------ |
| `(VFR/IFR)`   | `V` or `I`                                                   |
| `(acft)`      | Aircraft and equipment, in the format `#/(model)/(turb)-(equip)/(xpdr)` (e.g. `1/B738/M-SDE2E3FGHIRWXY/LB1`). Please refer to the [IVAO documents [PDF]](https://www.ivao.aero/training/documentation/books/SPP_ADC_Flightplan_Understanding.pdf) for further details. |
| `(spd)`       | Cruising airspeed, in knots. Prefixed with `K` (km/h) `N` (knots), or `M` (Mach). `K` /`N` are always 4-digit, and `M` is 3 digits without the decimal (e.g. M082 = Mach 0.82) |
| `(sched dep)` | Scheduled departure; e.g. ` 2215` or 10:15pm Z               |
| `(crz alt)`   | Cruising level. Prefixed with `F` (flight level in feet), `S` (metric flight level in x10 metres), `A` (altitude in x100 feet), `M` (altitude in x10 metres), or `VFR` (VFR flight with no specific altitude) |
| `(EET)`       | Estimated enroute time; e.g. `1:37`                          |
| `(FOB)`       | Fuel on board; e.g. `2:30`                                   |

*Further clarification is required on some fields.*




### VATSIM ###

VATSIM's flightplan format is simpler, and is based on the [FAA domestic flight plan [PDF]](https://www.faa.gov/documentLibrary/media/Form/FAA_Form_7233-1_7_31_17.pdf).

```
$FP(callsign):*A:(VFR/IFR):(acft type):(spd):(origin):(sched dep):(sched dep):(alt):(dest):(EET):(FOB):(alt airport):(remarks):(route)
```

| Placeholder   | Details                                                     |
| ------------- | ----------------------------------------------------------- |
| `(VFR/IFR)`   | `V` or `I`                                                  |
| `(acft type)` | Includes FAA prefixes/suffixes, if present; e.g. `H/B77W/L` |
| `(spd)`       | Ground speed, in knots                                      |
| `(sched dep)` | Scheduled departure; e.g. ` 2215` or 10:15pm Z              |
| `(EET)`       | Estimated enroute time; e.g. `1:37`                         |
| `(FOB)`       | Fuel on board; e.g. `2:30`                                  |


Flight plan packets are sent to all other clients on the server, even if not requested.

On VATSIM, once such a packet has been received, the client acknowledges it (see "Receipt of flight plan (VATSIM)" under "Text messages" on this page).

## ATC flight plans (VATSIM) ##

On VATSIM, an ATC client may edit an aircraft's flight plan as follows:

### Flightplan amendment ###

The flightplan amendment packet _does not include scratchpads or squawk assignments_. For those 2 separate packets, see the next 2 sections.

```
$AM(atc callsign):SERVER:(target callsign):(VFR/IFR):(acft type):(spd):(origin):(sched dep):(sched arr):(cruise alt):(destination):(num1):(num2):(num3):(num4):(alternate):(remarks):(route)
```

| Placeholder         | Details                                                             |
| ------------------- | ------------------------------------------------------------------- |
| `(atc callsign)`    | The callsign of the ATC client who creates/sends the amendment packet. |
| `(target callsign)` | The callsign that's being given the amended flightplan.             |
| `(VFR/IFR)`         | `V` or `I`                                                          |
| `(acft type`)       | Includes FAA prefixes/suffixes, if present; e.g. `H/B77W/L`         |
| `(spd)`             | Ground speed, in knots                                              |
| `(sched dep)`       | Scheduled departure; e.g. ` 2215` or 10:15pm Z                      |
| `(num1)` - `(num4)` | Unknown. This most likely contains EET, FOB, and 2 other fields.    |

### Squawk assignments ###

The following packets are sent to the server when an ATC client assigns a transponder code to an aircraft:

```
$CQ(callsign):@94835:BC:(target callsign):(code)
```

`(target callsign)` is the callsign that's being given the new squawk assignment.

`(code)` is the 4 digit transponder code being assigned to `(target callsign)`

```
#TM(callsign):FP:(target callsign) SET (code)
```

This next packet prefixed with `#TM` seems to be sent directly after the first squawk assignment packet is sent. The recipient `FP` may be an automatic service on the network that could handle automatic tasks, like statistics.

### ATC scratchpads ###

The following packet is sent to the server when an ATC client sets a scratchpad for an aircraft:

```
$CQ(callsign):@94835:SC:(target callsign):(scratchpad)
```

`(target callsign)` is the callsign that's being given the new scratchpad.

`(scratchpad)` is the new scratchpad string being assigned to `(target callsign)`. This can include uppercase and lowercase characters, along with numbers. The maximum length of this string that VATSIM seems to allow/support is 4 characters.
