---
v: 3

title: An Architecture for Trustworthy and Transparent Digital Supply Chains
abbrev: SCITT Architecture
docname: draft-ietf-scitt-architecture-latest

area: Security
wg: SCITT
kw: Internet-Draft
cat: std
consensus: yes
submissiontype: IETF

kramdown_options:
  auto_id_prefix: sec-

venue:
  group: SCITT
  mail: scitt@ietf.org
  github: ietf-wg-scitt/draft-ietf-scitt-architecture

author:
- name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- name: Antoine Delignat-Lavaud
  organization: Microsoft Research
  street: 21 Station Road
  code: 'CB1 2FB'
  city: Cambridge
  email: antdl@microsoft.com
  country: UK
- name: Cedric Fournet
  organization: Microsoft Research
  street: 21 Station Road
  code: 'CB1 2FB'
  city: Cambridge
  email: fournet@microsoft.com
  country: UK
- name: Yogesh Deshpande
  organization: ARM
  street: 110 Fulbourn Road
  code: 'CB1 9NJ'
  city: Cambridge
  email: yogesh.deshpande@arm.com
  country: UK
- ins: S. Lasker
  name: Steve Lasker
  org: DataTrails
  email: steve.lasker@datatrails.ai
  code: '98199'
  city: Seattle
  region: WA
  country: United States

contributor:
  - ins: O. Steele
    name: Orie Steele
    organization: Transmute
    email: orie@transmute.industries
    country: United States
    contribution: >
      Orie contributed to improving the generalization of COSE building blocks and document consistency.

normative:
  RFC5280: PKIX
  RFC6838:
  RFC8610: CDDL
  RFC9052: COSE
  RFC9360:
  RFC8392:
  COSWID: RFC9393
  I-D.draft-ietf-cose-merkle-tree-proofs: COMETRE

  CWT_CLAIMS_COSE: I-D.ietf-cose-cwt-claims-in-headers
  IANA.cwt:
  IANA.named-information:
  RFC6570: URITemplate
  RFC4648: Base64Url

informative:

  I-D.draft-ietf-rats-eat: draft-ietf-rats-eat
  NIST.SP.1800-19:
  NIST.SP.800-63-3:
  FIPS.201: DOI.10.6028/NIST.FIPS.201-3
  ISO.17000.2020:
    target: https://www.iso.org/standard/73029.html
    title: ISO/IEC 17000:2020

  RFC4949: Glossary
  RFC7523:
  RFC8725:
  RFC2397: DataURLs
  RFC8141: URNs
  RFC9162: CT
  RFC9334: rats-arch
  CWT_CLAIMS:
    target: https://www.iana.org/assignments/cwt/cwt.xhtml
    title: CBOR Web Token (CWT) Claims

  CycloneDX:
    target: https://cyclonedx.org/specification/overview/
    title: CycloneDX

  EQUIVOCATION: DOI.10.1145/1323293.1294280

  in-toto:
    target: https://in-toto.io/
    title: in-toto

  MERKLE: DOI.10.1007/3-540-48184-2_32

  PBFT: DOI.10.1145/571637.571640

  SLSA:
    target: https://slsa.dev/
    title: SLSA

  SPDX-CBOR:
    target: https://spdx.dev/use/specifications/
    title: SPDX Specification

  SPDX-JSON:
    target: https://spdx.dev/use/specifications/
    title: SPDX Specification

  SWID:
    target: https://csrc.nist.gov/Projects/Software-Identification-SWID/guidelines
    title: SWID Specification

  URLs:
    target: https://url.spec.whatwg.org/
    title: URL Living Standard

  I-D.draft-ietf-core-href: CURIs

  KEY-MANAGEMENT: DOI.10.6028/NIST.SP.800-57pt2r1

--- abstract

Traceability of physical and digital Artifacts in supply chains is a long-standing, but increasingly serious security concern.
The rise in popularity of verifiable data structures as a mechanism to make actors more accountable for breaching their compliance promises has found some successful applications to specific use cases (such as the supply chain for digital certificates), but lacks a generic and scalable architecture that can address a wider range of use cases.

This document defines a generic, interoperable and scalable architecture to enable transparency across any supply chain with minimum adoption barriers.
It provides flexibility, enabling interoperability across different implementations of Transparency Services with various auditing and compliance requirements.
Issuers can register their Signed Statements on any Transparency Service, with the guarantee that any Relying Parties will be able to verify them.

--- middle

# Introduction

This document describes the generic, interoperable, and scalable SCITT architecture.
Its goal is to enhance auditability and accountability across supply chains.

In supply chains, downstream Artifacts are built upon upstream Artifacts.
The complexity of traceability and quality control for these supply chains increases with the number of Artifacts and parties contributing to them.
There are many parties who publish information about Artifacts:
For example, the original manufacturer may provide information about the state of the Artifact when it left the factory.
The shipping company may add information about the transport environment of the Artifact.
Compliance Auditors may provide information about their compliance assessment of the Artifact.
Security companies may publish vulnerability information about an Artifact.
Some of these parties may publish information about their analysis or use of an Artifact.

SCITT provides a way for Relying Parties to obtain this information in a way that is "transparent", that is, parties cannot lie about the information that they publish without it being detected.
SCITT achieves this by having producers publish information in a Transparency Service, where Relying Parties can check the information.

## Requirements Notation

{::boilerplate bcp14-tagged}

# Terminology {#terminology}

The terms defined in this section have special meaning in the context of Supply Chain Integrity, Transparency, and Trust, which are used throughout this document.
When used in text, the corresponding terms are capitalized.
To ensure readability, only a core set of terms is included in this section.

The terms "header", "payload", and "to-be-signed bytes" are defined in {{-COSE}}.

The term "claim" is defined in {{RFC8392}}.

Append-only Log (Ledger):

: the verifiable append-only data structure that stores Signed Statements in a Transparency Service, often referred to by the synonym Ledger.
SCITT supports multiple Ledger and Receipt formats to accommodate different Transparency Service implementations, and the proof types associated with different types of Append-only Logs.

Artifact:

: a physical or non-physical item that is moving along a supply chain.

Auditor:

: an entity that checks the correctness and consistency of all Transparent Statements issued by a Transparency Service.
An Auditor is an example of a specialized Relying Party.

Client:

: an application making protected Transparency Service resource requests on behalf of the resource owner and with its authorization.

Envelope:

: metadata, created by the Issuer to produce a Signed Statement.
The Envelope contains the identity of the Issuer and information about the Artifact, enabling Transparency Service Registration Policies to validate the Signed Statement.
A Signed Statement is a COSE Envelope wrapped around a Statement, binding the metadata in the Envelope to the Statement.
In COSE, an Envelope consists of a protected header (included in the Issuer's signature) and an unprotected header (not included in the Issuer's signature).

Equivocation:

: a state where it is possible for a Transparency Service to provide different views of its Append-only log to Relying Parties about the same Artifact {{EQUIVOCATION}}.

