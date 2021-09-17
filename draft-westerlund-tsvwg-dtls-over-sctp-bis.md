---
docname: draft-ietf-tsvwg-dtls-over-sctp-bis-latest
title: "Datagram Transport Layer Security (DTLS) over Stream Control Transmission Protocol (SCTP)"
abbrev: DTLS over SCTP
cat: std
obsoletes: 6083
ipr: trust200902
wg: TSVWG
area: Transport

author:
-
   ins:  M. Westerlund
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
-
   ins: M. Tüxen
   name: Michael Tüxen
   org: Münster University of Applied Sciences
   abbrev: Münster Univ. of Appl. Sciences
   street: Stegerwaldstrasse 39
   code: 48565
   city: Steinfurt
   country: Germany
   email: tuexen@fh-muenster.de

informative:
  RFC3436:
  RFC3788:
  RFC5061:
  RFC6083:
  RFC6458:
  RFC6973:
  RFC7258:
  RFC7457:
  RFC7525:

  ANSSI-DAT-NT-003:
    target: <https://www.ssi.gouv.fr/uploads/2015/09/NT_IPsec_EN.pdf>
    title: Recommendations for securing networks with IPsec
    seriesinfo:
      ANSSI Technical Report DAT-NT-003
    author:
      -
        ins: Agence nationale de la sécurité des systèmes d'information
    date: August 2015

normative:
  RFC2119:
  RFC3758:
  RFC4895:
  RFC4960:
  RFC5246:
  RFC5705:
  RFC5746:
  RFC6347:
  RFC7540:
  RFC8174:
  RFC8260:
  RFC8446:
  RFC8996:
  I-D.ietf-tls-dtls13:

--- abstract

   This document describes a proposed update for the usage of the
   Datagram Transport Layer Security (DTLS) protocol to protect user
   messages sent over the Stream Control Transmission Protocol (SCTP).

   DTLS over SCTP provides mutual authentication, confidentiality,
   integrity protection, and replay protection for applications that
   use SCTP as their transport protocol and allows client/server
   applications to communicate in a way that is designed to give
   communications privacy and to prevent eavesdropping and detect
   tampering or message forgery.

   Applications using DTLS over SCTP can use almost all transport
   features provided by SCTP and its extensions. This document intends
   to obsolete RFC 6083 and removes the 16 kB limitation on user
   message size by defining a secure user message fragmentation so
   that multiple DTLS records can be used to protect a single user
   message. It further updates the DTLS versions to use, as well as
   the HMAC algorithms for SCTP-AUTH, and simplifies secure 
   implementation by some stricter requirements on the establishment
   procedures.

--- middle

# Introduction {#introduction}

## Overview

   This document describes the usage of the Datagram Transport Layer
   Security (DTLS) protocol, as defined in DTLS 1.2 {{RFC6347}}, and
   DTLS 1.3 {{I-D.ietf-tls-dtls13}}, over the Stream Control
   Transmission Protocol (SCTP), as defined in {{RFC4960}} with
   Authenticated Chunks for SCTP (SCTP-AUTH) {{RFC4895}}.

   This specification provides mutual authentication of endpoints,
   confidentiality, integrity protection, and replay protection of
   user messages for applications that use SCTP as their transport
   protocol.  Thus, it allows client/server applications to communicate
   in a way that is designed to give communications privacy and to
   prevent eavesdropping and detect tampering or message
   forgery. DTLS/SCTP uses DTLS for mutual authentication, key
   exchange with perfect forward secrecy for SCTP-AUTH, and
   confidentiality of user messages. DTLS/SCTP use SCTP and SCTP-AUTH
   for integrity protection and replay protection of user messages.

   Applications using DTLS over SCTP can use almost all transport
   features provided by SCTP and its extensions. DTLS/SCTP supports:

* preservation of message boundaries.

* a large number of unidirectional and bidirectional streams.

* ordered and unordered delivery of SCTP user messages.

* the partial reliability extension as defined in {{RFC3758}}.

* the dynamic address reconfiguration extension as defined in
      {{RFC5061}}.

* large user messages.

The method described in this document requires that the SCTP
implementation supports the optional feature of fragmentation of SCTP
user messages as defined in {{RFC4960}}. The implementation is also
required to have an SCTP API (for example the one described in
{{RFC6458}}) that supports partial user message delivery and also
recommended that I-DATA chunks as defined in {{RFC8260}} is used to
efficiently implement and support larger user messages.

To simplify implementation and reduce the risk for security holes,
limitations have been defined such that STARTTLS as specified in
{{RFC3788}} is no longer supported.

### Comparison with TLS for SCTP

TLS, from which DTLS was derived, is designed to run on top of a
byte-stream-oriented transport protocol providing a reliable, in-
sequence delivery. TLS over SCTP as described in {{RFC3436}} has
some serious limitations:

* It does not support the unordered delivery of SCTP user messages.

* It does not support partial reliability as defined in
   {{RFC3758}}.

* It only supports the usage of the same number of streams in both
      directions.

* It uses a TLS connection for every bidirectional stream, which
      requires a substantial amount of resources and message exchanges
      if a large number of streams is used.

