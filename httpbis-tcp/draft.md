---
title: TCP Performance Tuning for HTTP
abbrev: TCP for HTTP
docname: draft-stenberg-httpbis-tcp-00
date: 2015
category: bcp

ipr: trust200902
area: General
workgroup: httpbis
keyword: Internet-Draft

stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, comments, inline]

author:
 -
    ins: D. Stenberg
    name: Daniel Stenberg
    organization: Mozilla
    email: daniel@haxx.se
    uri: http://daniel.haxx.se

normative:
  RFC2119:
  RFC7230:
  RFC0793:
  RFC7540:

informative:
  RFC7430:
  RFC6928:
  RFC0896:

--- abstract

This document records current best practice for using all versions of HTTP over TCP.

--- middle

# Introduction

HTTP version 1.1 {{RFC7230}} as well as HTTP version 2 {{RFC7540}} are defined
to use TCP {{RFC0793}}, and their performance can depend greatly upon how TCP
is configured. This document records best current practice for using HTTP over
TCP, with a focus on improving end-user perceived performance.

These practices are generally applicable to HTTP/1 as well as HTTP/2, although
some may note particular impact or nuance regarding a particular protocol
version.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in {{RFC2119}}.


# TCP Fast Open

TCP Fast Open (a.k.a. TFO, {{RFC7430}}) allows data to be sent on the TCP
handshake, thereby allowing a request to be sent without any delay if a
connection is not open.

TFO requires both client and server support, and additionally requires
application knowledge, because the data sent on the SYN cannot be
non-idempotent. Therefore, TFO can only be used on idempotent, safe methods
(e.g., GET and HEAD), or with intervening negotiation (e.g, using TLS).

Support for TFO is growing in client platforms, especially mobile, due to the
significant performance advantage it gives.

# Initial Congestion Window

{{RFC6928}} specifies an initial congestion window of 10, and is now fairly
widely deployed server-side. There has been experimentation with larger
initial windows, in combination with packet pacing.

# Packet Pacing

TBD


# Explicit Congestion Control

Apple [deploying in iOS and OSX](https://developer.apple.com/videos/wwdc/2015/?id=719).


# Tail Loss Probes

[draft](http://tools.ietf.org/html/draft-dukkipati-tcpm-tcp-loss-probe-01)

# Slow Start after Idle

Slow-start is one of the algorithms that TCP uses to control congestion inside
the network. It is also known as the exponential growth phase. Each TCP
connection will start off in slow-start but will also go back to slow-start
after a certain amount of idle time.

In Linux systems you can prevent the TCP stack from going back to slow-start
after idle by settting

    net.ipv4.tcp_slow_start_after_idle = 0

# Nagle's Algorithm

Nagle's Algorithm {{RFC0896}} is the mechanism that makes the TCP stack hold
(small) outgoing packets for a short period of time so that it can
potentionally merge that packet with the next outgoing one. It is optmized for
throughput at the expence of latency.

HTTP/2 in particular requires that the client can send a packet back fast even
during transfers that are perceived as single direction transfers. Even small
delays in those sends can cause a significant performance loss.

HTTP/1.1 is also affected, especially when sending off a full request in a
single write() system call.

In POSIX systems you switch it off like this:

    int one = 1;
    setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof(one));


# Half-close

Client or server is free to half-close after a request or response has been
completed; or when there is no pending stream in HTTP/2.

Half-closing is sometimes the only way for a server to make sure it closes
down connections cleanly so that it doesn't accept more requests while still
allowing clients to receive the ongoing responses.

# Abort

No client abort for HTTP/1.1 after the request body has been sent. Delayed
full close is expected following an error response to avoid RST on the client.

# Keep-alive

TCP keep-alive is likely disabled - at least on mobile clients for energy
saving purposes. App-level keep-alive is then required for long-lived requests
to detect failed peers or connections reset by stateful firewalls etc.

# TCP-Bound Authentications

TBD

# Closing Idle Connections

TBD

# IANA Considerations

This document does not require action from IANA.

# Security Considerations


--- back
