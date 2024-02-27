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

Time synchronization is essential to Internet security as many security protocols and other applications require synchronization {{?RFC738}} [@MCBG]. Unfortunately widely deployed protocols such as the Network Time Protocol (NTP) {{?RFC5905}} lack essential security features, and even newer protocols like Network Time Security (NTS) {{?RFC8915}} lack mechanisms to ensure that the servers behave correctly. Furthermore clients may lack even a basic idea of the time, creating bootstrapping problems. Roughtime uses a list of keys and servers to resolve this issue.


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
## Data types
### int32
An int32 is a 32 bit signed integer. It is serialized least significant byte first in sign-magnitude representation with the sign bit in the most significant bit. The negative zero value (0x80000000) MUST NOT be used and any message with it is syntactically invalid and MUST be ignored.
### uint32
A uint32 is a 32 bit unsigned integer. It is serialized with the least significant byte first.
### uint64
A uint64 is a 64 bit unsigned integer. It is serialized with the least significant byte first.
### Tag
Tags are used to identify values in Roughtime messages. A tag is a uint32 but may also be listed in this document as a sequence of up to four ASCII characters {{!RFC20}}. ASCII strings shorter than four characters can be unambiguously converted to tags by padding them with zero bytes. For example, the ASCII string "NONC" would correspond to the tag 0x434e4f4e and "PAD" would correspond to 0x00444150. Note that when encoded into a message the ASCII values will be in the natural bytewise order.
### Timestamp
A timestamp is a uint64 count of seconds since the Unix epoch in UTC.
## Header
All Roughtime messages start with a header. The first four bytes of the header is the uint32 number of tags N, and hence of (tag, value) pairs. The following 4*(N-1) bytes are offsets, each a uint32. The last 4*N bytes in the header are tags.
Offsets refer to the positions of the values in the message values section. All offsets MUST be multiples of four and placed in increasing order. The first post-header byte is at offset 0. The offset array is considered to have a not explicitly encoded value of 0 as its zeroth entry. The value associated with the ith tag begins at offset[i] and ends at offset[i+1]-1, with the exception of the last value which ends at the end of the message. Values may have zero length. Tags MUST be listed in the same order as the offsets of their values and MUST also be sorted in ascending order by numeric value. A tag MUST NOT appear more than once in a header.
# Protocol Details

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back


# Acknowledgments
{:numbered="false"}

TODO acknowledge.