### Changes from RFC 6083

The DTLS over SCTP solution defined in RFC 6083 had the following
limitation:

* The maximum user message size is 2^14 bytes, which is a single
      DTLS record limit.

This update that replaces RFC 6083 defines the following changes:

* Removes the limitations on user messages sizes by defining a
     secure fragmentation mechanism.

* Mandates that more modern DTLS version are required (DTLS 1.2 or
     1.3)

* Mandates support of modern HMAC algorithm (SHA-256) in the SCTP
     authentication extension {{RFC4895}}.

* Recommends support of {{RFC8260}} to enable interleaving of large
     SCTP user messages to avoid scheduling issues.

* Applies stricter requirements on always using DTLS for all user
     messages in the SCTP association.

* Requires that SCTP-AUTH is applied to all SCTP Chunks that can be
     authenticated.

* Requires support of partial delivery of user messages.

Mandating DTLS 1.2 or DTLS 1.3 instead to using DTLS 1.0 limits the lifetime
of a DTLS connection and the data volume which can be transferred over a
DTLS connection. This is cause by:

* The number of renegotations in DTLS 1.2 is limited to 65534 compared to
  unlimited in DTLS 1.0.

* The number of KeyUpdates in DTLS 1.3 is limited to 65532 and renegotiations
  are not supported.

## DTLS Version

DTLS/SCTP as defined by this document can use either DTLS 1.2
{{RFC6347}} or DTLS 1.3 {{I-D.ietf-tls-dtls13}}. Some crucial
difference between the DTLS versions make it necessary for a user of
DTLS/SCTP to make an informed choice of the DTLS version to use based
on their application's requirements. In general, DTLS 1.3 is to
preferred being a newer protocol that addresses known vulnerabilities
and only defines strong algorithms without known major weaknesses at
the time of publication.

However, some applications using DTLS/SCTP are of semi-permanent
nature and use SCTP associations with lifetimes that are more than a
few hours, and where there is a significant cost of bringing down the
SCTP association in order to restart it. For such DTLS/SCTP usages
that need either of:

  * Periodic re-authentication of both endpoints (not only the DTLS client).
  
  * Periodic rerunning of Diffie-Hellman key-exchange to provide
    Perfect Forward Secrecy (PFS) to reduce the impact any key-reveal.
 
  * Perform SCTP-AUTH re-keying.

At the time of publication DTLS 1.3 does not support any of these,
where DTLS 1.2 renegotiation functionality can provide this
functionality in the context of DTLS/SCTP. The application will have
to analyze its needs and requirements on the above and based on this
select the DTLS version to use.

To address known vulnerabilities in DTLS 1.2 this document describes
and mandates implementation constraints on ciphers, protocol options
and how to use the DTLS renegotiation mechanism.

In the rest of the document, unless the version of DTLS is
specifically called out the text applies to both versions of DTLS.

## Terminology

This document uses the following terms:

Association:  An SCTP association.

Stream: A unidirectional stream of an SCTP association.  It is uniquely
identified by a stream identifier.

## Abbreviations

DTLS:  Datagram Transport Layer Security

MTU:  Maximum Transmission Unit

PFS:  Perfect Forward Secrecy

PPID:  Payload Protocol Identifier

SCTP:  Stream Control Transmission Protocol

TCP:  Transmission Control Protocol

TLS:  Transport Layer Security

ULP:  Upper Layer Protocol

# Conventions

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
   "OPTIONAL" in this document are to be interpreted as described in
   BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all
   capitals, as shown here.

# DTLS Considerations

## Version of DTLS

   This document defines the usage of either DTLS 1.3
   {{I-D.ietf-tls-dtls13}}, or DTLS 1.2 {{RFC6347}}.
   Earlier versions of DTLS MUST NOT be used (see {{RFC8996}}).
   It is expected that DTLS/SCTP as described in this document will work with
   future versions of DTLS.

##  Cipher Suites and Cryptographic Parameters

   For DTLS 1.2, the cipher suites forbidden by {{RFC7540}} MUST NOT
   be used. For all versions of DTLS, cryptographic parameters giving
   confidentiality and Perfect Forward Secrecy (PFS) MUST be used.

## Message Sizes {#Msg-size}

   DTLS/SCTP, automatically fragments and reassembles user
   messages. This specification defines how to fragment the user
   messages into DTLS records, where each DTLS record allows a
   maximum of 2^14 protected bytes. Each DTLS record adds some
   overhead, thus using records of maximum possible size are
   recommended to minimize the transmitted overhead. DTLS 1.3
   has much less overhead than DTLS 1.2 per record.

   The sequence of DTLS records is then fragmented into DATA or I-DATA
   Chunks to fit the path MTU by SCTP. The largest possible user
   messages using the mechanism defined in this specification is
   2^64-1 bytes.

   The security operations and reassembly process requires that the
   protected user message, i.e., with DTLS record overhead, is buffered
   in the receiver. This buffer space will thus put a limit on the
   largest size of plain text user message that can be transferred
   securely. However, by mandating the use of the partial delivery of
   user messages from SCTP and assuming that no two messages received
   on the same stream are interleaved (as it is the case when using
   the API defined in {{RFC6458}}) the required buffering prior to
   DTLS processing can be limited to a single DTLS record per used
   incoming stream. This enables the DTLS/SCTP implementation to
   provide the Upper Layer Protocol (ULP) with each DTLS record's
   content when it has been decrypted and its integrity been verified
   enabling partial user message delivery to the ULP. Implementations
   can trade-off buffer memory requirements in the DTLS layer with
   transport overhead by using smaller DTLS records.

   The DTLS/SCTP implementation is expected to behave very similar to
   just SCTP when it comes to handling of user messages and dealing
   with large user messages and their reassembly and
   processing. Making it the ULP responsible for handling any resource
   contention related to large user messages.
   
