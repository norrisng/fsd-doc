# Contributing #

*Subject to change without prior notice*.



## Poking around with FSD ##

Much of the poking around can be done in Wireshark. [See this blog post](https://simfed.org/blog/2014/10/10/wiresharking-vatsim-and-ivao/) on how to do that.

If you want to directly "poke" a FSD server by pretending to be a client and actually communicate with it, you can use `telnet`:

```bash
$ telnet (server address) 6809
Trying (server address)...
Connected to (server address).
Escape character is '^]'.
$DISERVER:CLIENT:VATSIM FSD V3.13:a93e2926340a
```

*Notes:*

*  `telnet` is a command-line utility included in Linux and Mac OSX. On Windows, you'll need to enable it first ([see here](https://social.technet.microsoft.com/wiki/contents/articles/38433.windows-10-enabling-telnet-client.aspx)) before you can use it the command prompt. The syntax is the same, however.
* The homepage of the documentation (see GitHub repo page for link) contains information on where to find a list of FSD server addresses for IVAO/VATSIM.

Once connected, simply type in correctly-formatted FSD packets and press `Enter` to send them.



## Markdown conventions ##

### Section headers ###

Section headers should be surrounded with `#`s on both sides, with a space between the text and the `#`s:

```markdown
## Section header ##
```

### Code blocks ###

Don't forget to indicate the language after the 3 backticks.

Example:

```
​```bash
(content)
​```
```

Do **not** indicate the language for blocks containing FSD packets.

Yes:

```
​```
#AA(callsign):SERVER:(full name):(ID):(password):1:100 
​```
```

No:

```
​```fsd
#AA(callsign):SERVER:(full name):(ID):(password):1:100 
​```
```



## Submitting changes ##

Please open a GitHub pull request in the repository, and indicate the changes you've made in the title/comments. 

If you're not sure what exactly you want to change (and therefore don't want to submit a pull request), create an issue on the **Issues** page.