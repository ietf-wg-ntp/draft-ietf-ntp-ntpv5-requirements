---
title: "NTPv5 use cases and requirements"
abbrev: "NTPv5 use cases and requirements"
docname: draft-ietf-ntp-ntpv5-requirements-latest
date:

author:
    -
      ins: J. Gruessing
      name: James Gruessing
      organization: Nederlandse Publieke Omroep
      country: Netherlands
      email: james.ietf@gmail.com

ipr: trust200902
category: info
area: Internet
workgroup: Network Time Protocol
keyword: Internet-Draft
stand_alone: yes
pi: [toc, tocindent, sortrefs, symrefs, strict, compact, subcompact, comments, inline]

normative:
    RFC2119:
    RFC2827:
    RFC5905:
    RFC8174:
    I-D.ietf-ntp-roughtime:

informative:
    drdos-amplification:
        title: "Amplification and DRDoS Attack Defense -- A Survey and New Perspectives"
        target: https://arxiv.org/abs/1505.07892
    ntp-misuse:
        title: "NTP server misuse and abuse"
        target: https://en.wikipedia.org/wiki/NTP_server_misuse_and_abuse
    ntppool:
        title: "pool.ntp.org: the internet cluster of ntp servers"
        target: "https://www.ntppool.org"
    IEEE-1588-2019:
        title: "IEEE Standard for a Precision Clock Synchronization Protocol for Networked Measurement and Control Systems"
        doi: "10.1109/IEEESTD.2020.9120376"
    TF.460-6:
        title: "Standard-frequency and time-signal emissions"
        target: "https://www.itu.int/rec/R-REC-TF.460-6-200202-I/en"

--- abstract

This document describes the use cases, requirements, and considerations that
should be factored in the design of a successor protocol to supersede version 4
of the NTP protocol {{!RFC5905}} presently referred to as NTP version 5
("NTPv5").

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-ntp-ntpv5-requirements>.

--- middle

# Introduction

NTP version 4 {{!RFC5905}} has seen active use for over a decade, and within
this time period the protocol has not only been extended to support new
requirements but has also fallen victim to vulnerabilities that have been used
for distributed denial of service (DDoS) amplification attacks. In order to
advance the protocol and address these known issues alongside add capabilities
for future usage this document defines the current known and applicable use
cases in existing NTPv4 deployments and defines requirements for the future.

## Notational Conventions

{::boilerplate bcp14-tagged}

# Use cases and existing deployments of NTP

There are several common scenarios for existing NTPv4 deployments: publicly
accessible NTP services such as the NTP Pool {{ntppool}} are used to offer clock
synchronisation for end users and embedded devices, ISP-provided servers are used to
synchronise devices such as customer-premises equipment where reduced accuracy
may be tolerable. Depending on the network and path these deployments may be
affected by variable latency as well as throttling or blocking by providers.

Data centres and cloud computing providers also have deployed and offer NTP
services both for internal use and for customers, particularly where the network
is unable to offer or does not require PTP {{IEEE-1588-2019}}. As these
deployments are less likely to be constrained by network latency or power the
potential for higher levels of accuracy and precision within the bounds of the
protocol are possible.

# Requirements

At a high level, NTPv5 should be a protocol that is capable of operating in
local networks and over public internet connections where packet loss,
delay, and filtering may occur. It should be able to provide enough
information for both basic time information and synchronisation.

## Resource management

Historically there have been many documented instances of NTP servers receiving
large amounts of unauthorised traffic {{ntp-misuse}} and the design of NTPv5
must ensure the risk of these can be minimised.

The protocol's loop avoidance mechanisms SHOULD NOT use identifiers tied to network
topology. In particular, any such mechanism should not rely on any FQDN, IP address
or identifier tied to a public certificate used or owned by the server. Servers
SHOULD be able to migrate and change any identifier used as stratum topologies or
network configuration changes occur.

An additional identifier mechanism MAY be considered for the purposes of client
allow/deny lists, logging and monitoring. Such a mechanism, when included, SHOULD
be independent of any loop avoidance mechanism, and authenticity requirements
SHOULD be considered.

The protocol MUST have the capability for servers to notify clients that the
service is unavailable, and clients MUST have clearly defined behaviours for
honouring this signalling. In addition servers SHOULD be able to communicate to
clients that they should reduce their query rate when the server is
under high load or has reduced capacity.

Clients SHOULD periodically re-establish connections with servers to prevent
attempting to maintain connectivity to a dead host and give network operators
the ability to move traffic away from hosts in a timely manner.

The protocol SHOULD have provisions for deployments where Network Address
Translation occurs, and define behaviours when NAT rebinding occurs. This should
also not compromise any DDoS mitigation(s) that the protocol may define.

## Data Minimisation

Payload formats SHOULD use the least amount of fields and information where
possible, favouring the use of extensions to transmit optional data. This is
done in part to minimise ongoing use of deprecated fields, in addition to
reducing risks of exposing identifying information of implementations and
deployments.

## Algorithms

The use of algorithms describing functions such as clock filtering, selection,
and clustering SHOULD have agility, allowing for implementations to develop and
deploy new algorithms independently. Signalling of algorithm use or preference
SHOULD NOT be transmitted by servers.