## Replay Protection

   SCTP-AUTH {{RFC4895}} does not have explicit replay
   protection. However, the combination of SCTP-AUTH's protection of
   DATA or I-DATA chunks and SCTP user message handling will prevent
   third party attempts to inject or replay SCTP packets resulting in
   impact on the received protected user message. In fact, this
   document's solution is dependent on SCTP-AUTH and SCTP to prevent
   reordering, duplication, and removal of the DTLS records within
   each protected user message.  This includes detection of changes to
   what DTLS records start and end the SCTP user message, and removal of
   DTLS records before an increment to the epoch.  Without SCTP-AUTH,
   these would all have required explicit handling.

   DTLS optionally supports record replay detection. Such replay
   detection could result in the DTLS layer dropping valid messages
   received outside of the DTLS replay window. As DTLS/SCTP provides
   replay protection even without DTLS replay protection, the replay
   detection of DTLS MUST NOT be used.

## Path MTU Discovery

   DTLS Path MTU Discovery MUST NOT be used.  Since SCTP provides Path
   MTU discovery and fragmentation/reassembly for user messages, and
   according to {{Msg-size}}, DTLS can send maximum sized DTLS
   Records.

## Retransmission of Messages

   SCTP provides a reliable and in-sequence transport service for DTLS
   messages that require it.  See {{Stream-Usage}}.  Therefore, DTLS
   procedures for retransmissions MUST NOT be used.

# SCTP Considerations

## Mapping of DTLS Records {#Mapping-DTLS}

   The SCTP implementation MUST support fragmentation of user messages
   using DATA {{RFC4960}}, and optionally I-DATA {{RFC8260}} chunks.

   DTLS/SCTP works as a shim layer between the user message API and
   SCTP. The fragmentation works similar as the DTLS fragmentation of
   handshake messages.  On the sender side a user message fragmented
   into fragments m0, m1, m2, each no larger than 2^14 - 1 = 16383
   bytes.

~~~~~~~~~~~
   m0 | m1 | m2 | ... = user_message
~~~~~~~~~~~

   The resulting fragments are protected with DTLS and the records are
   concatenated

~~~~~~~~~~~
   user_message' = DTLS( m0 ) | DTLS( m1 ) | DTLS( m2 ) ...
~~~~~~~~~~~

   The new user_message', i.e., the protected user message, is the input
   to SCTP.

   On the receiving side DTLS is used to decrypt the individual
   records. There are three failure cases an implementation needs to
   detect and then act on:

   1. Failure in decryption and integrity verification process of any
   DTLS record. Due to SCTP-AUTH preventing delivery of injected or
   corrupt fragments of the protected user message this should only
   occur in case of implementation errors or internal hardware
   failures.

   2. In case the SCTP layer indicates an end to a user message,
   e.g. when receiving a MSG_EOR in a recvmsg() call when using the
   API described in {{RFC6458}}, and the last buffered DTLS record
   length field does not match, i.e., the DTLS record is incomplete.

   3. Unable to perform the decryption processes due to lack of
   resources, such as memory, and have to abandon the user message
   fragment. This specification is defined such that the needed
   resources for the DTLS/SCTP operations are bounded for a given
   number of concurrent transmitted SCTP streams or unordered user
   messages. 
   
   The above failure cases all result in the receiver failing to
   recreate the full user message. This is a failure of the transport
   service that is not possible to recover from in the DTLS/SCTP layer
   and the sender could believe the complete message have been
   delivered. This error MUST NOT be ignored, as SCTP lacks any
   facility to declare a failure on a specific stream or user message,
   the DTLS connection and the SCTP association SHOULD be
   terminated. A valid exception to the termination of the SCTP
   association is if the receiver is capable of notifying the ULP
   about the failure in delivery and the ULP is capable of recovering
   from this failure.

   Note that if the SCTP extension for Partial Reliability (PR-SCTP)
   {{RFC3758}} is used for a user message, user message may be
   partially delivered or abandoned. These failures are not a reason
   for terminating the DTLS connection and SCTP association.

   The DTLS Connection ID SHOULD NOT be negotiated (Section 9 of
   {{I-D.ietf-tls-dtls13}}). If DTLS 1.3 is used, the length field
   MUST be included and a 16-bit sequence number SHOULD be used.

