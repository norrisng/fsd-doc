# Position updates

Position updates are sent every 5 seconds.



## Pilots ##

Aircraft updates (both outgoing and incoming) are prefixed with `@S` or `@N`. The distinction between the two is unclear, though outgoing updates are always use `@S`:

```
@S:(callsign):(squawk):(rating):(lat):(lon):(alt):(groundspeed):(num1):(num2) 
```

```
@N:(callsign):(squawk):(rating):(lat):(lon):(alt):(groundspeed):(num1):(num2) 
```

A squawk code of `7500` (aircraft hijacking) will result in an immediate disconnection from the server.

The meaning of the following fields is unclear:

* `(num1)` is a 10-digit number.  It seems to begin with 4.
* `(num2)` is an integer. There seems to be a wide range of values, anywhere between 1 and 3 digits, and can be positive or negative.



## ATC ##

ATC updates are prefixed with `%`:

```
%(callsign):(freq):(altitude):(protocol version):1:(lat):(lon)
```

ATC stations and observers also have a location and altitude associated with them, even though they do not move throughout a session.

