---
title: "NTPv5 use cases and requirements"
abbrev: "NTPv5 use cases and requirements"
docname: draft-gruessing-ntp-ntpv5-requirements-latest
date:

author:
    -
      ins: J. Gruessing
      name: James Gruessing
      organization: Nederlandse Publieke Omroep
      street: Postbus 26444
      code: 1202 JJ
      city: Hilversum
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
    IEEE-1588-2008:
        title: "IEEE Standard for a Precision Clock Synchronization Protocol for Networked Measurement and Control Systems"
        doi: "10.1109/IEEESTD.2008.4579760"

--- abstract

This document describes the use cases, requirements, and considerations that
should be factored in the design of a successor protocol to supersede version 4
of the NTP protocol {{!RFC5905}} presently referred to as NTP version 5
("NTPv5"). This document is non-exhaustive and does not in its current version
represent working group consensus.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/fiestajetsam/draft-gruessing-ntp-ntpv5-requirements>.

--- middle

# Introduction

NTP version 4 {{!RFC5905}} has seen active use for over a decade, and within
this time period the protocol has not only been extended to support new
requirements but also fallen victim to vulnerabilities that have made it used
for distributed denial of service (DDoS) amplification attacks.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this
document are to be interpreted as described in BCP 14 {{!RFC2119}} {{!RFC8174}}
when, and only when, they appear in all capitals, as shown here.

# Use cases and existing deployments of NTP

There are several common scenarios for existing NTPv4 deployments; publicly
accessible NTP services such as the NTP Pool {{ntppool}} are used to offer clock
synchronisation for end users and embedded devices, ISP provided servers to
synchronise devices such as customer-premises equipment where reduced accuracy
may be tolerable. Depending on the network and path these deployments may be
affected by variable latency as well as throttling or blocking by providers.

Data centres and cloud computing providers also have deployed and offer NTP
services both for internal use and for customers, particularly where the network
is unable to offer or does not require PTP {{IEEE-1588-2008}}. As these
deployments are less likely to be constrained by network latency or power the
potential for higher levels of accuracy and precision within the bounds of the
protocol are possible.

# Requirements

At a high level, NTPv5 should be a protocol that is capable of operating in both
local networks and also over public internet connections where packet loss,
delay, and even filtering may occur. It should be able to provide enough
information for both basic time information as well as synchronisation.

## Resource management

Historically there have been many documented instances of NTP servers taking a
large increase in unauthorised traffic {{ntp-misuse}} and the design of NTPv5
must ensure the risk of these can be minimised to the fullest extent.

Servers SHOULD have a new identifier that peers use as reference, this SHOULD
NOT be a FQDN, an IP address or identifier tied to a public certificate. Servers
SHOULD be able to migrate and change their identifiers as stratum topologies or
network configuration changes occur.

The protocol MUST have the capability for servers to notify clients that the
service is unavailable, and clients MUST have clearly defined behaviours
honouring this signalling. In addition servers SHOULD be able to communicate to
clients that they should reduce their query interval rate when the server is
under high bandwidth or has reduced capacity.

Clients SHOULD re-establish connections with servers at an interval to prevent
attempting to maintain connectivity to a dead host and give network operators
the ability to move traffic away from hosts in a timely manner.

The protocol SHOULD have provisions for deployments where Network Address
Translation occurs, and define behaviours when NAT rebinding occurs. This should
also not compromise any DDoS mitigation(s) that the protocol may define.

## Algorithms

The use of algorithms describing functions such as clock filtering, selection
and clustering SHOULD have agility, allowing for implementations to develop and
deploy new algorithms independantly. Signalling of algorithm use or preference
SHOULD NOT be transmitted by servers.

The working group should consider creating a separate informational document to
describe an algorithm to assist with implementation, and to consider adopting
future documents which describe new algorithms as they are developed. Specifying
client algorithms separately from the protocol allows will allow NTPv5 to meet
the needs of applications with a variety of network properties and performance
requirements.

## Timescales

