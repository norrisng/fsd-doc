# Requesting a METAR

Requests are prefixed with `$AX`:

```
$AX(callsign):SERVER:METAR:(ICAO airport code)
```



**Server response:**

```
$ARserver:(callsign):METAR:(METAR contents)
```



**Note:** METAR requests are handled by the server. They are not sent to other clients.