---
title: "TESLA Update for GNSS SBAS Authentication"
abbrev: "TESLA SBAS Authentication"
category: std
updates: rfc4082
docname: draft-moskowitz-rfc4082-update-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
# area: sec
pi:
  toc: yes
  symrefs: yes
  sortrefs: yes
  compact: yes
  subcompact: no
  iprnotified: no
  strict: no

venue:
#  mail: "rfc4082-update@ietf.org"
#  arch: "https://mailarchive.ietf.org/arch/browse/rfc4082-update"
  github: "rfc4082-update/draft-moskowitz-rfc4082-update"

author:
-
  ins: R. Moskowitz
  fullname: Robert Moskowitz
  organization: HTT Consulting
  email: rgm@labs.htt-consult.com
  city: Oak Park
  region: MI
  code: 48237
  country: USA
  role: editor
-
  ins: R. Canetti
  fullname: Ran Canetti
  organization: Boston University
  email: canetti@bu.edu
  city: Boston
  region: MA
  code: 02215
  country: USA

normative:
  RFC4082:

informative:
  RFC1035:
  RFC2104:
  I-D.ietf-cose-cbor-encoded-cert:
  NIST.SP.800-185: DOI.10.6028/nist.sp.800-185
  MAVLINK:
    title: "Micro Air Vehicle Communication Protocol"
    date: 2021
    target: https://mavlink.io/
  SBAS_Authentication: DOI.10.33012/navi.595


...

--- abstract

This document updates TESLA [RFC4082] to current
cryptographic methods leveraging the work done by the International
Civil Aviation Organization (ICAO) in their Global Navigation
Satellite System (GNSS) Satellite-based augmentation system (SBAS)
authentication protocol.  The TESLA updates are to align to this
and other current best practices.

--- middle

# Introduction

TESLA [RFC4082] (Timed Efficient Stream
Loss-Tolerant Authentication) uses the best practices for
cryptography when published in 2005.  This is quite dated, and any
modern use of TESLA needs to adjust to current algorithms and
methods.

This document starts with the TESLA design targeted by the
International Civil Aviation Organization (ICAO) in their Global
Navigation Satellite System (GNSS) Satellite-based augmentation
system (SBAS) authentication protocol.  As other modern uses are
shared, this document will be adjusted accordingly.

The SBAS authentication protocol is more than a modern TESLA
implementation.  It uses a very tightly designed PKI and the C509
certificate encoding [I-D.ietf-cose-cbor-encoded-cert]
to work within the very highly constrained SBAS
communication link.  The PKI is out-of-scope for this document and
is described elsewhere within ICAO.  But the process of Key
Disclosure used in SBAS will be included here.

This document is very much a "work in progress".  Various ICAO SBAS
documents need to be excised for their technical updates to TESLA.
Also, it is anticipated that other modern uses of TESLA will be
captured herein.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Notation {#notation}

||:

 : Signifies concatenation of information (e.g., X || Y is the
   concatenation of X with Y).

Ltrunc (x, K):

 : Denotes the lowest order K bits of the input x.

# Updates to TESLA {#Updates}

# TESLA Time Synchronization {#Time_Sync}

TESLA references "indirect time synchronization" like NTP [RFC1035].
It specifies that a controller and senders
"engaged  in a protocol for finding the value D^0_t between them",
with controller and receivers "find the value D^R_t".  This is not
practical with GNSS time services.

TESLA time synchronization with broadcast only time services, like
GNSS time, may be set up with out-of-band data (e.g. T_int) and
in-band public key authenticated data.  This in-band data
transmissions need regular transmissions to accommodate "late
joiner receivers".

There is a challenge for receivers to use GNSS time before TESLA is
authenticating those time broadcasts.  Thus a reciever should work
in the pre-authenticated mode only to enable switch to trusted GNSS
time.

Use of NTP should be limited to authenticated NTP. (Editor: more
needed here)

# TESLA Message Authentication Code {#TESLA_MAC}

TESLA uses a "cryptographic MAC" that MUST be cryptographically
secure.  It does not provide any guidelines to what is secure.  As
industry has shown that they will field cryptographically weak easy
keyed-MACs (e.g. Mavlink 2.0 {{MAVLINK}} this update specifies that
TESLA SHOULD, at a minimum, use HMAC [RFC2104] with at least SHA2 or
KMAC {{NIST.SP.800-185}}.

# Additional Info in MAC {#TESLA_MAC_info}

Current MAC best practices allow for the inclusion of Additional
Information added to the message block (e.g. M || "Message
Domain").  This is particularly important with very short messages
(e.g. SBAS 250 bit messages).

The MAC function used in a TESLA implementation SHOULD include
Additional Information.  The size of this Additional Information is
determined by the size of the original message to MAC and the MAc's
security characteristics.

# An Aggregated MAC for TESLA {#TESLA_aMAC}

In situations where the link capacity cannot support a TESLA packet
for each data message, a set of MAC messages may be aggregated,
aMAC, and then the aMAC is transmitted. This transmission savings
comes at the risk that if the aMAC is lost, a whole set of messages
are not authenticated.

~~~

  aMAC = Ltrunc (28, MAC(k, M1 || M2 || M3 || M4 || M5 || 0000))

~~~
{: #pseudoheader title="Aggregated MAC example"}

Mi:

 : Mi is the message broadcast at time t all using the same key

k:

  : k is the cryptographic key associated with M

# Adding Block Erasure Codes or FEC {#TESLA_aMAC_BEC}

When TESLA MACs individual packets, a loss of a MAC and thus an
unauthenticated may not matter.  When aMACs are used, a loss aMAC
could be disruptive; adding a FEC (Forward Error Correction) or
Block Erasure Codes may be worth the additional transmission cost.

This potential lost is highly likely in noisy links like GNSS SBAS
(due to natural or malicious interference) where adding Block
Erasure Codes is considered important.

The EVENODD code is an example of an erasure code that can be used
here.

It is a specific, highly efficient erasure coding scheme, primarily
used in RAID-6 storage systems, that employs simple XOR operations
and two redundant disks to protect against up to two simultaneous
failures.  Likewise, it can be used to recover up to two
simultaneous aMACs.

Author's note: Does this section needs expanding?.  Should more
details on EVENODD be provided?

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# SBAS use of TESLA {#About_SBAS}

The updating of TESLA in SBAS Authentication is outlined in
[SBAS_Authentication].  This document  is the public source of changes made to
TESLA and some of the justifications.

TBD - extracted from SBAS documents.

## Adding Block Erasure Codes or FEC {#SBAS_aMAC_BEC}

SBAS uses the ODDEVEN Block Erasure Code that is built on a set of
5 aMACS which are an aggregation of 5 MACs.  Thus the Block Erasure
Code recovers the MAC that protected 25 SBAS messages.

Author's note: This section needs expanding.  Details of the SBAS
Block Erasure Codes be included?

# Acknowledgments
{:numbered="false"}

This work is in conjunction with the ICAO SBAS Authention Study
Group members.  This includes, and is not limited to: Jed Dennis
(FAA Consultant), Abdel Youssouf (Eurocontrol), Timo Warns
(Airbus), Todd Walter (Stanford) and chair Mikaël Mabilleau
(Eurocontrol).
