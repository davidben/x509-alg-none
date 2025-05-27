---
title: "Unsigned X.509 Certificates"
category: std

docname: draft-ietf-lamps-x509-alg-none-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Limited Additional Mechanisms for PKIX and SMIME"
keyword:
- self-signed certificate
venue:
  group: "Limited Additional Mechanisms for PKIX and SMIME"
  type: "Working Group"
  mail: "spasm@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/spasm/"
  github: "davidben/x509-alg-none"
  latest: "https://davidben.github.io/x509-alg-none/draft-ietf-lamps-x509-alg-none.html"

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
in contexts where the consumer of the certificate is not expected to verify the
signature.

--- middle

# Introduction

An X.509 certificate {{!RFC5280}} relates two entities in the PKI: information
about a subject and a proof from an issuer. Viewing the PKI as a graph with
entities as nodes, as in {{?RFC4158}}, a certificate is an edge between the
subject and issuer.

In some contexts, an application needs standalone subject information instead of
a certificate. In the graph model, the application needs a node, not an edge.
For example, certification path validation ({{Section 6 of !RFC5280}}) begins at
a trust anchor, or root certification authority (root CA). The application
trusts this trust anchor information out-of-band and does not require an
issuer's signature.

X.509 does not define a structure for this scenario. Instead, X.509 trust
anchors are often represented with "self-signed" certificates, where the
subject's key signs over itself. Other formats, such as {{?RFC5914}} exist to
convey trust anchors, but self-signed certificates remain widely used.

Additionally, some TLS {{?RFC8446}} server deployments use self-signed
end entity certificates when they do not intend to present a CA-issued
identity, instead expecting the relying party to authenticate the certificate
out-of-band, e.g. via a known fingerprint.

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

* If the subject is an end entity, and the end entity's key is not a signing
  key (e.g. a KEM key), there is no valid signature algorithm to use with the key.

This document defines a profile for unsigned X.509 certificates, which may be
used when the certificate is used as a container for subject information,
without any specific issuer.

# Conventions and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Constructing Unsigned Certificates

This document defines the id-alg-unsigned object identifier (OID) under the OID arc
defined in {{!RFC8411}}:

~~~
  id-alg-unsigned OBJECT IDENTIFIER ::= {1 3 6 1 5 5 7 6 36}
~~~

To construct an unsigned X.509 certificate, the sender MUST set the
Certificate's signatureAlgorithm and TBSCertificate's signature fields each to
an AlgorithmIdentifier with algorithm id-alg-unsigned. The parameters for
id-alg-unsigned MUST be omitted. The Certificate's signatureValue field MUST be
a BIT STRING of length zero.

An unsigned certificate has no issuer, so there are no meaningful values to use for
the issuer and issuerUniqueID fields and the authority key identifier and issuer
alternative name extensions. This document does not mandate particular values
but gives general guidance: Senders SHOULD omit issuerUniqueID, authority key
identifier, and issuer alternative name, unless needed for compatibility with
existing applications.

{{Section 4.1.2.4 of !RFC5280}} does not permit empty issuers, so such a value
may not be interoperable with existing applications. Senders MAY use the subject
field (if the subject is not empty), as in a self-signed certificate.

Senders MAY alternatively use a short placeholder issuer consisting of a single
relative distinguished name, with a single attribute of type id-alg-unsigned and
value a zero-length UTF8String. This placeholder name, in the string
representation of {{?RFC2253}}, is:

~~~
1.3.6.1.5.5.7.6.36=
~~~

# Consuming Unsigned Certificates

X.509 signatures of type id-alg-unsigned are always invalid. This contrasts
with {{JWT}}. When processing X.509 certificates without verifying signatures,
receivers MAY accept id-alg-unsigned. When verifying X.509 signatures,
receivers MUST reject id-alg-unsigned. In particular, X.509 validators MUST
NOT accept id-alg-unsigned in the place of a signature in the certification
path.

X.509 applications must already account for unknown signature algorithms, so
applications are RECOMMENDED to satisfy these requirements by ignoring this
document. An unmodified X.509 validator will not recognize id-alg-unsigned
and is thus already expected to reject it in the certification path. Conversely,
in contexts where an X.509 application was ignoring the self-signature,
id-alg-unsigned will also be ignored, but more efficiently.

In other contexts, applications may require modifications. For example, an
application that uses self-signedness in interpreting its local configuration
may need to modify its configuration model or user interface before using an
unsigned certificate as a trust anchor.

# Security Considerations

If an application uses a self-signature when constructing a subject-only
certificate for a non-X.509 key, the X.509 signature payload and those of the
key's intended use may collide. The self-signature might then be used as part of
a cross-protocol attack. Using id-alg-unsigned avoids a single key being used
for both X.509 and the end-entity protocol, eliminating this risk.

If an application accepts id-alg-unsigned as part of a certification path, or
in any other context where it is necessary to verify the X.509 signature, the
signature check would be bypassed. Thus, {{consuming-unsigned-certificates}}
prohibits this and recommends that applications not treat id-alg-unsigned
differently from any other previously unrecognized signature algorithm.
Non-compliant applications that instead accept id-alg-unsigned as a valid
signature risk of vulnerabilities analogous to {{JWT}}.

# IANA Considerations

IANA is requested to create the following entry in the SMI Security for PKIX Module Identifier registry, defined by {{!RFC7299}}:

| Decimal | Description             | References |
|---------|-------------------------|------------|
| TBD     | id-mod-algUnsigned-2025 | [this-RFC] |

IANA is requested to add the following entry to the
"SMI Security for PKIX Algorithms" registry {{!RFC7299}}:

| Decimal | Description     | References |
|---------|-----------------|------------|
| 36      | id-alg-unsigned | [this-RFC] |

--- back

# ASN.1 Module

~~~
SignatureAlgorithmNone
  { iso(1) identified-organization(3) dod(6) internet(1)
    security(5) mechanisms(5) pkix(7) id-mod(0)
    id-mod-algUnsigned-2025(TBD) }

DEFINITIONS IMPLICIT TAGS ::=
BEGIN

IMPORTS
  SIGNATURE-ALGORITHM
  FROM AlgorithmInformation-2009  -- in [RFC5912]
    { iso(1) identified-organization(3) dod(6) internet(1)
      security(5) mechanisms(5) pkix(7) id-mod(0)
      id-mod-algorithmInformation-02(58) } ;

-- Unsigned Signature Algorithm

id-alg-unsigned OBJECT IDENTIFIER ::= { iso(1)
   identified-organization(3) dod(6) internet(1) security(5)
   mechanisms(5) pkix(7) alg(6) 36 }

sa-unsigned SIGNATURE-ALGORITHM ::= {
   IDENTIFIER id-alg-unsigned
   PARAMS ARE absent
}

END
~~~

# Acknowledgements
{:numbered="false"}

Thanks to Bob Beck, Nick Harper, and Sophie Schmieg for reviewing an early
iteration of this document. Thanks to Alex Gaynor for providing a link to cite
for {{JWT}}. Thanks to Russ Housley for additional input.
