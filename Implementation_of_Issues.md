* Issue #11 - removed reference to Partial message API *** Done ***

* Issue #12 - updated by John and merged 327dcda

* Issue #13 - Updated by John and merged 327dcda

* Issue #14 - The drying game solves the issue partially. This can be improved but afaik the solution is implementation dependent. I'd drop the issue together with issue #25

* Issue #15 - Was the Adaptation Layer Indication request fed to IANA?

* Issue #16 - AUTH related SHA1/SHA256 - I think it's ok as it is

* Issue #22 - Updated by John and merged 327dcda

* Issue #23 - Updated by John and merged 327dcda

* Issue #24 - Updated by John and merged 327dcda

* Issue #25 - See #14

* Issue #28 - Open: text is to be added to explain that In cases where not all DTLS records of a protected user messages is delivered it is vital that the DTLS layer knows that and either delivers nothing (atomic operations) or indicates that only partial delivery have occurred if some parts of the user message has been decrypted and delivered. *** Done ***

* Issue #31 - Updated by John and merged 327dcda

* Issue #35 - Open: text is needed for explicitly note that STARTTLS will not work with this specification (rfc3788). *** Done ***

* Issue #37 - Decision is needed if protocol violation answer with ABORT is MAY, RECOMMENDED or MUST

* Issue #38 - Should we add in rfc a method that limits the message size to the UL or shall we keep implementation specific?

