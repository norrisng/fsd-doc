# Other packets #

## Server heartbeat

The server will broadcast a heartbeat to all connected clients every 30 seconds:

```
#DLSERVER:*:0:0
```



## Challenge-response authentication (VATSIM)

The server will, at unknown intervals, sends an authentication challenge (`$ZC`):

```
$ZCSERVER:(callsign):(hash)
```

`(hash)` is a hexadecimal string, with a length of anywhere between 8 and 22 digits (inclusive).

The client must respond with the expected response (`$ZR`), within a predefined amount of time. Clients are booted from the network if this time limit is exceeded:

```
$ZR(callsign):SERVER:(hash)
```

`(hash)` is a 32-digit hexadecimal string.

The mechanism of this authentication (i.e. the secret and/or hash function) is unknown.

According to VATSIM forum threads, it appears that this feature may have been added in 2006.

IVAO servers do not appear to utilize this form of authentication.

## Kick User (VATSIM)

Kicking a user requires a client to send a specific kick packet prefixed with `$!!` to the server, along with the callsign to be kicked and a kick reason as parameters. The packet will not truly kick the user unless the kick-initiating client is logged in with an ATC rating at or above Supervisor. Kicking a user is usually initiated by using the `.kill` command found in many ATC and pilot clients.

```
$!!(callsign):(target callsign):(reason)
```

`(target callsign)` is the callsign that will be kicked from the network.

`(reason)` is the reason for the kick. This will be displayed to the user being kicked before being disconnected.