## DTLS Connection Handling

   The DTLS connection MUST be established at the beginning of the
   SCTP association and be terminated when the SCTP association is
   terminated, (i.e., there's only one DTLS connection within one
   association).  A DTLS connection MUST NOT span multiple SCTP
   associations.

   As it is required to establish the DTLS connection at the beginning
   of the SCTP association, either of the peers should never send any
   SCTP user messages that are not protected by DTLS. So, the case that
   an endpoint receives data that is not either DTLS messages on Stream
   0 or protected user messages in the form of a sequence of DTLS
   Records on any stream is a protocol violation. The receiver MAY
   terminate the SCTP association due to this protocol violation.

## Payload Protocol Identifier Usage

   SCTP Payload Protocol Identifiers are assigned by IANA.
   Application protocols using DTLS over SCTP SHOULD register and use
   a separate Payload Protocol Identifier (PPID) and SHOULD NOT reuse
   the PPID that they registered for running directly over SCTP.

   Using the same PPID does not harm as long as the application can
   determine whether or not DTLS is used.  However, for protocol
   analyzers, for example, it is much easier if a separate PPID is used.

   This means, in particular, that there is no specific PPID for DTLS.

## Stream Usage {#Stream-Usage}

   All DTLS Handshake, Alert, or ChangeCipherSpec (DTLS 1.2 only)
   messages MUST be transported on stream 0 with unlimited reliability
   and with the ordered delivery feature.

   DTLS messages of the record protocol, which carries the protected
   user messages, SHOULD use multiple streams other than stream 0;
   they MAY use stream 0.
   On stream 0 protected user messages as well as any DTLS
   messages that aren't record protocol will be mixed, thus the additional
   head of line blocking can occur.

## Chunk Handling

   DATA chunks of SCTP MUST be sent in an authenticated way as
   described in {{RFC4895}}.  All other chunks that can be
   authenticated, i.e., all chunk types that can be listed in the Chunk
   List Parameter {{RFC4895}}, MUST also be sent in an authenticated
   way.  This makes sure that an attacker cannot modify the stream in
   which a message is sent or affect the ordered/unordered delivery of
   the message.

   If PR-SCTP as defined in {{RFC3758}} is used, FORWARD-TSN chunks
   MUST also be sent in an authenticated way as described in
   {{RFC4895}}.  This makes sure that it is not possible for an
   attacker to drop messages and use forged FORWARD-TSN, SACK, and/or
   SHUTDOWN chunks to hide this dropping.

   I-DATA chunk type as defined in {{RFC8260}} is RECOMMENDED to be
   supported to avoid some of the down sides that large user messages
   have on blocking transmission of later arriving high priority user
   messages. However, the support is not mandated and negotiated
   independently from DTLS/SCTP. If I-DATA chunks are used, then
   they MUST be sent in an authenticated way as described in
   {{RFC4895}}.

## SCTP-AUTH Hash Function

   When using DTLS/SCTP, the SHA-256 Message Digest Algorithm MUST be
   supported in the SCTP-AUTH {{RFC4895}} implementation. SHA-1 MUST
   NOT be used when using DTLS/SCTP. {{RFC4895}} requires support and
   inclusion of SHA-1 in the HMAC-ALGO parameter, thus, to meet
   both requirements the HMAC-ALGO parameter will include both SHA-256
   and SHA-1 with SHA-256 listed prior to SHA-1 to indicate the
   preference.

## Renegotiation

   DTLS 1.2 renegotiation enables rekeying (with ephemeral
   Diffie-Hellman) of both DTLS and SCTP-AUTH as well as mutual
   reauthentication inside an DTLS 1.2 connection. It is up to the
   upper layer to use/allow it or not.  Application writers should be
   aware that allowing renegotiations may result in changes of
   security parameters. Renegotiation has been removed from DTLS 1.3
   and partly replaced with post-handshake messages such as
   KeyUpdate. See {{sec-Consideration}} for security considerations
   regarding rekeying.

### DTLS 1.2 Considerations

   Before sending during renegotiation a ClientHello message or ServerHello
   message, the DTLS endpoint MUST ensure that all DTLS messages using the
   previous epoch have been acknowledged by the SCTP peer in a non-revokable
   way.

   Prior to processing a received ClientHello message or ServerHello
   message, all other received SCTP user messages that are buffered in the
   SCTP layer and can be delivered to the DTLS layer MUST be read and
   processed by DTLS.

### DTLS 1.3 Considerations

   Before sending a KeyUpdate message, the DTLS endpoint MUST ensure that
   all DTLS messages have been acknowledged by the SCTP peer in a non-revokable
   way. After sending the KeyUpdate message, it stops sending DTLS messages until
   the corresponding Ack message has been processed.

   Prior to processing a received KeyUpdate message, all other received SCTP
   user messages that are buffered in the SCTP layer and can be delivered to
   the DTLS layer MUST be read and processed by DTLS.

## DTLS Epochs

   In general, DTLS implementations SHOULD discard records from earlier epochs.
   However, in the context of a reliable communication this is not appropriate.

### DTLS 1.2 Considerations

   The procedures of Section 4.1 of {{RFC6347}} MUST NOT be followed.
   Instead, when currently using epoch n, for n > 1, DTLS packets from epoch
   n - 1 and n MUST be processed.