The working group should consider creating a separate informational document to
describe an algorithm to assist with implementation, and consider adopting
future documents which describe new algorithms as they are developed. Specifying
client algorithms separately from the protocol will allow NTPv5 to meet
the needs of applications with a variety of network properties and performance
requirements.

## Timescales

The protocol SHOULD adopt a linear, monotonic timescale as the basis for
communicating time. The format should provide sufficient scale, precision, and
resolution to meet or exceed NTPv4's capabilities, and have a rollover date
sufficiently far into the future that the protocol's complete
obsolescence is likely to occur first.

The timescale, in addition to any other time-sensitive information, MUST be
sufficient to calculate representations of both UTC and TAI {{TF.460-6}}.
Through extensions the protocol SHOULD support additional timescale
representations outside of the main specification, and all transmissions of time
data SHALL indicate the timescale in use.

## Leap seconds

Transmission of UTC leap second information MUST be included in the protocol in
order for clients to generate a UTC representation, but must be transmitted as
separate information to the timescale. The specification MUST require that
servers transmit upcoming leap seconds greater than 1 calendar day in advance
if that information is known by the server. If the server learns of a leap
second less than 1 calendar day before a leap second event, it will start
transmitting the information immediately.

Leap second smearing SHOULD NOT be applied to timestamps transmitted by the
server, however this should not prevent implementers from applying leap second
smearing between the client and any clock it is training.

## Backwards compatibility with NTS and NTPv4

The desire for compatibility with older protocols should not prevent addressing
deployment issues or cause ossification of the protocol.

The model for backward compatibility is: servers that support multiple versions
of NTP must send a response in the same version as the request. This does not
preclude servers from acting as a client in one version of NTP and
a server in another.

Protocol ossification MUST be addressed to prevent existing NTPv4
deployments which respond incorrectly to clients posing as NTPv5 from causing
issues. Forward prevention of ossification (for a potential NTPv6 protocol in
the future) should also be taken into consideration.

### Dependent Specifications

Many other documents make use of NTP's data formats ({{RFC5905}} Section 6) for
representing time, notably for media and packet timestamp measurements. Any
changes to the data formats should consider the potential implementation
complexity that may be incurred.

## Extensibility

The protocol MUST have the capability to be extended; implementations
MUST ignore unknown extensions. Unknown extensions received by a server from a
lower stratum server SHALL not be added to response messages sent by the server
receiving these extensions.

## Security

Data authentication and optional data confidentiality MUST be integrated into
the protocol, and downgrade attacks by an in-path attacker must be mitigated.
The protocol SHOULD support different mechanisms to support different use cases.

Cryptographic agility must be supported, allowing for more secure cryptographic
primitives to be incorporated as they are developed and as
attacks and vulnerabilities with incumbent primitives are discovered.

Intermediate devices such as hardware capable of performing timestamping of
packets SHOULD be able to add information to packets in flight without
requiring modification or removal of authentication or confidentiality on the
packet.

Consideration must be given to how this will be incorporated into any
applicable trust model. Downgrading attacks that could lead to an adversary
disabling or removing encryption or authentication MUST NOT be possible in the
design of the protocol.

# Non-requirements

This section covers topics that are explicitly out of scope.

## Server malfeasance detection

Detection and reporting of server malfeasance should remain out of scope as
{{!I-D.ietf-ntp-roughtime}} already provides this capability as a core
functionality of the protocol.

# Threat model

The assumptions that apply to all of the threats and risks within this section
are based on observations of the use cases defined earlier in this document, and
focus on external threats outside of the trust boundaries which may be in place
within a network. Internal threats and risks such as a trusted operator are out
of scope.

## Delay-based attacks

The risk that an on-path attacker can delay packets between a client and server
exists in all time protocols operating on insecure networks and its mitigations
within the protocol are limited for a clock which is not yet synchronised.
Increased path diversity and protocol support for synchronisation across
multiple heterogeneous sources are likely the most effective mitigations.

## Payload manipulation

Conversely, on-path attackers who can manipulate timestamps could also speed up a
client's clock, resulting in drift-related malfunctions and errors such
as premature expiration of certificates on affected hosts. An
attacker may also manipulate other data in flight to disrupt service and cause
de-synchronisation. Message authentication with regular key rotation should mitigate
both of these cases; however consideration should also be made for
hardware-based timestamping.

## Denial of Service and Amplification

NTPv4 has previously suffered from DDoS amplification attacks using a
combination of IP address spoofing and private mode commands used in many NTP
implementations, leading to an attacker being able to direct very large volumes of
traffic to a victim IP address. Current mitigations are
disabling private mode commands and encouraging network
operators to implement BCP 38 {{RFC2827}}. The NTPv5
protocol specification should reduce the amplification factor in
request/response payload sizes {{drdos-amplification}} through the use of
padding and consideration of payload data.

# IANA Considerations

This document makes no requests of IANA.

# Security Considerations

As this document is intended to create discussion and consensus, it introduces
no security considerations of its own.

--- back

# Acknowledgements

The author would like to thank Doug Arnold, Hal Murray, Paul Gear, and David
Venhoek for contributions to this document, and would like to acknowledge Daniel
Franke, Watson Ladd, Miroslav Lichvar for their existing documents and ideas.
The author would also like to thank Angelo Moriondo, Franz Karl Achard, and
Malcom McLean for providing the author with motivation.
