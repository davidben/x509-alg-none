---
title: "Unsigned X.509 Certificates"
category: std

docname: draft-davidben-x509-alg-none-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "lamps"

author:
 -
    ins: "D. Benjamin"
    name: "David Benjamin"
    organization: "Google LLC"
    email: davidben@google.com

informative:
  JWT:
    title: How Many Days Has It Been Since a JWT alg:none Vulnerability?
    target: https://www.howmanydayssinceajwtalgnonevuln.com/
    date: 2024-10-09
    author:
    -
      ins: "J. Sanderson"
      name: "James 'zofrex' Sanderson"

--- abstract

This document defines a placeholder X.509 signature algorithm that may be used
in contexts where the consumer of the certificate does not intend to verify the
signature.

--- middle

# Introduction

An X.509 certificate {{!RFC5280}} relates two entities in the PKI: information
about a subject and a proof from an issuer. Some applications, however, only
require subject information. For example, an X.509 trust anchor is described by
information about the subject (a root certification authority, or root CA). The
relying party trusts this information out-of-band and does not require an
issuer's signature.

X.509 does not define such a structure. Instead, X.509 trust anchors often use
"self-signed" certificates, where the CA's key is used to sign the certificate.
Other formats, such as {{?RFC5914}} exist to convey trust anchors, but
self-signed certificates remain widely used. Additionally, some TLS {{?RFC8446}}
server deployments use self-signed certificates when they do not intend to
present a CA-issued identity, instead expecting the relying party to
authenticate the certificate out-of-band, e.g. via a known fingerprint.

These self-signatures typically have no security value, aren't checked by
the receiver, and only serve as placeholders to meet syntactic requirements of
an X.509 certificate.

Computing signatures as placeholders has some drawbacks:

* Post-quantum signature algorithms are large, so including a self-signature
  significantly increases the size of the payload.

* If the subject is an end entity, rather than a CA, computing an X.509
  signature risks cross-protocol attacks with the intended use of the key.

* It is ambiguous whether such a self-signature requires the CA bit in basic
  constraints or keyCertSign in key usage. If the key is intended for a
  non-X.509 use, asserting those capabilities is an unnecessary risk.

* If end entity's key is not a signing key (e.g. a KEM key), there is no valid
  signature algorithm to use with the key.

This document defines a profile for unsigned X.509 certificates, which may be
used when the certificate is used as a container for subject information,
without any specific issuer.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Constructing Unsigned Certificates

This document defines the id-unsigned OID:

~~~
id-unsigned OBJECT IDENTIFER ::= { TBD }
~~~

To construct an unsigned X.509 certificate, the sender MUST set the
Certificate's signatureAlgorithm and TBSCertificate's signature fields each to
an AlgorithmIdentifier with algorithm id-unsigned and omitted parameters. The
Certificate's signatureValue field MUST be an empty BIT STRING.

# Consuming Unsigned Certificates

Signatures of type id-unsigned are always invalid. This contrasts with {{JWT}}.
When processing X.509 certificates without verifying signatures, receivers MAY
accept id-unsigned. When verifying X.509 signatures, receivers MUST reject
id-unsigned. In particular, X.509 validators MUST NOT accept id-unsigned in the
place of a signature in the certification path.

X.509 applications must already account for unknown signature algorithms, so
applications are RECOMMENDED to satisfy these requirements by ignoring this
document. An unmodified X.509 validator will not recognize id-unsigned and is
thus already expected to reject it in the certification path.

# Security Considerations

If an application uses a self-signature when constructing a subject-only
certificate for a non-X.509 key, the X.509 signature payload and those of the
key's intended use may collide. The self-signature might then be used as part of
a cross-protocol attack. Using id-unsigned avoids a single key being used for
both X.509 and the end-entity protocol, eliminating this risk.

If an application accepts id-unsigned as part of a certification path, or in any
other context where it is necessary to verify the X.509 signature, the signature
check would be bypassed. Thus, {{consuming-unsigned-certificates}} prohibits
this and recommends that applications not treat id-unsigned differently from any
other previously unrecognized signature algorithm. Non-compliant applications
that instead accept id-unsigned as a valid signature risk of vulnerabilities
analogous to {{JWT}}.

# IANA Considerations

This document has no IANA actions.

--- back

# Acknowledgements
{:numbered="false"}

Thanks to Bob Beck, Nick Harper, and Sophie Schmieg for reviewing an early
iteration of this document.
