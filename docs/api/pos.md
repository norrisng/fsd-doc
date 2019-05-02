# Position updates

Position updates are sent every 5 seconds.



## Pilots ##

Pilot position updates are prefixed with `@`:

```
@(mode):(callsign):(squawk):(rating):(lat):(lon):(alt):(groundspeed):(num1):(num2) 
```

### Transponder mode ###

`(mode)` refers to the transponder mode. It consists of a single letter:

| Mode | Description  |
| ---- | ------------ |
| S    | Standby      |
| N    | Mode C       |
| Y    | Squawk ident |

Regardless of transponder mode, the altitude is always included in a position update.

### Squawk code ###

A squawk code of `7500` (aircraft hijacking) will result in an immediate disconnection from the server.

### Unknown fields ###

The meaning of the following fields is unclear:

* `(num1)` is a number of up to 10 digits.  It seems to begin with 4.
* `(num2)` is an integer. There seems to be a wide range of values, anywhere between 1 and 3 digits, and can be positive or negative.



## ATC ##

ATC updates are prefixed with `%`:

```
%(callsign):(freq):(altitude):(protocol version):(rating):(lat):(lon)
```

ATC stations and observers also have a location and altitude associated with them, even though they do not move throughout a session.