Issuer:

: an identifier representing an organization, device, user, or entity securing Statements about supply chain Artifacts.
An Issuer may be the owner or author of Artifacts, or an independent third party such as an Auditor, reviewer or an endorser.
In SCITT Statements and Receipts, the `iss` CWT Claim is a member of the COSE header parameter `15: CWT_Claims` within the protected header of a COSE Envelope.

Non-equivocation:

: a state where it is impossible for a Transparency Service to provide different views of its Append-only Log to Relying Parties about the same Artifact.
Over time, an Issuer may register new Signed Statements about an Artifact in a Transparency Service with new information.
However, the consistency of a collection of Signed Statements about the Artifact can be checked by all Relying Parties.

Receipt:

: a cryptographic proof that a Signed Statement is included in the Append-only Log.
Receipts are signed proofs of verifiable data-structure properties.
The types of Receipts MUST support inclusion proofs and MAY support other proof types, such as consistency proofs.

Registration:

: the process of submitting a Signed Statement to a Transparency Service, applying the Transparency Service's Registration Policy, adding to the Append-only Log, and producing a Receipt.

Registration Policy:

: the pre-condition enforced by the Transparency Service before registering a Signed Statement, based on information in the non-opaque header and metadata contained in its COSE Envelope.

Relying Party:

: a Relying Parties consumes Transparent Statements, verifying their proofs and inspecting the Statement payload, either before using corresponding Artifacts, or later to audit an Artifact's provenance on the supply chain.

Signed Statement:

: an identifiable and non-repudiable Statement about an Artifact signed by an Issuer.
In SCITT, Signed Statements are encoded as COSE signed objects; the `payload` of the COSE structure contains the issued Statement.

Statement:

: any serializable information about an Artifact.
To help interpretation of Statements, they must be tagged with a media type (as specified in {{RFC6838}}).
A Statement may represent a Software Bill Of Materials (SBOM) that lists the ingredients of a software Artifact, an endorsement or attestation about an Artifact, indicate the End of Life (EOL), redirection to a newer version,  or any content an Issuer wishes to publish about an Artifact.
The additional Statements about an Artifact are correlated by the Subject defined in the {{CWT_CLAIMS}} protected header.
The Statement is considered opaque to Transparency Service, and MAY be encrypted.

Subject:

: an identifier, defined by the Issuer, that represents the organization, device, user, entity, or Artifact about which Statements (and Receipts) are made and by which a logical collection of Statements can be grouped.
It is possible that there are multiple Statements about the same Artifact.
In these cases, distinct Issuers (`iss`) might agree to use the `sub` CWT Claim to create a coherent sequence of Signed Statements about the same Artifact and Relying Parties can leverage `sub` to ensure completeness and Non-equivocation across Statements by identifying all Transparent Statements associated to a specific Subject.

Transparency Service:

: an entity that maintains and extends the Append-only Log, and endorses its state.
A Transparency Service can be a complex system, requiring the Transparency Service to provide many security guarantees about its Append-only Log.
The identity of a Transparency Service is captured by a public key that must be known by Relying Parties in order to validate Receipts.

Transparent Statement:

: a Signed Statement that is augmented with a Receipt created via Registration in a Transparency Service.
The Receipt is stored in the unprotected header of COSE Envelope of the Signed Statement.
A Transparent Statement remains a valid Signed Statement, and may be registered again in a different Transparency Service.

Verifiable Data Structure:

: a data structure which supports one or more proof types, such as "inclusion proofs" or "consistency proofs" (as defined in {{-COMETRE}}).

