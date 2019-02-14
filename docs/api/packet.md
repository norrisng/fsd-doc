# Overview #

Most packets roughly follow the format below:

```
(command):(destination):(source):(data)\r\n
```


## `command` ##
The name of the command. See each section for the commands.

Each command is prefixed with one of the following:

| Symbol | Details                                |
| ------ | -------------------------------------- |
| `$`    | Requests and responses                 |
| `#`    | Adding/removing clients, text messages |
| `%`    | ATC update                             |
| `@`    | Aircraft update                        |


## `destination` ##

The destination of the packet. This may be any one of the following. Case does not seem to matter:

* `DATA`
* `SERVER`
* `FP`
* The callsign of an online station / aircraft

The first three appear to be VATSIM-specific.


## `source` ##

The sender of the packet. See `destination` for valid formats.



## `data` ##

The content of the packet. See the following sections for valid formats.


