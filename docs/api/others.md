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

