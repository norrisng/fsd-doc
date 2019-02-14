# Background #

FSD is a protocol first written in the 90s by Marty Bochane for SATCO (now VATSIM). Today's online networks, VATSIM and IVAO, have since made their own modifications to it. Owing to their shared history, many of them are shared, though it seems that VATSIM has made more. In any case, neither have publicly shared them, as they are locked down by NDAs imposed by the respective networks.

The original FSD source code can be found [here on GitHub](https://github.com/kuroneko/fsd).

The actual protocol in use by both networks seems to depart quite a bit from the original documentation. It is unclear as to why basic functions (e.g. adding/removing clients) have been completely redefined using completely different "keywords".

There are various sections involving what appears to be two-way authentication. The mechanism behind them is unknown, and is therefore undocumented below. It is likely that such sections are the main focus of network-specific NDAs.

For all intents and purposes, IVAO's FSD implementation can be considered a subset of VATSIM's. VATSIM-unique implementations are indicated as appropriate.



# What this document covers #

This document covers all aspects of FSD that are in plaintext.

In the case of VATSIM, this excludes server/client identification hashes, as well as the challenge-response authentication used by the network.



# Comparison with other modern protocols #

> *As for FSD, it's a pile of shit...it's basically a really REALLY badly implemented ircd.*
>
> \- **Chris Collins**, XSquawkbox developer ([source](https://www.reddit.com/r/flightsim/comments/82t5yn/vatsim_practices/dvemjes/))

There are multiple areas where the FSD protocol (both original and VATSIM/IVAO-modified) breaks modern software engineering paradigms:

* Packets are in what is essentially an inconsistent comma-separated format.
* Packet definitions are inconsistent. Their use is also inconsistent.
* Lack of separation of concerns. Much of the information managed by FSD could be better served over a RESTful API and/or stored in a RDBMS, such as:
    - Flight plans
    - ATIS information
    - METARs
    - User registration
* Passwords are stored in a text file, in plain text, on the server named `cert.txt` (it is not a relational database). [According to VATSIM staff](https://forums.vatsim.net/viewtopic.php?p=525158#p525158), their implementation (which they call CERT) is encrypted, though it is likely not hashed due to the way authentication works in FSD.
* Irrelevant packets are sent to clients:
    * ATC stations receive aircraft state data (e.g. state of control surfaces, landing gear etc.)
    * Packets not requiring client response (e.g. addressed to other clients) are sent by the server
