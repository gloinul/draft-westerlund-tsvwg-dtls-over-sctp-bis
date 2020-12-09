---
docname: draft-westerlund-tsvwg-dtls-over-sctp-bis-latest
title: "Datagram Transport Layer Security (DTLS) over Stream Control Transmission Protocol (SCTP)"
abbrev: DTLS over SCTP
cat: std
obsoletes: 6083
ipr: trust200902
wg: TSVWG
area: Transport

author:
  -
   ins: M. Westerlund
   name: Magnus Westerlund
   org: Ericsson
   email: magnus.westerlund@ericsson.com
  -
   ins: J. Preuß Mattsson
   name: John Preuß Mattsson
   org: Ericsson
   email: john.mattsson@ericsson.com
  -
   ins: C. Porfiri
   name: Claudio Porfiri
   org: Ericsson
   email: claudio.porfiri@ericsson.com

informative:
  RFC0793:
  RFC3436:
  RFC5061:
  RFC6083:
  RFC6973:
  RFC7258:


normative:
  RFC2119:
  RFC3758:
  RFC4347:
  RFC4895:
  RFC4960:
  RFC5705:
  RFC6347:
  RFC7540:
  RFC8174:
  RFC8260:
  RFC8446:
  I-D.ietf-tls-dtls13:

--- abstract

This document describes a proposed update for the usage of the
Datagram Transport Layer Security (DTLS) protocol to protect user
messages sent over the Stream Control Transmission Protocol (SCTP).

DTLS over SCTP provides mutual authentication, confidentiality,
integrity protection, and replay protection for applications that
use SCTP as their transport protocol and allows client/server
applications to communicate in a way that is designed to give
communications privacy and to prevent eavesdropping and detect tampering or message forgery. 

Applications using DTLS over SCTP can use almost all transport
features provided by SCTP and its extensions. This document intend to
obsolete RFC 6083 and removes the 16 kB limitation on user message size by defining a secure user message fragmentation
so that multiple DTLS records can be used to protect a single user
message. It further updates the DTLS versions to use, as well as the
HMAC algorithms for SCTP-AUTH, and simplifies the implementation by
some stricter requirements on the procedures.

--- middle

#Introduction {#introduction}

##Overview

This document describes the usage of the Datagram Transport Layer
Security (DTLS) protocol, as defined in {{I-D.ietf-tls-dtls13}}, over
the Stream Control Transmission Protocol (SCTP), as defined in
{{RFC4960}}.

DTLS over SCTP provides mutual authentication, confidentiality,
integrity protection, and replay protection for applications that
use SCTP as their transport protocol and allows client/server
applications to communicate in a way that is designed to give
communications privacy and to prevent eavesdropping and detect tampering or message forgery. It also
provides a convinient keying mechanism for SCTP-Auth {{RFC4895}} that
prevents tampering with SCTP chunks after the DTLS handshake has
completed.

Applications using DTLS over SCTP can use almost all transport
features provided by SCTP and its extensions.

TLS, from which DTLS was derived, is designed to run on top of a
byte-stream-oriented transport protocol providing a reliable, in-
sequence delivery.  Thus, TLS is currently mainly being used on top of
the Transmission Control Protocol (TCP), as defined in {{RFC0793}}.

TLS over SCTP as described in {{RFC3436}} has some serious limitations:

   o It does not support the unordered delivery of SCTP user messages.

   o It does not support partial reliability as defined in
   {{RFC3758}}.

   o It only supports the usage of the same number of streams in both
      directions.

   o It uses a TLS connection for every bidirectional stream, which
      requires a substantial amount of resources and message exchanges
      if a large number of streams is used.

DTLS over SCTP as described in this document overcomes these
limitations of TLS over SCTP.  In particular, DTLS/SCTP supports:

   o  preservation of message boundaries.

   o  a large number of unidirectional and bidirectional streams.

   o  ordered and unordered delivery of SCTP user messages.

   o  the partial reliability extension as defined in {{RFC3758}}.

   o  the dynamic address reconfiguration extension as defined in
      {{RFC5061}}.

However, {{RFC6083}} had the following limitation:

   o The maximum user message size is 2^14 bytes, which is a single
      DTLS record limit.