### DTLS 1.3 Considerations

   The procedures of Section 4.2.1 of {{I-D.ietf-tls-dtls13}} are irrelevant.
   When receiving DTLS packets using epoch n, no DTLS packets from earlier
   epochs are received.

## Handling of Endpoint-Pair Shared Secrets

   SCTP-AUTH {{RFC4895}} is keyed using Endpoint-Pair Shared
   Secrets. In SCTP associations where DTLS is used, DTLS is used to
   establish these secrets. The endpoints MUST NOT use another
   mechanism for establishing shared secrets for SCTP-AUTH.
   The endpoint-pair shared secret for Shared Key Identifier 0 is
   empty and MUST be used when establishing a DTLS connection.

### DTLS 1.2 Considerations

   Whenever the master secret changes, a 64-byte shared secret is derived from
   every master secret and provided as a new endpoint-pair shared secret by
   using the TLS-Exporter described in {{RFC5705}}.
   The 64-byte shared secret MUST be provided to the SCTP stack as soon as
   the computation is possible.
   The exporter MUST use the label given in Section {{IANA-Consideration}}
   and no context.
   The new Shared Key Identifier MUST be the old Shared Key Identifier
   incremented by 1.

   After sending the DTLS Finished message, the active SCTP-AUTH key
   MUST be switched to the new one.

   Once the initial Finished message from the peer has been processed by DTLS,
   the SCTP-AUTH key with Shared Key Identifier 0 MUST be removed.
   Once the Finished message using DTLS epoch n with n > 2 has been processed
   by DTLS, the SCTP-AUTH key with Shared Key Identifier n - 2 MUST be removed.

### DTLS 1.3 Considerations

   When the exporter_secret can be computed, a 64-byte shared secret is derived
   from it and provided as a new endpoint-pair shared secret by using the
   TLS-Exporter described in {{RFC8446}}.
   The 64-byte shared secret MUST be provided to the SCTP stack as soon as
   the computation is possible.
   The exporter MUST use the label given in Section {{IANA-Consideration}}
   and no context.
   This shared secret MUST use Shared Key Identifier 1.

   After sending the DTLS Finished message, the active SCTP-AUTH key
   MUST be switched to use Shared Key Identifier 1.

   Once the Finished message from the peer has been processed by DTLS,
   the SCTP-AUTH key with Shared Key Identifier 0 MUST be removed.
   
## Shutdown

   To prevent DTLS from discarding DTLS user messages while it is
   shutting down, a CloseNotify message MUST only be sent after all
   outstanding SCTP user messages have been acknowledged by the SCTP
   peer in a non-revokable way.

   Prior to processing a received CloseNotify, all other received SCTP
   user messages that are buffered in the SCTP layer MUST be read and
   processed by DTLS.

# DTLS over SCTP Service {#Negotiation}

   The adoption of DTLS over SCTP according to the current description
   is meant to add to SCTP the option for transferring encrypted data.
   When DTLS over SCTP is used, all data being transferred MUST be
   protected by chunk authentication and DTLS encrypted.  Chunks that
   need to be received in an authenticated way will be specified
   in the CHUNK list parameter according to {{RFC4895}}.
   Error handling for authenticated chunks is according to {{RFC4895}}.

## Adaptation Layer Indication in INIT/INIT-ACK

   At the initialization of the association, a sender of the INIT or
   INIT ACK chunk that intends to use DTLS/SCTP as specified in this
   specification MUST include an Adaptation Layer Indication Parameter
   with the IANA assigned value TBD ({{sec-IANA-ACP}}) to inform its
   peer that it is able to support DTLS over SCTP per this
   specification.


## DTLS over SCTP Initialization {#DTLS-init}

   Initialization of DTLS/SCTP requires all the following options to
   be part of the INIT/INIT-ACK handshake:

   RANDOM: defined in {{RFC4895}}

   CHUNKS: list of permitted chunks, defined in {{RFC4895}}

   HMAC-ALGO: defined in {{RFC4895}}

   ADAPTATION-LAYER-INDICATION: defined in {{RFC5061}}

   When all the above options are present, the Association will start
   with support of DTLS/SCTP.  The set of options indicated are the
   DTLS/SCTP Mandatory Options.  No data transfer is permitted before
   DTLS handshake is complete. Chunk bundling is permitted according
   to {{RFC4960}}. The DTLS handshake will enable authentication of
   both the peers and also have the declare their support message
   size.

   The extension described in this document is given by the following
   message exchange.

~~~~~~~~~~~
   --- INIT[RANDOM; CHUNKS; HMAC-ALGO; ADAPTATION-LAYER-IND] --->
   <- INIT-ACK[RANDOM; CHUNKS; HMAC-ALGO; ADAPTATION-LAYER-IND] -
   ------------------------ COOKIE-ECHO ------------------------>
   <------------------------ COOKIE-ACK -------------------------
   ---------------- AUTH; DATA[DTLS Handshake] ----------------->
                               ...
                               ...
   <--------------- AUTH; DATA[DTLS Handshake] ------------------
~~~~~~~~~~~

