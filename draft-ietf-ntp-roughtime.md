---
title: "Roughtime"
category: info

docname: draft-ietf-ntp-roughtime-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "Network Time Protocols"
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: "Network Time Protocols"
  type: "Working Group"
  github: "wbl/roughtime-draft"

author:
 -
    fullname: "Watson Ladd"
    organization: Akamai Technologies
    email: "watsonbladd@gmail.com"
 -
    fullname: "Marcus Dansarie"
    email: marcus@dansarie.se

normative:

informative:


--- abstract

This document specifies Roughtime - a protocol that aims to achieve rough time synchronization even for clients without any idea of what time it is.


--- middle

# Introduction

Time synchronization is essential to Internet security as many security protocols and other applications require synchronization [@RFC7384] [@MCBG]. Unfortunately widely deployed protocols such as the Network Time Protocol (NTP) [RFC5905] lack essential security features, and even newer protocols like Network Time Security (NTS) [@RFC8915] lack mechanisms to ensure that the servers behave correctly. Furthermore clients may lack even a basic idea of the time, creating bootstrapping problems. Roughtime uses a list of keys and servers to resolve this issue.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Protocol Overview
Roughtime is a protocol for rough time synchronization that enables clients to provide cryptographic proof of server malfeasance. It does so by having responses from servers include a signature over a value derived from a nonce in the client request. This provides cryptographic proof that the timestamp was issued after the server received the client's request. The derived value included in the server's response is the root of a Merkle tree which includes the hash of the client's nonce as the value of one of its leaf nodes. This enables the server to amortize the relatively costly signing operation over a number of client requests. Single server mode: At its most basic level, Roughtime is a one round protocol in which a completely fresh client requests the current time and the server sends a signed response. The response includes a timestamp and a radius used to indicate the server's certainty about the reported time. For example, a radius of 1,000,000 microseconds means the server is absolutely confident that the true time is within one second of the reported time. The server proves freshness of its response as follows. The client's request contains a nonce which the server incorporates into its signed response. The client can verify the server's signatures and - provided that the nonce has sufficient entropy - this proves that the signed response could only have been generated after the nonce.

# The Guarrentee

 A Roughtime server guarantees that a response to a query sent at t1, received at t2, and with timestamp t3 has been created between the transmission of the query and its reception. If t3 is not within that interval, a server inconsistency may be detected and used to impeach the server. The propagation of such a guarantee and its use of type synchronization is discussed in (#integration-into-ntp). No delay attacker may affect this: they may only expand the interval between t1 and t2, or of course stop the measurement in the first place.

# Message Format

Roughtime messages are maps consisting of one or more (tag, value) pairs. They start with a header, which contains the number of pairs, the tags, and value offsets. The header is followed by a message values section which contains the values associated with the tags in the header. Messages MUST be formatted according to Figure TODO as described in the following sections.

Messages MAY be recursive, i.e. the value of a tag can itself be a Roughtime message.

!---
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   Number of pairs (uint32)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
.                                                               .
.                     N-1 offsets (uint32)                      .
.                                                               .
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
.                                                               .
.                        N tags (uint32)                        .
.                                                               .
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
.                                                               .
.                            Values                             .
.                                                               .
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
!---

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

<reference anchor="MCBG" target="https://eprint.iacr.org/2015/1020">
        <front>
          <title>Attacking the Network Time Protocol</title>
          <author initials="A." surname="Malhotra" fullname="A. Malhotra">
            <organization/>
          </author>
          <author initials="I." surname="Cohen" fullname="I. Cohen">
            <organization/>
          </author>
          <author initials="E." surname="Brakke" fullname="E. Brakke">
            <organization/>
          </author>
          <author initials="S." surname="Goldberg" fullname="S. Goldberg">
            <organization/>
          </author>
          <date year="2015"/>
        </front>
</reference>

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