{: #mybody}

# Definition of Transparency

In this document, the definition of transparency is intended to build over abstract notions of Append-only Logs and Receipts.
Existing transparency systems such as Certificate Transparency are instances of this definition.

A Signed Statement is an identifiable and non-repudiable Statement made by an Issuer.
The Issuer selects additional metadata and attaches a proof of endorsement (in most cases, a signature) using the identity key of the Issuer that binds the Statement and its metadata.
Signed Statements can be made transparent by attaching a proof of Registration by a Transparency Service, in the form of a Receipt.
Receipts demonstrate inclusion of Signed Statements in the Append-only Log of a Transparency Service.
By extension, the Signed Statement may say an Artifact (for example, a firmware binary) is transparent if it comes with one or more Transparent Statements from its author or owner, though the context should make it clear what type of Signed Statements is expected for a given Artifact.

Transparency does not prevent dishonest or compromised Issuers, but it holds them accountable.
Any Artifact that may be verified, is subject to scrutiny and auditing by other parties.
The Transparency Service provides a history of Statements, which may be made by multiple Issuers, enabling Relying Parties to make informed decisions.

Transparency is implemented by providing a consistent, append-only, cryptographically verifiable, publicly available record of entries.
A SCITT instance is referred to as a Transparency Service.
Implementations of Transparency Services may protect their Append-only Log using a combination of trusted hardware, replication and consensus protocols, and cryptographic evidence.
A Receipt is an offline, universally-verifiable proof that an entry is registered in the Append-only Log.
Requesting a receipt can result in the production of a new receipt for the same signed statement.
A Receipt's verification key, signing algorithm, validity period, header parameters or other claims MAY change each time a Receipt is produced.

Anyone with access to the Transparency Service can independently verify its consistency and review the complete list of Transparent Statements registered by each Issuer.
However, the Registrations on a separate Transparency Service is generally disjoint, though it is possible to take a Transparent Statement (i.e. a Signed Statement with a Receipt in its unprotected header, from a from the first Transparency Service) and register it on another Transparency Service, where the second Receipt will be over the first Receipt in the unprotected header.

Reputable Issuers are thus incentivized to carefully review their Statements before signing them to produce Signed Statements.
Similarly, reputable Transparency Services are incentivized to secure their Append-only Log, as any inconsistency can easily be pinpointed by any Auditor with read access to the Transparency Service.

The building blocks defined in SCITT are intended to support applications in any supply chain that produces or relies upon digital Artifacts, from the build and supply of software and IoT devices to advanced manufacturing and food supply.

SCITT is a generalization of Certificate Transparency (CT) {{-CT}}, which can be interpreted as a transparency architecture for the supply chain of X.509 certificates.
Considering CT in terms of SCITT:

- CAs (Issuers) sign the ASN.1 DER encoded tbsCertificate structure to produce an X.509 certificate (Signed Statements)
- CAs submit the certificates to one or more CT logs (Transparency Services)
- CT logs produce Signed Certificate Timestamps (Transparent Statements)
- Signed Certificate Timestamps, Signed Tree Heads, and their respective consistency proofs are checked by Relying Parties
- The Append-only Log can be checked by Auditors

# Architecture Overview

The SCITT architecture consists of a very loose federation of Transparency Services, and a set of common formats and protocols for issuing and registering Signed Statements, and auditing Transparent Statements.

In order to accommodate as many Transparency Service implementations as possible, this document only specifies the format of Signed Statements (which must be used by all Issuers) and a very thin wrapper format for Receipts, which specifies the Transparency Service identity and the agility parameters for the Signed Inclusion Proofs.
Most of the details of the Receipt's contents are specified in the COSE Signed Merkle Tree Proof document {{-COMETRE}}.

{{fig-concept-relationship}} illustrates the roles and processes that comprise a Transparency Service independent of any one use case.

This section describes the three main roles and associated processes in SCITT:

- Issuers that use their credentials to create Signed Statements about Artifacts
- Transparency Services that evaluate Signed Statements against Registration Policies, producing Receipts upon successful Registration.
The returned Receipt may be combined with the Signed Statement to create a Transparent Statement.
- Relying Parties that:
  - collect Receipts of Signed Statements for subsequent registration of Transparent Statements;
  - retrieve Transparent Statements for analysis of Statements about Artifacts themselves (e.g. verification);
  - or replay all the Transparent Statements to check for the consistency of the Transparency Service's Append-only Log (e.g. auditing)

In addition, {{fig-concept-relationship}} illustrates multiple Transparency Services and multiple Receipts as a single Signed Statement MAY be registered with one or more Transparency Service.
Each Transparency Service produces a Receipt, which may be aggregated in a single Transparent Statement, demonstrating the Signed Statement was registered by multiple Transparency Services.

The arrows indicate the flow of information.

~~~aasvg
                        +------------+
 .----------.           |   Issuer   |
|  Artifact  |          +-+--------+-+
 '----+-----'             v        v
      v          .--------+-.    .-+--------.
 .----+----.    /   sign   /    /  verify  /
| Statement |  '-----+----+    '------+---+
 '----+----'         |                |
      |    .--------' '--.            |
      |   |               |           |
      v   v               |        .-' '-.
 .----+---+---.           |       |       |
|    Signed    |          |       |       |
|   Statement  |          |       |       |
 '------+-----'           v       v       |
        |             +---+-------+---+   |
    .--' '----------->+ Transparency  |   |
   |   .--------.     |               |   |
   |  | Receipt  +<---+   Service     +-+ |
   |  |          +.   +--+------------+ | |
   |   '-+------'  |     | Transparency | |
   |     | Receipt +<----+              | |
   |      '------+'      |   Service    | |
    '-------. .-'        +------------+-+ |
             |                        |   |
             v                        |   |
       .-----+-----.                  |   |
      | Transparent |                 |   |
      |  Statement  |                 |   |
       '--+--------'                  |   |
          |                           |   |
          |'-----------.   .----------)--'
          |             | |           |
          |             v v           |
          |    .--------+-+---------. |
          |   / Verify Transparent /  |
          |  /      Statement     /   |
          | '----+---------------+    |
          |      | Relying Party |    |
          |      +---------------+    |
          v                           v
  .-------+-------------.  .----------+------.
 / Collecting Receipts /  /   Replay Log    /
'-----+---------------+  '-+---------------+
      | Relying Party |    | Relying Party |
      +---------------+    +---------------+
~~~
{: #fig-concept-relationship title="Relationship of Concepts in SCITT"}

The subsequent sections describe the main concepts, namely Transparency Service, Signed Statements, Registration, and Transparent Statements in more detail.

## Transparency Service

Transparency Services MUST feature an Append-only Log.
The Append-only Log is the verifiable data structure that records registered Signed Statements and supports the production of Receipts.

All Transparency Services MUST expose APIs for the Registration of Signed Statements and issuance of Receipts.

Transparency Services MAY support additional APIs for auditing, for instance, to query the history of Signed Statements.

Typically a Transparency Service has a single Issuer identity which is present in the `iss` Claim of Receipts for that service.

Multi-tenant support can be enabled through the use of identifiers in the `iss` Claim, for example, `ts.example` may have a distinct Issuer identity for each sub domain, such as `customer1.ts.example` and `customer2.ts.example`.

### Registration Policies

Registration Policies refer to additional checks over and above the Mandatory Registration Checks that are performed before a Signed Statement is accepted to be registered to the Append-only Log.

Transparency Services MUST maintain Registration Policies.
Transparency Services MUST maintain a list of trust anchors (see definition of trust anchor in {{-Glossary}}).
Transparency Services MUST authenticate signed statements as part of a Registration Policy.
For instance, a trust anchor could be an X.509 root certificate, a pointer to an OpenID Connect identity provider, or any other COSE-compatible trust anchor.

Registration Policies and trust anchors MUST be made transparent and available to all Relying Parties of the Transparency Service by registering them as Signed Statements on the Append-only Log, and distributing the associated Receipts.

This specification leaves implementation, encoding and documentation of Registration Policies and trust anchors to the operator of the Transparency Service.

#### Mandatory Registration Checks

During Registration, a Transparency Service MUST, at a minimum, syntactically check the Issuer of the Signed Statement by cryptographically verifying the COSE signature according to {{RFC9052}}.
The Issuer identity MUST be bound to the Signed Statement by including an identifier in the protected header.
If the protected header includes multiple identifiers, all those that are registered by the Transparency Service MUST be checked.

In essence, when using X.509 Signed Statements, the Transparency Service MUST build and validate a complete certification path from an Issuer's certificate to one of the root certificates most recently registered as a trust anchor by the Transparency Service.

The protected header of the COSE_Sign1 Envelope MUST include either the Issuer's certificate as `x5t` or the chain including the Issuer's certificate as `x5chain`.
If `x5t` is included in the protected header, an `x5chain` with a leaf certificate corresponding to the `x5t` value MAY be included in the unprotected header.

The Transparency Service MUST apply the Registration Policy that was most recently added to the Append-only Log at the time of Registration.

#### Auditability of Registration

The operator of a Transparency Service MAY update the Registration Policy or the trust anchors of a Transparency Service at any time.

Transparency Services MUST ensure that for any Signed Statement they register, enough information is made available to Auditors (either in the Append-only Log and retrievable through audit APIs, or included in the Receipt) to reproduce the Registration checks that were defined by the Registration Policies at the time of Registration.

### Initialization and Bootstrapping {#ts-initialization}

Since the mandatory Registration checks rely on having registered Signed Statements for the Registration Policy and trust anchors, Transparency Services MUST support at least one of the three following bootstrapping mechanisms:

- Pre-configured Registration Policy and trust anchors;
- Acceptance of a first Signed Statement whose payload is a valid Registration Policy, without performing Registration checks
- An out-of-band authenticated management interface

### Append-only Log

The security properties of the Append-only Log are determined by the choice of the verifiable data structure used by the Transparency Service to implement the Log.
This verifiable data structure MUST support the following security requirements:

Append-Only:

: once included in the verifiable data structure, a Signed Statement cannot be modified, deleted, or reordered; hence its Receipt provides an offline verifiable proof of Registration.

Non-equivocation:

: there is no fork in the Append-only Log.
Everyone with access to its content sees the same collection of Signed Statements and can check that it is consistent with any Receipts they have verified.

Replayability:

: the Append-only Log includes sufficient information to enable authorized actors with access to its content to check that each included Signed Statement has been correctly registered.

In addition to Receipts, some verifiable data structures might support additional proof types, such as proofs of consistency, or proofs of non inclusion.

Specific verifiable data structures, such those describes in {{-CT}} and {{-COMETRE}}, and the review of their security requirements for SCITT are out of scope for this document.

### Adjacent Services

Transparency Services can be deployed along side other database or object storage technologies.
For example, a Transparency Service that is supporting a software package management system, might be referenced from the APIs exposed for package management.
Providing an ability to request a fresh Receipt for a given software package, or to request a list of Signed Statements associated with the software package.

## Signed Statements

This specification prioritizes conformance to {{RFC9052}} and its required and optional properties.
Profiles and implementation specific choices should be used to determine admissability of conforming messages.
This specification is left intentionally open to allow implementations to make the restrictions that make the most sense for their operational use cases.

There are many types of Statements (such as SBOMs, malware scans, audit reports, policy definitions) that Issuers may want to turn into Signed Statements.
An Issuer must first decide on a suitable format (`3`: payload type) to serialize the Statement payload.
For a software supply chain, payloads describing the software Artifacts may include:

- {{COSWID}}
- {{CycloneDX}}
- {{in-toto}}
- {{SPDX-CBOR}}
- {{SPDX-JSON}}
- {{SLSA}}
- {{SWID}}

Once all the Envelope headers are set, an Issuer MUST use a standard COSE implementation to produce an appropriately serialized Signed Statement.
The SCITT tag `COSE_Sign1_Tagged` is outside the scope of COSE, and used to indicate that a signed object is a Signed Statement.

Issuers can produce Signed Statements about different Artifacts under the same Identity.
Issuers and Relying Parties must be able to recognize the Artifact to which the Statements pertain by looking at the Signed Statement.
The `iss` and `sub` Claims, within the CWT_Claims protected header, are used to identify the Artifact the Statement pertains to.
(See Subject under {{terminology}} Terminology.)

Issuers MAY use different signing keys (identified by `kid` in the protected header) for different Artifacts, or sign all Signed Statements under the same key.

An Issuer can make multiple Statements about the same Artifact.
For example, an Issuer can make amended Statements about the same Artifact as their view changes over time.

Multiple Issuers can make different, even conflicting Statements, about the same Artifact.
Relying Parties can choose which Issuers they trust.

Multiple Issuers can make the same Statement about a single Artifact, affirming multiple Issuers agree.

At least one identifier representing one credential MUST be included in the protected header of the COSE Envelope, as one of `x5t`, `x5chain` or `kid`.
Additionally, `x5chain` that corresponds to either `x5t` or `kid` identifying the leaf certificate in the included certification path MAY be included in the unprotected header of the COSE Envelope.

- When using x.509 certificates, support for either `x5t` or `x5chain` in the protected header is REQUIRED to implement.
- Support for `kid` in the protected header and `x5chain` in the unprotected header is OPTIONAL to implement.

When `x5t` or `x5chain` is present in the protected header, `iss` MUST be a string that meets URI requirements defined in {{RFC8392}}.
The `iss` value's length MUST be between 1 and 8192 characters in length.

The `kid` header parameter MUST be present when neither `x5t` nor `x5chain` is present in the protected header.
Key discovery protocols are out-of-scope of this document.

The protected header of a Signed Statement and a Receipt MUST include the `CWT Claims` header parameter as specified in {{Section 2 of CWT_CLAIMS_COSE}}.
The `CWT Claims` value MUST include the `Issuer Claim` (Claim label 1) and the `Subject Claim` (Claim label 2) {{IANA.cwt}}.

A Receipt is a Signed Statement, (cose-sign1), with addition Claims in its protected header related to verifying the inclusion proof in its unprotected header.
See {{-COMETRE}}.

### Signed Statement Examples

{{fig-signed-statement-cddl}} illustrates a normative CDDL definition (see {{-CDDL}}) for of the protected header and unprotected header of Signed Statements and Receipts.

This definition specifies the minimal mandatory labels.
Implementation-specific Registration Policies may define additional mandatory labels.
A Transparency Service implementation MUST reject registering Signed Statements that do not meet their current Registration Policy requirements.
Each implementation SHOULD provide details for their registration policies through documentation or discovery APIs.

~~~ cddl
{::include signed_statement.cddl}
~~~
{: #fig-signed-statement-cddl title="CDDL definition for Signed Statements and Receipts"}

{{fig-signed-statement-edn}} illustrates an instance of a Signed Statement in Extended Diagnostic Notation (EDN), with a payload that is detached.
Detached payloads support large Statements, and ensure Signed Statements can integrate with existing storage systems.

~~~ cbor-diag
18(                                 / COSE Sign 1      /
    [
      h'a4012603...6d706c65',       / Protected        /
      {},                           / Unprotected      /
      nil,                          / Detached payload /
      h'79ada558...3a28bae4'        / Signature        /
    ]
)
~~~
{: #fig-signed-statement-edn title="CBOR Extended Diagnostic Notation example of a Signed Statement"}

{{fig-signed-statement-protected-header-edn}} illustrates the decoded protected header of the Signed Statement in {{fig-signed-statement-edn}}.
It indicates the Signed Statement is securing a JSON content type, and identifying the content with the `sub` Claim "vendor.product.example".

~~~ cbor-diag
{                                   / Protected        /
  1: -7,                            / Algorithm        /
  3: application/example+json,      / Content type     /
  4: h'50685f55...50523255',        / Key identifier   /
  15: {                             / CWT Claims       /
    1: software.vendor.example,     / Issuer           /
    2: vendor.product.example,      / Subject          /
  }
}
~~~
{: #fig-signed-statement-protected-header-edn title="CBOR Extended Diagnostic Notation example of a Signed Statement's Protected Header"}

## Registration

To register a Signed Statement, the Transparency Service performs the following steps:

1. **Client authentication:** A Client authenticates with the Transparency Service before registering Signed Statements on behalf of one or more Issuers.
Authentication and authorization are implementation-specific and out of scope of the SCITT architecture.
1. **TS Signed Statement Verification and Validation:** The Transparency Service MUST perform signature verification per {{Section 4.4 of RFC9052}} and MUST verify the signature of the Signed Statement with the signature algorithm and verification key of the Issuer per {{RFC9360}}.
The Transparency Service MUST also check the Signed Statement includes the required protected headers.
The Transparency Service MAY validate the Signed Statement payload in order to enforce domain specific registration policies that apply to specific content types.
1. **Apply Registration Policy:** The Transparency Service MUST check the attributes required by a Registration Policy are present in the protected headers.
  Custom Signed Statements are evaluated given the current Transparency Service state and the entire Envelope, and may use information contained in the attributes of named policies.
1. **Register the Signed Statement** to the Append-only Log.
1. **Return the Receipt**, which MAY be asynchronous from Registration.
The Transparency Service MUST be able to provide a Receipt for all registered Signed Statements.
Details about generating Receipts are described in {{Receipt}}.

The last two steps may be shared between a batch of Signed Statements registered in the Append-only Log.

A Transparency Service MUST ensure that a Signed Statement is registered before releasing its Receipt.

The same Signed Statement may be independently registered in multiple Transparency Services, producing multiple, independent Receipts.
The multiple Receipts may be attached to the unprotected header of the Signed Statement, creating a Transparent Statement.

## Transparent Statements {#Receipt}

The Client (which is not necessarily the Issuer) that registers a Signed Statement and receives a Receipt can produce a Transparent Statement by adding the Receipt to the unprotected header of the Signed Statement.
Client applications MAY register Signed Statements on behalf of one or more Issuers.
Client applications MAY request Receipts regardless of the identity of the Issuer of the associated Signed Statement.

When a Signed Statement is registered by a Transparency Service a Receipt becomes available.
When a Receipt is included in a Signed Statement a Transparent Statement is produced.

Receipts are based on Signed Inclusion Proofs as described in COSE Signed Merkle Tree Proofs ({{-COMETRE}}) that also provides the COSE header parameter semantics for label 394.

The Registration time is recorded as the timestamp when the Transparency Service added this Signed Statement to its Append-only Log.

{{fig-transparent-statement-cddl}} illustrates a normative CDDL definition of Transparent Statements.

~~~ cddl
{::include transparent_statement.cddl}
~~~
{: #fig-transparent-statement-cddl title="CDDL definition for a Transparent Statement"}

{{fig-transparent-statement-edn}} illustrates a Transparent Statement with a detached payload, and two Receipts in its unprotected header.
The type of label 394 `receipts` in the unprotected header is a CBOR array that can contain one or more Receipts (each entry encoded as a .cbor encoded Receipts).

~~~ cbor-diag
18(                                 / COSE Sign 1               /
    [
      h'a4012603...6d706c65',       / Protected                 /
      {                             / Unprotected               /
        394: [                      / Receipts (2)              /
          h'd284586c...4191f9d2'    / Receipt 1                 /
          h'c624586c...8f4af97e'    / Receipt 2                 /
        ]
      },
      nil,                          / Detached payload          /
      h'79ada558...3a28bae4'        / Signature                 /
    ]
)
~~~
{: #fig-transparent-statement-edn title="CBOR Extended Diagnostic Notation example of a Transparent Statement"}

{{fig-receipt-edn}} one of the decoded Receipt from {{fig-transparent-statement-edn}}.
The Receipt contains inclusion proofs for verifiable data structures.
The unprotected header contains verifiable data structure proofs.
See the protected header for details regarding the specific verifiable data structure used.
Per the COSE Verifiable Data Structure Registry documented in {{-COMETRE}}, the COSE key type RFC9162_SHA256 is value `1`.
Labels identify inclusion proofs (`-1`) and consistency proofs (`-2`).

~~~ cbor-diag
18(                                 / COSE Sign 1               /
    [
      h'a4012604...6d706c65',       / Protected                 /
      {                             / Unprotected               /
        -222: {                     / Proofs                    /
          -1: [                     / Inclusion proofs (1)      /
            h'83080783...32568964', / Inclusion proof 1         /
          ]
        },
      },
      nil,                          / Detached payload          /
      h'10f6b12a...4191f9d2'        / Signature                 /
    ]
)
~~~
{: #fig-receipt-edn title="CBOR Extended Diagnostic Notation example of a Receipt"}

{{fig-receipt-protected-header-edn}} illustrates the decoded protected header of the Transparent Statement in {{fig-transparent-statement-edn}}.
The verifiable data structure (`-111`) uses `1` from (RFC9162_SHA256).

~~~ cbor-diag
{                                   / Protected                 /
  1: -7,                            / Algorithm                 /
  4: h'50685f55...50523255',        / Key identifier            /
  -111: 1,                          / Verifiable Data Structure /
  15: {                             / CWT Claims                /
    1: transparency.vendor.example, / Issuer                    /
    2: vendor.product.example,      / Subject                   /
  }
}
~~~
{: #fig-receipt-protected-header-edn title="CBOR Extended Diagnostic Notation example of a Receipt's Protected Header"}

{{fig-receipt-inclusion-proof-edn}} illustrates the decoded inclusion proof from {{fig-receipt-edn}}.
This inclusion proof indicates that the size of the Append-only Log was `8` at the time the Receipt was issued.
The structure of this inclusion proof is specific to the verifiable data structure used (RFC9162_SHA256).

~~~ cbor-diag
[                                   / Inclusion proof 1         /
  8,                                / Tree size                 /
  7,                                / Leaf index                /
  [                                 / Inclusion hashes (3)      /
     h'c561d333...f9850597'         / Intermediate hash 1       /
     h'75f177fd...2e73a8ab'         / Intermediate hash 2       /
     h'0bdaaed3...32568964'         / Intermediate hash 3       /
  ]
]
~~~
{: #fig-receipt-inclusion-proof-edn title="CBOR Extended Diagnostic Notation example of a Receipt's Inclusion Proof"}

### Validation {#validation}

Relying Parties MUST apply the verification process as described in Section 4.4 of RFC9052, when checking the signature of Signed Statements and Receipts.

A Relying Party MUST trust the verification key or certificate and the associated identity of at least one Issuer of a Receipt.

A Relying Party MAY decide to verify only a single Receipt that is acceptable to them, and not check the signature on the Signed Statement or Receipts which rely on verifiable data structures which they do not understand.

APIs exposing verification logic for Transparent Statements may provide more details than a single boolean result.
For example, an API may indicate if the signature on the Receipt or Signed Statement is valid, if Claims related to the validity period are valid, or if the inclusion proof in the Receipt is valid.

Relying Parties MAY be configured to re-verify the Issuer's Signed Statement locally.

In addition, Relying Parties MAY apply arbitrary validation policies after the Transparent Statement has been verified and validated.
Such policies may use as input all information in the Envelope, the Receipt, and the Statement payload, as well as any local state.

# Privacy Considerations

Transparency Services MAY support anonymous access.
Issuers SHOULD ensure Signed Statements submitted to public access services are acceptable for public disclosure.
Publicly accessible Signed Statements MUST NOT carry confidential information.
Once a Signed Statement is inserted into the Append-only Log maintained by a Transparency Service, it cannot be removed from the Log.
In some deployments, a special role, such as an Auditor, might require access to Signed Statements.

# Security Considerations

On its own, verifying a Transparent Statement does not guarantee that its Envelope or contents are trustworthy.
Just that they have been signed by the apparent Issuer and counter-signed by the Transparency Service.
If the Relying Party trusts the Issuer, after validation of the Issuer identity, it can infer that an Issuer's Signed Statement was issued with this Envelope and contents, which may be interpreted as the Issuer saying the Artifact is fit for its intended purpose.
If the Relying Party trusts the Transparency Service, it can independently infer that the Signed Statement passed the Transparency Service Registration Policy and that has been persisted in the Append-only Log.
Unless advertised in the Transparency Service Registration Policy, the Relying Party cannot assume that the ordering of Signed Statements in the Append-only Log matches the ordering of their issuance.

Similarly, the fact that an Issuer can be held accountable for its Transparent Statements does not on its own provide any mitigation or remediation mechanism in case one of these Transparent Statements turned out to be misleading or malicious.
Just that signed evidence will be available to support them.

An Issuer that knows of a changed state of quality for an Artifact, SHOULD Register a new Signed Statement, using the same `15` CWT `iss` and `sub` Claims.

Issuers MUST ensure that the Statement payloads in their Signed Statements are correct and unambiguous, for example by avoiding ill-defined or ambiguous formats that may cause Relying Parties to interpret the Signed Statement as valid for some other purpose.

Issuers and Transparency Services MUST carefully protect their private signing keys and avoid these keys being used for any purpose not described in this architecture document.
In cases where key re-use is unavoidable, keys MUST NOT sign any other message that may be verified as an Envelope as part of a Signed Statement.

For instance, the code for the Registration Policy evaluation and endorsement may be protected by running in a Trusted Execution Environment (TEE).

The Transparency Service may be replicated with a consensus algorithm, such as Practical Byzantine Fault Tolerance {{PBFT}} and may be used to protect against malicious or vulnerable replicas.
Threshold signatures may be use to protect the service key, etc.

Issuers and Transparency Services MUST rotate their keys in well-defined cryptoperiods, see {{KEY-MANAGEMENT}}.

A Transparency Service MAY provide additional authenticity assurances about its secure implementation and operation, enabling remote attestation of the hardware platforms and/or software Trusted Computing Bases (TCB) that run the Transparency Service.
If present, these additional authenticity assurances MUST be registered in the Append-only Log and MUST always be exposed by the Transparency Services' APIs.
An example of Signed Statement's payloads that can improve authenticity assurances are trustworthiness assessments that are RATS Conceptual Messages, such as Evidence, Endorsements, or corresponding Attestation Results (see {{-rats-arch}}).

For example, if a Transparency Service is implemented using a set of redundant replicas, each running within its own hardware-protected trusted execution environments (TEEs), then each replica can provide fresh Evidence or fresh Attestation Results about its TEEs.
The respective Evidence can show, for example, the binding of the hardware platform to the software that runs the Transparency Service, the long-term public key of the service, or the key used by the replica for signing Receipts.
The respective Attestation Result, for example, can show that the remote attestation Evidence was appraised by a Relying Party and complies with well-known Reference Values and Endorsements.

Auditors should be aware that the certification path information included in an unprotected `x5chain` header of a to-be-registered Signed Statement can be tampered with by a malicious Transparency Service (e.g., one that does not incorporate remote attestation), which may replace the intermediate certificates and ultimately connect to an unexpected root.
This modification helps protect against person-in-the-middle attacks, but not denial-of-service.
Auditors MUST perform certification path validation in accordance with PKIX rules specified in {{-PKIX}}.
In particular, Auditors MUST verify that certification paths chain to one or more trust anchors (often represented as root certificates).

## Security Guarantees

SCITT provides the following security guarantees:

1. Statements made by Issuers about supply chain Artifacts are identifiable, can be authenticated, and once authenticated, are non-repudiable
1. Statement provenance and history can be independently and consistently audited
1. Issuers can efficiently prove that their Statement is logged by a Transparency Service

The first guarantee is achieved by requiring Issuers to sign their Statements and associated metadata using a distributed public key infrastructure.
The second guarantee is achieved by storing the Signed Statement on an Append-only Log.
The third guarantee is achieved by implementing the Append-only Log using a verifiable data structure (such as a Merkle Tree {{MERKLE}}).

## Threat Model

This section provides a generic threat model for SCITT, describing its residual security properties when some of its actors (Issuers, Transparency Services, and Auditors) are corrupt or compromised.

This threat model may need to be refined to account for specific supply chain use cases.

SCITT primarily supports checking of Signed Statement authenticity, both from the Issuer (authentication) and from the Transparency Service (transparency).
These guarantees are meant to hold for extensive periods of time, possibly decades.

It can never be assumed that some Issuers and some Transparency Services will not be corrupt.

SCITT entities explicitly trust one another on the basis of their long-term identity, which maps to shorter-lived cryptographic credentials.
A Relying Party SHOULD validate a Transparent Statement originating from a given Issuer, registered at a given Transparency Service (both identified in the Relying Party's local authorization policy) and would not depend on any other Issuer or Transparency Services.

Issuers cannot be stopped from producing Signed Statements including false assertions in their Statement payload (either by mistake or by corruption), but these Issuers can made accountable by ensuring their Signed Statements are systematically registered at a Transparency Service.

Similarly, providing strong residual guarantees against faulty/corrupt Transparency Services is a SCITT design goal.
Preventing a Transparency Service from registering Signed Statements that do not meet its stated Registration Policy, or to issue Receipts that are not consistent with their Append-only Log is not possible.
In contrast Transparency Services can be held accountable and they can be called out by any Auditor that replays their Append-only Log against any contested Receipt.
Note that the SCITT Architecture does not require trust in a single centralized Transparency Service.
Different actors may rely on different Transparency Services, each registering a subset of Signed Statements subject to their own policy.

In both cases, the SCITT architecture provides generic, universally-verifiable cryptographic proofs to individually blame Issuers or the Transparency Service.
On one hand, this enables valid actors to detect and disambiguate malicious actors who employ Equivocation with Signed Statements to different entities.
On the other hand, their liability and the resulting damage to their reputation are application specific, and out of scope of the SCITT architecture.

Relying Parties and Auditors need not be trusted by other actors.
In particular, so long as actors maintain proper control of their signing keys and identity infrastructure they cannot "frame" an Issuer or a Transparency Service for Signed Statements they did not issue or register.

### Append-only Log

If a Transparency Service is honest, then a Transparent Statement including a correct Receipt ensures that the associated Signed Statement passed its Registration Policy and was registered appropriately.

Conversely, a corrupt Transparency Service may:

1. refuse or delay the Registration of Signed Statements
1. register Signed Statements that do not pass its Registration Policy (e.g., Signed Statement with Issuer identities and signatures that do not verify)
1. issue verifiable Receipts for Signed Statements that do not match its Append-only Log
1. refuse access to its Transparency Service (e.g., to Auditors, possibly after storage loss)

An Auditor granted (partial) access to a Transparency Service and to a collection of disputed Receipts will be able to replay it, detect any invalid Registration (2) or incorrect Receipt in this collection (3), and blame the Transparency Service for them.
This ensures any Relying Party that trusts at least one such Auditor that (2, 3) will be blamed to the Transparency Service.

Due to the operational challenge of maintaining a globally consistent Append-only Log, some Transparency Services may provide limited support for historical queries on the Signed Statements they have registered, and accept the risk of being blamed for inconsistent Registration or Issuer Equivocation.

Relying Parties and Auditors may also witness (1, 4) but may not be able to collect verifiable evidence for it.

### Availability of Receipts

Networking and Storage are trusted only for availability.

Auditing may involve access to data beyond what is persisted in the Transparency Services.
For example, the registered Transparency Service may include only the hash of a detailed SBOM, which may limit the scope of auditing.

Resistance to denial-of-service is implementation specific.

Actors may want to independently keep their own record of the Signed Statements they issue, endorse, verify, or audit.

### Confidentiality and Privacy

All contents exchanged between actors is protected using secure authenticated channels (e.g., TLS) but may not exclude network traffic analysis.

The Transparency Service is trusted with the confidentiality of the Signed Statements presented for Registration.
Some Transparency Services may publish every Signed Statement in their logs, to facilitate their dissemination and auditing.
Transparency Services MAY return Receipts to Client applications synchronously or asynchronously.

A collection of Signed Statements must not leak information about the contents of other Signed Statements registered on the Transparency Service.

Issuers must carefully review the inclusion of private/confidential materials in their Statements.
For example, Issuers must remove Personally Identifiable Information (PII) as clear text in the Statement.
Alternatively, Issuers may include opaque cryptographic Statements, such as hashes.

The confidentiality of queries is implementation-specific, and generally not guaranteed.
For example, while offline Envelope validation of Signed Statements is private, a Transparency Service may monitor which of its Transparent Statements are being verified from lookups to ensure their freshness.

### Cryptographic Agility

The SCITT Architecture supports cryptographic agility.
The actors depend only on the subset of signing and Receipt schemes they trust.
This enables the gradual transition to stronger algorithms, including e.g. post-quantum signature algorithms.

### Transparency Service Client Applications

Authentication of Client applications is out of scope for this document.
Transparency Services MUST authenticate both Client applications and the Issuer of Signed Statements in order to ensure that implementation specific authentication and authorization policies are enforced.
The specification of authentication and authorization policies is out of scope for this document.

### Impersonation

The identity resolution mechanism is trusted to associate long-term identifiers with their public signature-verification keys.
Transparency Services and other parties may record identity-resolution evidence to facilitate its auditing.

If one of the credentials of an Issuer gets compromised, the SCITT Architecture still guarantees the authenticity of all Signed Statements signed with this credential that have been registered on a Transparency Service before the compromise.
It is up to the Issuer to notify Transparency Services of credential revocation to stop Relying Parties from accepting Signed Statements signed with compromised credentials.

# IANA Considerations

## Media Type Registration

Pending WG discussion.

--- back

# Common Terminology Disambiguation

This document has been developed in coordination with the COSE, OAUTH and RATS WG and uses terminology common to these working groups.

This document uses the terms "Issuer", and "Subject" as described in {{RFC8392}}, however the usage is consistent with the broader interpretation of these terms in both JOSE and COSE, and in particular, the guidance in {{RFC8725}} generally applies the COSE equivalent terms with consistent semantics.

The terms "verifier" and "Relying Party" are used interchangeably through the document.
While these terms are related to "Verifier" and "Relying Party" as used in {{RFC9334}}, they do not imply the processing of RATS conceptual messages, such as Evidence or Attestation Results that are specific to remote attestation.
A SCITT "verifier" and "Relying Party" and "Issuer" of Receipts or Statements might take on the role of a RATS "Attester".
Correspondingly, all RATS conceptual messages, such as Evidence and Attestation Results, can be the content of SCITT Statements and a SCITT "verifier" can also take on the role of a RATS "Verifier" to, for example, conduct the procedure of Appraisal of Evidence as a part of a SCITT "verifier"'s verification capabilities.

The terms "Claim" and "Statement" are used throughout this document, where Claim is consistent with the usage in {{-draft-ietf-rats-eat}} and {{RFC7523}}, and Statement is reserved for any arbitrary bytes, possibly identified with a media type, about which the Claims are made.

The term "Subject" provides an identifier of the Issuer's choosing to refer to a given Artifact, and ensures that all associated Statements can be attributed to the identifier chosen by the Issuer.

In simpler language, a SCITT Statement could be some vendor-specific software bill of materials (SBOM), results from a model checker, static analyzer, or RATS Evidence about the authenticity of an SBOM creation process, where the Issuer identifies themselves using the `iss` Claim, and the specific software that was analyzed as the Subject using the `sub` Claim.

In {{RFC7523}}, the Authorization Server (AS) verifies Private Key JWT client authentication requests, and issues access tokens to clients configured to use "urn:ietf:params:oauth:client-assertion-type:jwt-bearer".
This means the AS initially acts as a "verifier" of the authentication credentials in form of a JWT, and then later as an "Issuer" of access and refresh tokens.
This mirrors how Signed Statements are verified before Receipts are issued by a Transparency Service.
Note that the use of {{RFC7523}} is only one possible approach for client authentication in OAuth.

{{FIPS.201}} defines "assertion" as "A verifiable statement from an IdP to an RP that contains information about an end user".

{{NIST.SP.800-63-3}} defines "assertion" as "A statement from a verifier to an RP that contains information about a subscriber.
Assertions may also contain verified attributes."

This document uses the term Statement to refer to potentially unsecured data and associated Claims, and Signed Statement and Receipt to refer to assertions from an Issuer, or the Transparency Service.

{{NIST.SP.1800-19}} defines "attestation" as "The process of providing a digital signature for a set of measurements securely stored in hardware, and then having the requester validate the signature and the set of measurements."

NIST guidance "Software Supply Chain Security Guidance EO 14028" uses the definition from {{ISO.17000.2020}}, which states that an "attestation" is "The issue of a statement, based on a decision, that fulfillment of specified requirements has been demonstrated.".
In the RATS context, a "NIST attestation" is similar to a RATS "Endorsement".
Occasionally, RATS Evidence and RATS Attestation Results or the procedures of creating these conceptual messages are referred to as "attestation" or (in cases of the use as a verb) "to attest".
The stand-alone use of "attestation" and "to attest" is discouraged outside a well-defined context, such as specification text that highlights the application of terminology, explicitly.
Correspondingly, it is often useful for the intended audience to qualify the term "attestation" to avoid confusion and ambiguity.

# Identifiers

This section provides informative examples of identifiers for Statements, Signed Statements, and Receipts.

SCITT Identifiers are primarily meant to be understood by humans and secondarily meant to be understood by machines, as such we define text encodings for message identifiers first, and then provide binary translations according to standard transformations for URLs and URNs to binary formats.

SCITT Identifiers for URLs and URNs that are not Data URLs MUST be represented in binary using {{-CURIs}}.

For each SCITT conceptual message, we define a Data URL format according to {{-DataURLs}}, a URN format according to {{-URNs}} and a URL format according to {{URLs}}.

Note that Data URLs require base64 encoding, but the URN definitions require base64url encoding.

Resolution and dereferencing of these identifiers is out of scope for this document, and can be implemented by any concrete api implementing the abstract interface defined as follows:

~~~
resource: content-type = dereference(identifier: identifier-type)
~~~

These identifiers MAY be present in a `tstr` field that does not otherwise restrict the string in ways that prevent a URN or URL from being present.

This includes `iss`, and `sub` which are used to express the Issuer and Subject of a Signed Statement or Receipt.

This also includes `kid` which is used to express a hint for which public key should be used to verify a signature.

All SCITT identifiers share common parameters to promote interoperability:

Let hash-name be an algorithm name registered in {{IANA.named-information}}.

To promote interoperability, the hash-name MUST be "sha-256".

Let base-encoding, be a base encoding defined in {{-Base64Url}}.

To promote interoperability, the base encoding MUST be "base64url".

In the blocks and examples that follow, note '\' line wrapping per RFC 8792.

## Identifiers For Binary Content

Identifiers for binary content, such as Statements, or even Artifacts themselves are computed as follows:

Let the `base64url-encoded-bytes-digest` for the message be the base64url encoded digest with the chosen hash algorithm of bytes / octets.

Let the SCITT name for the message be the URN constructed from the following URI template, according to {{-URITemplate}}:

Let the `message-type`, be "statement" for Statements about Artifacts.

~~~
urn:ietf:params:scitt:\
{message-type}:\
{hash-name}:{base-encoding}:\
{base64url-encoded-bytes-digest}
~~~

## Identifiers For SCITT Messages

Identifiers for COSE Sign 1 based messages, such as identifiers for Signed Statements and Receipts are computed as follows:

Let the `base64url-encoded-to-be-signed-bytes-digest` for the message be the base64url encoded digest with the chosen hash algorithm of the "to-be-signed bytes", according to {{Section 8.1 of RFC9052}}.

Let the SCITT name for the message be the URN constructed from the following URI template, according to {{-URITemplate}}:

Let the `message-type`, be "signed-statement" for Signed Statements, and "receipt" for Receipts.

~~~
urn:ietf:params:scitt:\
{message-type}:\
{hash-name}:{base-encoding}:\
{base64url-encoded-to-be-signed-bytes-digest}
~~~

Note that this means the content of the signature is not included in the identifier, even though signature related Claims, such as activation or expiration information in protected headers are included.

As a result, an attacker may construct a new Signed Statement that has the same identifier as a previous Signed Statement, but has a different signature.

## Identifiers For Transparent Statements

Identifiers for Transparent Statements are defined as identifiers for binary content, but with "transparent-statement" as the `message-type`.

~~~
urn:ietf:params:scitt:\
{message-type}:\
{hash-name}:{base-encoding}:\
{base64url-encoded-bytes-digest}
~~~

Note that because this identifier is computed over the unprotected header of the Signed Statement, any changes to the unprotected header, such as changing the order of the unprotected header map key value pairs, adding additional Receipts, or adding additional proofs to a Receipt, will change the identifier of a Transparent Statement.

Note that because this identifier is computed over the signatures of the Signed Statement and signatures in each Receipt, any canonicalization of the signatures after the fact will produce a distinct identifier.

## Statements

### Statement URN

~~~
urn:ietf:params:scitt:statement:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-statement-urn align="left" title="Example Statement URN"}

### Statement URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:statement:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-statement-url align="left" title="Example Statement URL"}

### Statement Data URL

~~~
data:application/json;base64,SGVsb...xkIQ==
~~~
{: #example-statement-data-url align="left" title="Example Statement Data URL"}

## Signed Statements

### Signed Statement URN

~~~
urn:ietf:params:scitt:\
signed-statement:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-signed-statement-urn align="left" title="Example Signed Statement URN"}

### Signed Statement URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:\
signed-statement:sha-256:base64url:5i6...r1o
~~~
{: #example-signed-statement-url align="left" title="Example Signed Statement URL"}

### Signed Statement Data URL

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #example-signed-statement-data-url align="left" title="Example Signed Statement Data URL"}

## Receipts

### Receipt URN

~~~
urn:ietf:params:scitt:receipt:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-receipt-urn align="left" title="Example Receipt URN"}

### Receipt URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:receipt:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-receipt-url align="left" title="Example Receipt URL"}

### Receipt Data URL

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #example-receipt-data-url align="left" title="Example Receipt Data URL"}

## Transparent Statements

### Transparent Statement URN

~~~
urn:ietf:params:scitt:\
transparent-statement:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-transparent-statement-urn align="left" title="Example Transparent Statement URN"}

### Transparent Statement URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:\
transparent-statement:sha-256:base64url:5i6UeR...qnGmr1o
~~~
{: #example-transparent-statement-url align="left" title="Example Transparent Statement URL"}

### Transparent Statement Data URL

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #example-transparent-statement-data-url align="left" title="Example Transparent Statement Data URL"}

# Signing Statements Remotely

Statements about digital Artifacts, containing digital Artifacts, or structured data regarding any type of Artifacts, can be too large or too sensitive to be send to a remote Transparency Services over the Internet.
In these cases a Statement can also be hash, which becomes the payload included in COSE to-be-signed bytes.
A Signed Statement (cose-sign1) MUST be produced from the to-be-signed bytes according to {{Section 4.4 of RFC9052}}.

~~~aasvg
   .----+-----.
  |  Artifact  |
   '+-+-------'
    | |
 .-'  v
|  .--+-------.
| |  Hash      +-+
|  '----------'  |     /\
 '-.             |    /  \     .----------.
    |            +-->+ OR +-->+  Payload   |
    v            |    \  /     '--------+-'
   .+--------.   |     \/               |
  | Statement +--+                      |
   '---------'                          |
                                        |
                                        |
           ...  Producer Network ...    |

                      ...

           ...   Issuer Network ...     |
                                        |
                                        |
 .---------.                            |
| Identity  |     (iss, x5t)            |
| Document  +--------------------+      |
 `----+----`                     |      |
      ^                          |      |
 .----+-------.                  |      |
| Private Key  |                 |      |
 '----+-------'                  v      |
      |                     .----+---.  |
      |                    |  Header  | |
      |                     '----+---'  |
      v                          v      v
    .-+-----------.       .------+------+--.
   /             /       /                  \
  /    Sign     +<------+ To Be Signed Bytes |
 /             /         \                  /
'-----+-------'           '----------------'
      v
 .----+-------.
| COSE Sign 1  |
 '------------'
~~~