## Client Use Case

   When a client initiates an SCTP Association with DTLS protection,
   i.e., the SCTP INIT containing DTSL/SCTP Mandatory Options, it can
   receive an INIT-ACK also containing DTLS/SCTP Mandatory Options, in
   that case the Association will proceed as specified in the previous
   {{DTLS-init}} section.  If the peer replies with an INIT-ACK not
   containing all DTLS/SCTP Mandatory Options, the client SHOULD reply
   with an SCTP ABORT.

## Server Use Case

   If a SCTP Server supports DTLS/SCTP, i.e., per this specification,
   when receiving an INIT chunk with all DTLS/SCTP Mandatory Options
   it will reply with an INIT-ACK also containing all the DTLS/SCTP
   Mandatory Options, following the sequence for DTLS initialization
   {{DTLS-init}} and the related traffic case.  If a SCTP Server that
   supports DTLS and configured to use it, receives an INIT chunk
   without all DTLS/SCTP Mandatory Options, it SHOULD reply with an
   SCTP ABORT.

## RFC 6083 Fallback {#Fallback}

   This section discusses how an endpoint supporting this
   specification can fallback to follow the DTLS/SCTP behavior in RFC6083.
   It is recommended to define a setting that represents the
   policy to allow fallback or not. However, the possibility to use
   fallback is based on the ULP can operate using user messages that
   are no longer than 16383 bytes and where the security issues can be
   mitigated or considered acceptable. Fallback is NOT RECOMMEND to be
   enabled as it enables downgrade attacks to weaker algorithms and versions
   of DTLS.

   An SCTP endpoint that receives an INIT chunk or an INIT-ACK chunk that does
   not contain the SCTP-Adaptation-Indication parameter with the  DTLS/SCTP
   adaptation layer codepoint, see {{sec-IANA-ACP}}, may in certain cases
   potentially perform a fallback to RFC 6083 behavior.
   However, the fallback attempt should only be performed if policy says that
   is acceptable.

   If fallback is allowed, it is possible that the client will send
   plain text user messages prior to DTLS handshake as it is allowed
   per RFC 6083.  So that needs to be part of the consideration for a
   policy allowing fallback. 

# Socket API Considerations

   This section describes how the socket API defined in {{RFC6458}} is extended
   to provide a way for the application to observe the HMAC algorithms used for
   sending and receiving of AUTH chunks.

   Please note that this section is informational only.

   A socket API implementation based on {{RFC6458}} is, by means of the
   existing SCTP_AUTHENTICATION_EVENT event, extended to provide the event
   notification whenever a new HMAC algorithm is used in a received AUTH chunk.

   Furthermore, two new socket options for the level IPPROTO_SCTP and the
   name SCTP_SEND_HMAC_IDENT and SCTP_EXPOSE_HMAC_IDENT_CHANGES are defined
   as described below.
   The first socket option is used to query the HMAC algorithm used for sending
   AUTH chunks.
   The second enables the monitoring of HMAC algorithms used in received
   AUTH chunks via the SCTP_AUTHENTICATION_EVENT event.

   Support for the SCTP_SEND_HMAC_IDENT and SCTP_EXPOSE_HMAC_IDENT_CHANGES
   socket options also needs to be added to the function sctp_opt_info().

## Socket Option to Get the HMAC Identifier being Sent (SCTP_SEND_HMAC_IDENT)

   During the SCTP association establishment a HMAC Identifier is selected
   which is used by an SCTP endpoint when sending AUTH chunks.
   An application can access the result of this selection by using this
   read-only socket option, which uses the level IPPROTO_SCTP and the name
   SCTP_SEND_HMAC_IDENT.

   The following structure is used to access HMAC Identifier used for sending
   AUTH chunks:

~~~~~~~~~~~
struct sctp_assoc_value {
    sctp_assoc_t assoc_id;
    uint32_t assoc_value;
};
~~~~~~~~~~~

   assoc_id:
   :  This parameter is ignored for one-to-one style sockets.
      For one-to-many style sockets, the application fills in an association
      identifier.
      It is an error to use SCTP_{FUTURE|CURRENT|ALL}_ASSOC in assoc_id.

   assoc_value:
   :  This parameter contains the HMAC Identifier used for sending AUTH chunks.

## Exposing the HMAC Identifiers being Received {#API-SCTP-AUTHENTICATION-EVENT}

   Section 6.1.8 of {{RFC6458}} defines the SCTP_AUTHENTICATION_EVENT event,
   which uses the following structure:

~~~~~~~~~~~
struct sctp_authkey_event {
    uint16_t auth_type;
    uint16_t auth_flags;
    uint32_t auth_length;
    uint16_t auth_keynumber;
    uint32_t auth_indication;
    sctp_assoc_t auth_assoc_id;
};
~~~~~~~~~~~

   This document updates this structure to