This update that replaces RFC6083 defines the following changes:

   * Removes the limitations on user messages sizes by defining a
     secure fragmentation mechanism.

   * Mandates that more modern DTLS version are required (DTLS 1.2 or
     1.3)

   * Mandates use of modern HMAC algorithms (SHA-256) in the SCTP
     authentication extension {{RFC4895}}.

   * Recommends support of {{RFC8260}} to enable interleaving of large
     SCTP user messages to avoid scheduling issues.

   * Applies stricter requirements on always using DTLS for all user
     messages in the SCTP association. By defining a new SCTP parameter
     peers can determine these stricter requirements apply.

The method described in this document requires that the SCTP
implementation supports the optional feature of fragmentation of SCTP
user messages as defined in {{RFC4960}} and the SCTP authentication
extension defined in {{RFC4895}}.

## Motivation for changes

This document proposes a number of changes to RFC 6083 that have
various different motivations:

Supporting Large User Messages: RFC 6083 allowed only user messages
   that could fit within a single DTLS record . 3GPP has run into this
   limitation where they have at least four SCTP using protocols (F1,
   E1, Xn, NG-C) that can potentially generate messages over the size
   of 16384 bytes.

New Versions: Almost 10 years has passed since RFC 6083 was written,
   and significant evolution has happened in the area of DTLS and
   security algorithms. Thus DTLS 1.3 is the newest version of DTLS
   and also the SHA-1 HMAC algorithm of RFC 4895 is getting towards
   the end of usefulness. Thus, this document mandates usage of
   relevant versions and algorithms.

Clarifications: Some implementation experiences has been gained that
   motivates additional clarifications on the specification.

 * Avoid unsecured messages prior to DTLS handshake have completed.

 * Make clear that all messages are encrypted after DTLS handshake.

## Terminology

This document uses the following terms:

Association:  An SCTP association.

Stream: A unidirectional stream of an SCTP association.  It is uniquely
identified by a stream identifier.


##  Abbreviations

DTLS:  Datagram Transport Layer Security

MTU:  Maximum Transmission Unit

PPID:  Payload Protocol Identifier

SCTP:  Stream Control Transmission Protocol

TCP:  Transmission Control Protocol

TLS:  Transport Layer Security

#  Conventions

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in
   BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all
   capitals, as shown here.

#  DTLS Considerations

##  Version of DTLS

   This document is based on DTLS 1.3 {{I-D.ietf-tls-dtls13}}, but
   works also for DTLS 1.2 {{RFC6347}}. Earlier versions of DTLS MUST
   NOT be used. It is expected that DTLS/SCTP as described in this
   document will work with future versions of DTLS.

##  Cipher Suites

   For DTLS 1.2, the cipher suites forbidden by {{RFC7540}} MUST NOT
   be used. Cipher suites without encryption MUST NOT be used.

## Message Sizes {#Msg-size}

   DTLS/SCTP, automatically fragment and reassemble user
   messages. There are two different fragmentations mechanism, first
   to fragment the user messages into DTLS records, where each DTLS
   1.3 record allows a maximum of 2^14 protected bytes. The
   fragmentation mechanism headers requires 16 bytes, thus limiting
   each DTLS 1.3 record to contain a maximum of 2^14-16 = 16368 bytes
   of user message.

   Each DTLS record adds additional overhead, thus using records of
   maximum possible size are recommended to minimize the overhead. The
   sequence of DTLS records is then fragmented into DATA Chunks to fit
   the path MTU by SCTP. The largest possible user messages the
   mechanism defined in this specification can protect is 2^64-1
   bytes.

   Note: Buffering for protection operations can have practical limits.

## Replay Protection

As SCTP with SCTP-AUTH provides replay protection for DATA chunks, DTLS/SCTP provides replay protection for user messages.

DTLS optionally supports record replay detection. Such replay detection could result in the DTLS layer dropping valid messages received outside of the DTLS replay window. As DTLS/SCTP provides replay protection even without DTLS replay protection, the replay detection of DTLS MUST NOT be used.

