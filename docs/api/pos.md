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

### Now-Clear 'Unknown' fields answered by phoudoin ###

> (num1) = pitch / bank / heading / onground value
> a decimal sent as string whose value is the following unsigned 32bits binary number:
> 
> ```
>  3         2         1         0
> 10987654321098765432109876543210
> <  pitch ><  bank  ><heading >OU
> ```
> 
> * **U** bit:
>   
>   * unused
> * **O** bit:
>   
>   * Aircraft onground indicator.
> * **heading** 10bits field:
>   
>   * aircraft current heading, in degree
>   * values range [0, 1023] (unsigned 10 bits number)
>   * 1 = 360/1024 degree
>   * 0 = magnetic north
> * **bank** 10bits field:
>   
>   * aircraft current bank / roll angle, in degree
>   * values range [-511, 512] (signed 10 bits number),
>   * 1 = 180/512 degree,
>   * negative value = right wing banking.
> * **pitch** 10bits field:
>   
>   * aircraft current pitch angle, in degree
>   * values range [-511, 512] (signed 10 bits number)
>   * 1 = 90/256 degree
>   * 0 = aircraft is at level
>   * 256 = aircraft pitch is 90 degree up (aka skyrocket!)
>   * negative value = down pitch
> 
> Onground aircrafts have usually 0 degree pitch and bank angles, their (num1) value is mostly heading encoded, a small value, very stable until they start taxiing.
> 
> On the contrary, airborne aircrafts have rarely perfect 0 degree pitch and banking angles, leading to big (num1) values.

> (num2) = difference between Above Ground Level altitude (AGL) and Above Mean Sea Level (AMSL) altitude, as reported by aircraft barometric altimeter
> 
> Unit is feet, obviously.

## ATC ##

ATC updates are prefixed with `%`:

```
%(callsign):(freq):(altitude):(protocol version):(rating):(lat):(lon)
```

ATC stations and observers also have a location and altitude associated with them, even though they do not move throughout a session.