~~~~~~~~~~~
struct sctp_authkey_event {
    uint16_t auth_type;
    uint16_t auth_flags;
    uint32_t auth_length;
    uint16_t auth_identifier; /* formerly auth_keynumber */
    uint32_t auth_indication;
    sctp_assoc_t auth_assoc_id;
};
~~~~~~~~~~~

   by renaming auth_keynumber to auth_identifier.
   auth_identifier just replaces auth_keynumber in the context of {{RFC6458}}.
   In addition to that, the SCTP_AUTHENTICATION_EVENT event is extended to
   also indicate when a new HMAC Identifier is received and such reporting is
   explicitly enabled as described in {{API-SCTP-EXPOSE-HMAC-IDENT-CHANGES}}.
   In this case auth_indication is SCTP_AUTH_NEW_HMAC and the new
   HMAC identifier is reported in auth_identifier.

## Socket Option to Expose HMAC Identifier Usage (SCTP_EXPOSE_HMAC_IDENT_CHANGES) {#API-SCTP-EXPOSE-HMAC-IDENT-CHANGES}

   This options allows the application to enable and disable the reception of
   SCTP_AUTHENTICATION_EVENT events when a new HMAC Identifiers has been
   received in an AUTH chunk (see {{API-SCTP-AUTHENTICATION-EVENT}}).
   This read/write socket option uses the level IPPROTO_SCTP and the name
   SCTP_EXPOSE_HMAC_IDENT_CHANGES.
   It is needed to provide backwards compatibility and the default is that
   these events are not reported.

   The following structure is used to enable or disable the reporting of newly
   received HMAC Identifiers in AUTH chunks:

~~~~~~~~~~~
struct sctp_assoc_value {
    sctp_assoc_t assoc_id;
    uint32_t assoc_value;
};
~~~~~~~~~~~

   assoc_id:
   :  This parameter is ignored for one-to-one style sockets.
      For one-to-many style sockets, the application may fill in an association
      identifier or SCTP_{FUTURE|CURRENT|ALL}_ASSOC.

   assoc_value:
   :  Newly received HMAC Identifiers are reported if, and only if, this
      parameter is non-zero.

# IANA Considerations {#IANA-Consideration}

## TLS Exporter Label

   RFC 6083 defined a TLS Exporter Label registry as described in
   {{RFC5705}}. IANA is requested to update the reference for the
   label "EXPORTER_DTLS_OVER_SCTP" to this specification.

## SCTP Adaptation Layer Indication Code Point {#sec-IANA-ACP}

   {{RFC5061}} defined a IANA registry for Adaptation Code Points to
   be used in the Adaptation Layer Indication parameter. The registry
   was at time of writing located:
   https://www.iana.org/assignments/sctp-parameters/sctp-parameters.xhtml#sctp-parameters-27
   IANA is requested to assign one Adaptation Code Point for DTLS/SCTP
   per the below proposed entry in {{iana-ACP}}.