The protocol SHOULD adopt a linear, monotonic timescale as the basis for
communicating time. The format should meet sufficient scale and precision with
resolution either meeting or exceeding NTPv4, and have a rollover date
sufficiently far enough into the future that the protocol's complete
obsolescence is most likely to occur first.

The timescale in addition to any other time sensitive information MUST be
sufficient to calculate representations of both UTC and TAI. Through extensions
the protocol SHOULD support additional timescale representations outside of the
main specification, and all transmissions of time data SHALL indicate the
timescale in use.

## Leap seconds

Tranmission of UTC leap second information MUST be included in the protocol in
order for clients to generate a UTC representation but must be transmitted as
separate information to the timescale. The specification SHOULD also be capable
of transmitting upcoming leap seconds greater than 1 calendar day in advance.

Leap second smearing SHOULD NOT be applied to timestamps transmitted by the
server, however this should not prevent implementers from applying leap second
smearing between the client and any clock it is training.

## Backwards compatibility to NTS and NTPv4

The support for compatibility with other protocols should not prevent addressing
issues that have previous caused issues in deployments or cause ossification of
the protocol.

Protocol ossification MUST be addressed to prevent existing NTPv4
deployments which incorrectly respond to clients posing as NTPv5 from causing
issues. Forward prevention of ossification (for a potential NTPv6 protocol in
the future) should also be taken into consideration.

The model for backward compatibility is servers that support multiple versions
NTP and send a response in the same version as the request. This does not
preclude servers from acting as a client in one version of NTP and
a server in another.

### Dependent Specifications

Many other documents make use of NTP's data formats ({{RFC5905}} Section 6) for
representing time, notably for media and packet timestamp measurements. Any
changes to the data formats should consider the potential implementation
complexity that may be incurred.

## Extensibility

The protocol MUST have the capability to be extended, and that implementations
MUST ignore unknown extensions. Unknown extensions received by a server from a
lower stratum server SHALL not be added to response messages sent by the server
receiving these extensions.

## Security

Data authentication and optional data confidentiality MUST be integrated into
the protocol, and downgrade attacks by an in-path attacker must be mitigated.

Cryptographic agility must be available, allowing for the protocol to update to
the use of more secure cryptographic primitives as they are developed and as
attacks and vulnerabilities with incumbent primitives are discovered.
Intermediate devices such as hardware capable of performing timestamping of
packets SHOULD be able to include information to packets in flight without
requiring modification or removal of authentication or confidentiality on the
packet.

Consideration must be given around how this will be incorporated into any
applicable trust model. Downgrading attacks that could lead to an adversary
disabling or removing encryption or authentication MUST NOT be possible in the
design of the protocol.

# Non-requirements

This section covers topics that are explicitly out of scope.

## Server malfeasence detection

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
are limited within the protocol with a clock which is not yet synchronised.
Increased path diversity and protocol support for synchronisation across
multiple heterogeneous sources are likely the most effective mitigations.

## Payload manipulation

Conversely on-path attackers who can manipulate timestamps could also speed up a
client's clock, also resulting into drift-related malfunctions and errors such
as incorrect expiration of public certificates on the affected hosts. An
attacker may also manipulate other data in flight to disrupt service and cause
de-synchronisation. In both cases having message authentication with a regular
key rotation interval should mitigate; however consideration should be made for
hardware based timestamping.

## Denial of Service and Amplification

NTPv4 has previously suffered from DDoS amplification attacks using a
combination of IP address spoofing with a private mode commands used in many NTP
implementations, leading to an attacker being able to orders of magnitude of
traffic to a victim IP address. Current mitigation uses a combination of
disabling the use of private mode commands, in addition to encouraging network
operators to implement BCP 38 {{RFC2827}}. Additional mitigations in future
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

The author would like to thank Doug Arnold and Hal Murray for contributions to
this document, and would like to acknowledge Daniel Franke, Watson Ladd,
Miroslav Lichvar for their existing documents and ideas. The author would also
like to thank Angelo Moriondo, Franz Karl Achard, and Malcom McLean for
providing the author with motivation.
