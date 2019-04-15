# Welcome! #

Welcome to the unofficial documentation of FSD, the protocol behind the [IVAO](http://ivao.aero/) and [VATSIM](http://vatsim.net/) online networks. 

This documentation was created during the reverse engineering process for [FCOM](https://github.com/norrisng/FcomClient/), which forwards messages to a web server. [FCOM's implementation of this server](https://github.com/norrisng/FcomServer/) forwards it over Discord DM, but it's entirely possible to write a server that does anything it wants (which may or may not involve forwarding) with the forwarded messages.

**Note:** information may change as new information regarding FSD is discovered. This document is currently heavily focused on VATSIM's implementation (which is much "heavier" than IVAO's), but this will change in the future as I figure out which parts are unique to either network.



## Generating the documentation ##

The documentation is written in Markdown.

To generate webpages with this documentation, use the Python tool [MkDocs](https://www.mkdocs.org/). It can be installed via `pip`:

```bash
pip install mkdocs
cd fsd-doc
```



### Preview ###

To generate a live "preview" version that auto-refreshes whenever one of the Markdown files is updated:

```bash
mkdocs serve
```



### Building the site

To finalize everything into HTML pages:

```bash
mkdocs build
```





## Watching FSD in action ##

To see FSD in action, use Wireshark to monitor traffic on TCP port 6809. [Here's a tutorial](https://simfed.org/blog/2014/10/10/wiresharking-vatsim-and-ivao/) to get started.

As is the case with anything on the [application layer](https://en.wikipedia.org/wiki/Application_layer) that's in plaintext, FSD can be interacted with over Telnet:

```bash
telnet (server address) 6809
```

IVAO uses `eu*.ivan.ivao.aero` (replace `*` with the server number). A list of servers can be found at https://heartbeat.ivao.aero/ .

VATSIM uses static IPs. These IPs can be found at any of the URLs starting with `url1` at https://status.vatsim.net/. As of the time of writing, two mirrors are available ([hardern.net](http://vatsim-data.hardern.net/vatsim-servers.txt), [vroute.net](http://info.vroute.net/vatsim-servers.txt))



## Contributing ##

Have something you'd like to add to the documentation? Just fork and create a pull request on the [GitHub repository](https://github.com/norrisng/fsd-doc/). 