##  Path MTU Discovery

   SCTP provides Path MTU discovery and fragmentation/reassembly for
   user messages.  According to {{Msg-size}}, DTLS can send maximum sized
   DTLS Records.  Therefore, Path MTU discovery of DTLS MUST NOT be used.

##  Retransmission of Messages

   SCTP provides a reliable and in-sequence transport service for DTLS
   messages that require it.  See {{Stream-Usage}}.  Therefore, DTLS
   procedures for retransmissions MUST NOT be used.

#  SCTP Considerations

##  Mapping of DTLS Records

   The supported maximum length of SCTP user messages MUST be at least
   1024 kB. In particular, the SCTP implementation MUST support
   fragmentation of user messages.

   DTLS/SCTP works as a shim layer between the user message API and
   SCTP. The fragmentation works similar as the DTLS fragmentation of
   handshake messages.  On the sender side a user message fragmented
   into fragments m0, m1, m2, each smaller than 2^14 - 16 = 16368
   bytes.

~~~~~~~~~~~
   m0 | m1 | m2 | ... = uint64(length) | user_message
~~~~~~~~~~~

   The resulting fragments are protected with DTLS and the records are
   concatenated

~~~~~~~~~~~
   user_message' = DTLS( m0' ) | DTLS( m1' ) | DTLS( m2' ) ...
~~~~~~~~~~~

   where

~~~~~~~~~~~
   mi' = uint64(nonce) | uint64(i) | mi
~~~~~~~~~~~

   and where nonce is has a different value for each user message
   (e.g. a counter). The new user_message' is the input to SCTP.

   On the receiving side DTLS is used to decrypt the records and the
   fields uint64(length), uint64(nonce), and uint64(i) are
   removed. The user_message is valid if all DTLS records are valid,
   uint64(nonce) is the same in all records, numbers uint64(i) is an ordered sequence
   from 0 to the number of records, and uint64(length) is the length
   of the resulting user_message. If a DTLS decryption fails or
   a user_message is not valid, the DTLS connection and the SCTP
   association are terminated.