| Code Point (32-bit number) | Description | Reference |
| -------------------------- | ----------- | --------- |
| 0x00000002 | DTLS/SCTP | \[RFC-TBD\] |
{: #iana-ACP title="Adaptation Code Point"}

RFC-Editor Note: Please replace \[RFC-TBD\] with the RFC number given to
this specification.

# Security Considerations {#sec-Consideration}

   The security considerations given in {{I-D.ietf-tls-dtls13}},
   {{RFC4895}}, and {{RFC4960}} also apply to this document.

## Cryptographic Considerations

   Over the years, there have been several serious attacks on earlier
   versions of Transport Layer Security (TLS), including attacks on
   its most commonly used ciphers and modes of operation.  {{RFC7457}}
   summarizes the attacks that were known at the time of publishing
   and BCP 195 {{RFC7525}} provides recommendations for improving the
   security of deployed services that use TLS.

   When DTLS/SCTP is used with DTLS 1.2 {{RFC6347}}, DTLS 1.2 MUST be
   configured to disable options known to provide insufficient
   security. HTTP/2 {{RFC7540}} gives good minimum requirements based
   on the attacks that where publicly known in 2015. DTLS 1.3
   {{I-D.ietf-tls-dtls13}} only define strong algorithms without major
   weaknesses at the time of publication. Many of the TLS registries
   have a "Recommended" column. Parameters not marked as "Y" are NOT
   RECOMMENDED to support.

   DTLS 1.3 requires rekeying before algorithm specific AEAD limits
   have been reached. The AEAD limits equations are equally valid for
   DTLS 1.2 and SHOULD be followed for DTLS/SCTP, but are not mandated
   by the DTLS 1.2 specification.

   HMAC-SHA-256 as used in SCTP-AUTH has a very large tag length and
   very good integrity properties. The SCTP-AUTH key can be used until
   the DTLS handshake is re-run at which point a new SCTP-AUTH key is
   derived using the TLS-Exporter. As discussed below DTLS 1.3 does
   not currently support renegotiation and lacks the capability of
   updating the SCTP-AUTH key.

   DTLS/SCTP is in many deployments replacing IPsec. For IPsec, NIST
   (US), BSI (Germany), and ANSSI (France) recommends very frequent
   re-run of Diffie-Hellman to provide Perfect Forward Secrecy. ANSSI
   writes "It is recommended to force the periodic renewal of the
   keys, e.g., every hour and every 100 GB of data, in order to limit
   the impact of a key compromise." {{ANSSI-DAT-NT-003}}.

   For many DTLS/SCTP deployments the DTLS connections are expected to
   have very long lifetimes of months or even years. For connections
   with such long lifetimes there is a need to frequently
   re-authenticate both client and server.

   When using DTLS 1.2 {{RFC6347}}, AEAD limits, frequent
   re-authentication and frequent re-run of Diffie-Hellman can be
   achieved with frequent renegotiation, see TLS 1.2 {{RFC5246}}. When
   renegotiation is used both clients and servers MUST use the
   renegotiation_info extension {{RFC5746}} and MUST follow the
   renegotiation guidelines in BCP 195 {{RFC7525}}. In particular,
   both clients and servers MUST NOT accept a change of identity
   during renegotiation.

   In DTLS 1.3 renegotiation has been removed from DTLS 1.3 and partly
   replaced with Post-Handshake KeyUpdate. When using DTLS 1.3
   {{I-D.ietf-tls-dtls13}}, AEAD limits and frequent rekeying can be
   achieved by sending frequent post-handshake KeyUpdate
   messages. Symmetric rekeying gives less protection against key
   leakage than re-running Diffie-Hellman.  After leakage of
   application_traffic_secret_N, a passive attacker can passively
   eavesdrop on all future application data sent on the connection
   including application data encrypted with
   application_traffic_secret_N+1, application_traffic_secret_N+2,
   etc. The is no way to do post-handshake server authentication or
   Ephemeral Diffie-Hellman inside a DTLS 1.3 connection. Note that
   KeyUpdate does not update the exporter_secret.

   For upper layer protocols where frequent re-run of Diffie-Hellman,
   rekeying of SCTP-AUTH, and server reauthentication is required and
   creating a new SCTP connection with DTLS 1.3 to replace the current
   is not a viable option it is RECOMMENDED to use DTLS 1.2.

## Downgrade Attacks

   A peer supporting DTLS/SCTP according to this specification,
   DTLS/SCTP according to {{RFC6083}} and/or SCTP without DTLS may be
   vulnerable to downgrade attacks where on on-path attacker
   interferes with the protocol setup to lower or disable security. If
   possible, it is RECOMMENDED that the peers have a policy only
   allowing DTLS/SCTP according to this specification.

## Authentication and Policy Decisions

   DTLS/SCTP MUST be mutually authenticated. It is RECOMMENDED that
   DTLS/SCTP is used with certificate-based authentication.  All
   security decisions MUST be based on the peer's authenticated
   identity, not on its transport layer identity.

   It is possible to authenticate DTLS endpoints based on IP addresses
   in certificates. SCTP associations can use multiple IP addresses
   per SCTP endpoint. Therefore, it is possible that DTLS records will
   be sent from a different source IP address or to a different
   destination IP address than that originally authenticated. This is
   not a problem provided that no security decisions are made based on
   the source or destination IP addresses.

## Privacy Considerations

   {{RFC6973}} suggests that the privacy considerations of IETF
   protocols be documented.

   For each SCTP user message, the user also provides a stream
   identifier, a flag to indicate whether the message is sent ordered
   or unordered, and a payload protocol identifier.  Although
   DTLS/SCTP provides privacy for the actual user message, the other
   three information fields are not confidentiality protected.  They
   are sent as cleartext because they are part of the SCTP DATA
   chunk header.

   It is RECOMMENDED that DTLS/SCTP is used with certificate based
   authentication in DTLS 1.3 {{I-D.ietf-tls-dtls13}} to provide
   identity protection. DTLS/SCTP MUST be used with a key exchange
   method providing Perfect Forward Secrecy. Perfect Forward Secrecy
   significantly limits the amount of data that can be compromised due
   to key compromise.

## Pervasive Monitoring

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
   other protocols increase the risk of identifying individual users.

# Acknowledgments

   The authors of RFC 6083 which this document is based on are
   Michael Tüxen, Eric Rescorla, and Robin Seggelmann.

   The RFC 6083 authors thanked Anna Brunstrom, Lars Eggert, Gorry
   Fairhurst, Ian Goldberg, Alfred Hoenes, Carsten Hohendorf, Stefan
   Lindskog, Daniel Mentz, and Sean Turner for their invaluable
   comments.

   The authors of this document want to thank GitHub user vanrein for
   their contribution.

--- back

# Motivation for Changes

This document proposes a number of changes to RFC 6083 that have
various different motivations:

Supporting Large User Messages: RFC 6083 allowed only user messages
   that could fit within a single DTLS record. 3GPP has run into this
   limitation where they have at least four SCTP using protocols (F1,
   E1, Xn, NG-C) that can potentially generate messages over the size
   of 16384 bytes.

New Versions: Almost 10 years has passed since RFC 6083 was written,
   and significant evolution has happened in the area of DTLS and
   security algorithms. Thus DTLS 1.3 is the newest version of DTLS
   and also the SHA-1 HMAC algorithm of RFC 4895 is getting towards
   the end of usefulness. Thus, this document mandates usage of
   relevant versions and algorithms.

Clarifications: Some implementation experiences have been gained that
   motivates additional clarifications on the specification.

* Avoid unsecured messages prior to DTLS handshake have completed.

* Make clear that all messages are encrypted after DTLS handshake.