##  DTLS Connection Handling

   The DTLS connection MUST be established at the beginning of the SCTP
   association and be terminated when the SCTP association is terminated,
   (i.e. there's only one DTLS connection within one association).
   A DTLS connection MUST NOT span multiple SCTP associations.

##  Payload Protocol Identifier Usage

   Application protocols using DTLS over SCTP SHOULD register and use a
   separate payload protocol identifier (PPID) and SHOULD NOT reuse the
   PPID that they registered for running directly over SCTP.

   Using the same PPID does not harm as long as the application can
   determine whether or not DTLS is used.  However, for protocol
   analyzers, for example, it is much easier if a separate PPID is used.

   This means, in particular, that there is no specific PPID for DTLS.

##  Stream Usage {#Stream-Usage}

   All DTLS messages of the Handshake, Alert, or ChangeCipherSpec protocol
   (DTLS 1.2 only) MUST be transported on stream 0 with unlimited reliability
   and with the ordered delivery feature.

   DTLS messages of the record protocol SHOULD use multiple
   streams other than stream 0; they MAY use stream 0 for everything if
   they do not care about minimizing head of line blocking.

##  Chunk Handling

   DATA chunks of SCTP MUST be sent in an authenticated way as
   described in {{RFC4895}}.  All other chunks that may be
   authenticated, i.e. all chunks that can be listed in the Chunk List Parameter
   {{RFC4895}}, MUST also be sent in an authenticated way.  This makes
   sure that an attacker cannot modify the stream in which a message
   is sent or affect the ordered/unordered delivery of the message.

   If PR-SCTP as defined in {{RFC3758}} is used, FORWARD-TSN chunks
   MUST also be sent in an authenticated way as described in
   {{RFC4895}}.  This makes sure that it is not possible for an
   attacker to drop messages and use forged FORWARD-TSN, SACK, and/or
   SHUTDOWN chunks to hide this dropping.

   I-DATA chunk type as defined in {{RFC8260}} is RECOMMENDED to be
   supported to avoid some of the down sides that large user messages
   have on blocking transmission of later arriving high priority user
   messages. However, the support is not mandated and negotiated
   independently from DTLS/SCTP. If I-DATA chunks are used then
   they MUST be sent in an authenticated way as described in
   {{RFC4895}}.

##  SCTP-AUTH Hash Function

   When using DTLS/SCTP, the SHA-256 Message Digest Algorithm MUST be supported in the
   SCTP-AUTH {{RFC4895}} implementation. SHA-1 MUST NOT be used when
   using DTLS/SCTP. {{RFC4895}} requires support and inclusion of of
   SHA-1 in the HMAC-ALGO parameter, thus, to meet both requirements
   the HMAC-ALGO parameter will include both SHA-256 and SHA-1 with
   SHA-256 listed prior to SHA-1 to indicate the preference.

## Renegotiation

   Renegotiation MUST NOT be used.

##  Handshake {#HANDSHAKE}

   A DTLS implementation discards DTLS messages from older epochs
   after some time, as described in Section 4.1 of {{RFC4347}}.  This
   is not acceptable when the DTLS user performs a reliable data
   transfer.  To avoid discarding messages, the following procedures
   are required.

   Before sending a ChangeCipherSpec message, all outstanding SCTP
   user messages MUST have been acknowledged by the SCTP peer and MUST
   NOT be revoked by the SCTP peer.

   Prior to processing a received ChangeCipherSpec, all other received
   SCTP user messages that are buffered in the SCTP layer MUST be read
   and processed by DTLS.

   User messages that arrive between ChangeCipherSpec and Finished
   messages and use the new epoch have probably passed the Finished
   message and MUST be buffered by DTLS until the Finished message is
   read.

##  Handling of Endpoint-Pair Shared Secrets

SCTP-AUTH {{RFC4895}} is keyed using Endpoint-Pair Shared Secrets. In
SCTP associations where DTLS is used, DTLS is used to establish these
secrets. The endpoints MUST NOT use another mechanism for establishing
shared secrets for SCTP-AUTH.

The endpoint-pair shared secret for Shared Key Identifier 0 is empty
and MUST be used when establishing a DTLS connection.  Whenever the
master key changes, a 64-byte shared secret is derived from every
master secret and provided as a new endpoint-pair shared secret by
using the TLS-Exporter. For DTLS 1.3, the exporter is described in
{{RFC8446}}. For DTLS 1.2, the exporter is described in
{{RFC5705}}. The exporter MUST use the label given in Section
{{IANA-Consideration}} and no context.  The new Shared Key Identifier
MUST be the old Shared Key Identifier incremented by 1.  If the old
one is 65535, the new one MUST be 1.

Before sending the DTLS Finished message, the active SCTP-AUTH key
MUST be switched to the new one.

Once the corresponding Finished message from the peer has been
received, the old SCTP-AUTH key SHOULD be removed.

##  Shutdown

   To prevent DTLS from discarding DTLS user messages while it is
   shutting down, a CloseNotify message MUST only be sent after all
   outstanding SCTP user messages have been acknowledged by the SCTP
   peer and MUST NOT still be revoked by the SCTP peer.

   Prior to processing a received CloseNotify, all other received SCTP
   user messages that are buffered in the SCTP layer MUST be read and
   processed by DTLS.

## Negotiation of DTLS support {#Negotiation}

To distinguish supporters of this specification compared to RFC 6083
as well as enable certain improvements that simplifies implementation
a new SCPT parameter is defined.

### New option at INIT/INIT-ACK {#DTLS-supported}


   The following new OPTIONAL parameter is added to the INIT and INIT
   ACK chunks.

~~~~~~~~~~~
   Parameter Name                      Status      Type Value
   --------------------------------------------------------------
   DTLS-Supported                      OPTIONAL    XXXXX (0x????)
~~~~~~~~~~~

   At the initialization of the association, the sender of the INIT or
   INIT ACK chunk MAY include this OPTIONAL parameter to inform its
   peer that it is able to support DTLS over SCTP per this
   specification. The format of this parameter is defined as follows:

~~~~~~~~~~~
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |    Parameter Type = XXXXX     |  Parameter Length = 4         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~~

Type: 16 bit u_int
      XXXXX, DTLS-Supported parameter

Length: 16 bit u_int
      Indicates the size of the parameter, i.e., 4.

## DTLS over SCTP service

   The adoption of DTLS over SCTP according to the current description
   is meant to add to SCTP the option for transferring encrypted data.
   When DTLS-option is enabled, all data being transferred must be
   protected by chunk authentication and DTLS encrypted.  Chunks that
   can be transferred will be specified in the CHUNK list parameter
   according to {{RFC4895}}.  Error handling for authenticated chunks
   is according to {{RFC4895}}.

### DTLS over SCTP initialization {#DTLS-init}

   Initialization of DTLS/SCTP requires all the following options to
   be part of the INIT/INIT-ACK handshake:

   RANDOM: defined in {{RFC4895}}

   CHUNKS: list of permitted chunks, defined in {{RFC4895}}

   HMAC-ALGO: defined in {{RFC4895}}

   DTLS-Supported: defined in {{DTLS-supported}}

   When all the above options are present, the Association will start
   with support of DTLS/SCTP.  The set of options indicated are the
   DTLS/SCTP Mandatory Options.  No data transfer is permitted before
   DTLS handshake is complete.  Data chunks that are received before
   DTLS handshake will be silently discarded.  Chunk bundling is
   permitted according to {{RFC4960}}

   The extension described in this document is given by the following
   message exchange.

~~~~~~~~~~~
    -------- INIT[RANDOM; CHUNKS; HMAC-ALGO; DTLS-Supported] ------->
    <----- INIT-ACK[RANDOM; CHUNKS; HMAC-ALGO; DTLS-Supported] ------
    -------------------------- COOKIE-ECHO ------------------------->
    <-------------------------- COOKIE-ACK --------------------------
    ------------------ AUTH; DATA[DTLS Handshake] ------------------>
                                ...
                                ...
    <----------------- AUTH; DATA[DTLS Handshake] -------------------
~~~~~~~~~~~

### Client Use Case

   When a SCTP Client initiates an Association with DTLS/SCTP
   Mandatory Options, it can receive an INIT-ACK also containing
   DTLS/SCTP Mandatory Options, in that case the Association will
   proceed as specified in the previous {{DTLS-init}} section.  If the
   peer replies with an INIT-ACK not containing all DTLS/SCTP
   Mandatory Options, the Client can decide to keep on working with
   plain data only or to ABORT the association.

### Server Use Case

   If a SCTP Server supports DTLS/SCTP, when receiving an INIT chunk
   with all DTLS/SCTP Mandatory Options it must reply with INIT-ACK
   also containing the all DTLS/SCTP Mandatory Options, then it must
   follow the sequence for DTLS initialization {{DTLS-init}} and the
   related traffic case.  If a SCTP Server supports DTLS, when
   receiving an INIT chunk with not all DTLS/SCTP Mandatory
   Options, it can decide to continue by creating an Association with
   plain data only or to ABORT it.


#  IANA Considerations {#IANA-Consideration}

## TLS Exporter Label

RFC 6083 defined a TLS Exporter Label registry as described in
{{RFC5705}}. IANA is requested to update the reference for the label
"EXPORTER_DTLS_OVER_SCTP" to this specification.

## SCTP Parameter

IANA is requested to register a new SCTP parameter "DTLS-support".

#  Security Considerations

   The security considerations given in {{I-D.ietf-tls-dtls13}},
   {{RFC4895}}, and {{RFC4960}} also apply to this document.

##  Cryptographic Considerations

   When DTLS/SCTP is used with DTLS 1.2 {{RFC6347}}, DTLS 1.2 MUST be
   configured to disable options known to provide insufficient
   security. HTTP/2 {{RFC7540}} gives good minimum requirements based
   on the attacks that where publicly known in 2015. DTLS 1.3
   {{I-D.ietf-tls-dtls13}} only define strong algorithms without major
   weaknesses.

   DTLS 1.3 requires rekeying before algorithm specific AEAD limits
   have been reached. HMAC-SHA-256 as used in SCTP-AUTH has a very
   large tag length and very good integrity properties. The SCTP-AUTH
   key can be used until the DTLS handshake is re-run at which point a
   new SCTP-AUTH key is derived using the TLS-Exporter.

   DTLS/SCTP is in many deployments replacing IPsec. For IPsec, NIST
   (US), BSI (Germany), and ANSSI (France) recommends very frequent
   re-run of Diffie-Hellman to provide Perfect Forward Secrecy. ANSSI
   writes "It is recommended to force the periodic renewal of the
   keys, e.g. every hour and every 100 GB of data, in order to limit
   the impact of a key compromise.". This is RECOMMENDED also for
   DTLS/SCTP.

##  Downgrade Attacks

   A peer supporting DTLS/SCTP according to this specification,
   DTLS/SCTP according to {{RFC6083}} and/or SCTP without DTLS may be
   vulnerable to downgrade attacks where on on-path attacker
   interferes with the protocol setup to lower or disable security. If
   possible, it is RECOMMENDED that the peers have a policy only
   allowing DTLS/SCTP according to this specification.

##  Authentication and Policy Decisions

   DTLS/SCTP MUST be mutually authenticated. It is RECOMMENDED that
   DTLS/SCTP is used with certificate based authentication.  All
   security decisions MUST be based on the peer's authenticated
   identity, not on its transport layer identity.

   It is possible to authenticate DTLS endpoints based on IP addresses
   in certificates. SCTP associations can use multiple IP addresses
   per SCTP endpoint. Therefore, it is possible that DTLS records will
   be sent from a different source IP address or to a different
   destination IP address than that originally authenticated. This is
   not a problem provided that no security decisions are made based on
   the source or destination IP addresses.

##  Privacy Considerations

   {{RFC6973}} suggests that the privacy considerations of IETF
   protocols be documented.

   For each SCTP user message, the user also provides a stream
   identifier, a flag to indicate whether the message is sent ordered
   or unordered, and a payload protocol identifier.  Although
   DTLS/SCTP provides privacy for the actual user message, the other
   three information fields are not confidentiality protected.  They
   are sent as clear text, because they are part of the SCTP DATA
   chunk header.

   It is RECOMMENDED that DTLS/SCTP is used with certificate based
   authentication in DTLS 1.3 {{I-D.ietf-tls-dtls13}} to provide
   identity protection. DTLS/SCTP MUST be used with a key exchange
   method providing Perfect Forward Secrecy. Perfect Forward Secrecy
   significantly limits the amount of data that can be compromised due
   to key compromise.

##  Pervasive Monitoring

   As required by {{RFC7258}}, work on IETF protocols needs to
   consider the effects of pervasive monitoring and mitigate them when
   possible.

   Pervasive Monitoring is widespread surveillance of users.  By
   encrypting more information including user identities, DTLS 1.3
   offers much better protection against pervasive monitoring.

   Massive pervasive monitoring attacks relying on key exchange
   without forward secrecy has been reported. By mandating perfect
   forward secrecy, DTLS/SCTP effectively mitigate many forms of
   passive pervasive monitoring and limits the amount of compromised
   data due to key compromise.

   In addition to the privacy attacks discussed above, surveillance on
   a large scale may enable tracking of a user over a wider
   geographical area and across different access networks.  Using
   information from DTLS/SCTP together with information gathered from
   other protocols increases the risk of identifying individual users.

# Changes from RFC 6083

This specification of DTLS over SCTP has the following changes
compared to the DTLS over SCTP that defined in {{RFC6083}}.

 * Defines a mechanism to fragment user message across multiple DTLS
   records in secure way.

 * Defines a SCTP parameters to negotiate support of DTLS over SCTP.

 * Requires that the DTLS handshake needs to occur immediately after
   SCTP handshake prior to any other user messages when this
   specification is supported.

 * Requires that SHA-256 is supported in SCTP-AUTH {{RFC4895}} when
   combined with DTLS/SCTP. Similarily SHA-1 is forbidden to be used.


#  Acknowledgments

   The authors of RFC 6083 which this document was based on are
   Michael Tüxen, Eric Rescorla, and Robin Seggelmann.

   The authors wish to thank Anna Brunstrom, Lars Eggert, Gorry
   Fairhurst, Ian Goldberg, Alfred Hoenes, Carsten Hohendorf, Stefan
   Lindskog, Daniel Mentz, and Sean Turner for their invaluable
   comments.
