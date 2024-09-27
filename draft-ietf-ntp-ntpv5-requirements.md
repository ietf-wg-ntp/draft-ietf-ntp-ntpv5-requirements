---
title: "NTPv5 Use Cases and Requirements"
abbrev: "NTPv5 Use Cases and Requirements"
docname: draft-ietf-ntp-ntpv5-requirements-latest
date:

author:
    -
      ins: J. Gruessing
      name: James Gruessing
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
    RFC7384:
    RFC8085:
    RFC8174:
    I-D.ietf-ntp-roughtime:

informative:
    RFC4566:
    RFC7808:
    RFC8762:
    RFC9065:
    drdos-amplification:
        title: "Amplification and DRDoS Attack Defense -- A Survey and New Perspectives"
        target: https://arxiv.org/abs/1505.07892
    ntp-misuse:
        title: "NTP server misuse and abuse"
        target: https://en.wikipedia.org/wiki/NTP_server_misuse_and_abuse
    ntppool:
        title: "pool.ntp.org: the internet cluster of ntp servers"
        target: "https://www.ntppool.org"
    TF.460-6:
        title: "Standard-frequency and time-signal emissions"
        target: "https://www.itu.int/rec/R-REC-TF.460-6-200202-I/en"
    google-smear:
        title: "Google Leap Smear"
        target: "https://developers.google.com/time/smear"

--- abstract

This document describes the use cases, requirements, and considerations that
should be included in the design of a successor protocol to NTP version 4,
presently referred to as NTP version 5 ("NTPv5"). It aims to
define what capabilities and requirements such a protocol possesses, informing
the design of the protocol in addition to capturing any working group consensus
made in development.

--- note_Note_to_Readers

*RFC Editor: please remove this section before publication*

Source code and issues for this draft can be found at
<https://github.com/ietf-wg-ntp/draft-ietf-ntp-ntpv5-requirements>.

--- middle

# Introduction

NTP version 4 {{RFC5905}} has seen active use for over a decade, and within
this time period the protocol has not only been extended to support new
requirements but has also fallen victim to vulnerabilities that have been used
for distributed denial of service (DDoS) amplification attacks. In order to
advance the protocol and address these known issues alongside add capabilities
for future usage this document defines the current known and applicable use
cases in existing NTPv4 deployments and defines requirements for the future.

## Notational Conventions

{::boilerplate bcp14-tagged}

Use of time specific terminology used in this document may further be specified
in {{RFC7384}} or NTP specific terminology and concepts within {{RFC5905}}.

# Use Cases and Existing Deployments of NTP

As a protocol, NTP is used synchronise large amounts of computers via both
private networks and the open internet, and there are several common scenarios
for existing NTPv4 deployments: Publicly accessible NTP services such as the NTP
Pool {{ntppool}} are used to offer clock synchronisation for end user devices
and embedded devices. ISP-provided servers are used to synchronise devices such
as customer-premises equipment. Depending on the network and path, these
deployments may be affected by partial or complete packet loss, variable
latency, or throttling.

Data centres and cloud computing providers have also deployed and offer NTP
services both for internal use and for customers, particularly where the network
is unable to offer or does not require the capabilities other protocols can
provide, or where there may be familiarity with NTP already. As these
deployments are less likely to be constrained by network latency or power, the
potential for higher levels of accuracy and precision within the bounds of the
protocol are possible, particularly through the use of modifications such as the
use of bespoke algorithms.

# Threat Analysis and Modeling

A considerable motivation towards a new version of the protocol is the inclusion
of security primitives such as authentication and encryption to bring the
protocol in-line with current best practices for protocol design.

There are numerous potential threats to a deployment or network handling traffic
time synchronisation protocols that {{RFC7384}} section 3 describes, which can
be summarised into three basic groups: Denial of Service (DoS), degradation of
accuracy, and false time, all of which in various forms apply to NTP. However,
not all threats apply specifically to NTP directly, most notable attacks on time
sources (section 3.2.10) and L2/L3 DoS Attacks (section 3.2.7) as both are
outside the scope of the protocol, and the protocol itself cannot provide much
in the way of mitigations.

## Denial of Service and Amplification

NTPv4 has previously suffered from DDoS amplification attacks using a
combination of IP address spoofing and private mode commands used in some NTP
implementations, leading to an attacker being able to direct very large volumes
of traffic to a victim IP address. Current mitigations are disabling private
mode commands susceptible to attacks and encouraging network operators to
implement BCP 38 {{RFC2827}} as well as source address validation where
possible.

The NTPv5 protocol specification should be designed with current best practices
for UDP based protocols in mind {{RFC8085}}. It should reduce the potential
amplification factors in request/response payload sizes {{drdos-amplification}}
through the use of payload data padding to equalise the size of request and
response packets, in addition to restricting command and diagnostic modes which
could be exploited.

## Accuracy Degradation

The risk that an on-path attacker can systemically delay packets between a
client and server exists in all time protocols operating on insecure networks
and its mitigations within the protocol are limited for a clock which is not yet
synchronised. Increased path diversity and protocol support for synchronisation
across multiple heterogeneous sources are likely the most effective mitigations.

## False Time

Conversely, on-path attackers who can manipulate timestamps could also speed up
or slow down a client's clock, resulting in drift-related malfunctions and errors such as
premature expiration of certificates on affected hosts. An attacker may also
manipulate other data in flight to disrupt service and cause de-synchronisation.
Additionally attacks via replaying previously transmitted packets can also delay
or confuse receiving clocks, impacting ongoing synchronisation.

Message authentication with regular key rotation should mitigate all of these
cases; however deployments should consider finding an appropriate compromise
between the frequency of rotation to balance the window of attack and the rate
of re-keying.

# Requirements

At a high level, NTPv5 should be a protocol that is capable of operating in
local networks and over public internet connections where packet loss,
delay, and filtering may occur. It should provide both basic time information
and synchronisation.

## Resource Management

Historically there have been many documented instances of NTP servers receiving
ongoing large volumes of unauthorised traffic {{ntp-misuse}} and the design of
NTPv5 must ensure the effects of these can be minimised through the use of
signalling unwanted traffic (e.g. Kiss of Death) or easily identifiable packet
formats which make rate-limiting, filtering, or blocking by firewalls possible.

The protocol's loop avoidance mechanisms SHOULD be able to use identifiers that
change over time. Identifiers MUST NOT relate to network topology. In
particular they should not rely on any FQDN, IP address, or identifier
tied to a public certificate used or owned by the server. Servers SHOULD be
able to migrate and change the identifiers they use as stratum topologies or network
configurations change.

An additional identifier mechanism MAY be considered for the purposes of client
allow/deny lists, logging, and monitoring. Such a mechanism, when included, SHOULD
be independent of any loop avoidance mechanism, and authenticity requirements
SHOULD be considered.

The protocol MUST have the capability for servers to notify clients that the
service is unavailable and clients MUST have clearly defined behaviours for
honouring this signalling. In addition servers SHOULD be able to communicate to
clients that they should reduce their query rate when the server is
under high load, has reduced capacity, or otherwise wishes to limit traffic.

Clients SHOULD periodically re-establish associations with servers to detect
unresponsive hosts and to give operators
the ability to move traffic away from hosts in a timely manner.

The protocol SHOULD have provisions for deployments where Network Address
Translation occurs and define behaviours when NAT rebinding occurs. This should
not compromise any DDoS mitigation(s) that the protocol may define.

Client and server protocol modes MUST be supported. Other modes such as
symmetric and broadcast MAY be supported by the protocol but SHOULD NOT be
required of implementations. Considerations should be made in these
modes to avoid implementation vulnerabilities and to protect deployments from
attacks.

## Data Minimisation

To minimise ongoing use of deprecated fields and exposing identifying
information of implementations and deployments, payload formats SHOULD use the
smallest number of fields and the least amount of information possible, realising that data
minimisation and resource management can be at odds with one another. The use of
extensions should be preferred when transmitting optional data.

## Algorithms

The use of algorithms describing functions such as clock filtering, selection,
and clustering SHOULD have agility, allowing for implementations to develop and
deploy new algorithms independently. Signalling of algorithm use or preference
SHOULD NOT be transmitted by servers. However, essential properties of the
algorithm (e.g. precision) SHOULD be obvious.

The working group should consider creating a separate informational document to
describe an algorithm to assist with implementation, and consider adopting
future documents which describe new algorithms as they are developed. Specifying
client algorithms separately from the protocol will allow NTPv5 to meet
the needs of applications with a variety of network properties and performance
requirements.

## Timescales

The protocol should adopt a linear, monotonic timescale as the basis for
communicating time. The format should provide sufficient scale, precision, and
resolution to meet or exceed NTPv4's capabilities, and have a roll-over date
sufficiently far into the future that the protocol's complete obsolescence is
likely to occur first. Ideally it should be similar or identical to the existing
epoch and data model that NTPv4 defines to allow for implementations to better
support both versions of the protocol, simplifying implementation.

The timescale, in addition to any other time-sensitive information, MUST be
sufficient to calculate representations of both UTC and TAI {{TF.460-6}}, noting
that UTC itself as the current timescale used in NTPv4 is neither linear nor
monotonic, unlike TAI. Through extensions, the protocol SHOULD support additional
timescale representations outside of the main specification, and all
transmissions of time data MUST indicate the timescale in use.

## Leap seconds

Transmission of UTC leap second information MUST be included in the protocol in
order for clients to generate a UTC representation, but must be transmitted as
separate information to the timescale. The specification MUST require that
servers transmit upcoming leap seconds greater than 24 hours
in advance if that information is known by the server. If the server learns of a
leap second less than 24 hours before an upcoming leap second event, it MUST
start transmitting the information immediately.

Smearing {{google-smear}} of leap seconds SHOULD be supported in the protocol,
and the protocol MUST support servers transmitting information about whether they are
configured to smear leap seconds and whether they are actively doing so. Behaviours
for both client and server in handling leap seconds MUST be part of the
specification; in particular, how clients handle multiple servers where some may
use leap seconds and others smearing, that servers should not apply both leap
seconds and smearing, and details around smearing timescales. Supported
smearing algorithms MUST be defined or referenced.

## Backwards Compatibility with NTS and NTPv4

The desire for compatibility with older protocols should not prevent addressing
deployment issues or cause ossification of the protocol caused by middleboxes
{{RFC9065}}.

Servers that support multiple versions of NTP MUST send a response in the same
version as the request, to support backwards compatibility. This does not
preclude servers from acting as a client in one version of NTP and a server in
another.

Protocol ossification MUST be addressed to prevent issues caused by existing
NTPv4 deployments which respond incorrectly to clients posing as NTPv5. Forward
compatibility with a potential future NTPv6 protocol should also be considered.

### Dependent Specifications

Many other documents make use of NTP's data formats ({{RFC5905}} Section 6) for
representing time, notably for media and packet timestamp measurements, such as
SDP {{RFC4566}} and STAMP {{RFC8762}}. Any changes to the data formats should
consider the potential implementation complexity that may be incurred.

## Extensibility

The protocol MUST have the capability to be extended; implementations MUST
ignore unknown extensions. Unknown extensions received from a lower stratum
server SHALL NOT be re-transmitted towards higher stratum servers.

## Security

Data authentication and integrity MUST be supported by the protocol, with
optional support for data confidentiality. Downgrade attacks by an in-path
attacker must be mitigated. The protocol MUST define at least one common
mechanism to ensure interoperability, but should also include support for
different mechanisms to support different deployment use cases. Extensions
and additional modes SHOULD also incorporate authentication and integrity
on data which could be manipulated by an attacker, on-path or off-path.

Upgrading cryptographic algorithms must be supported, allowing for more secure
cryptographic primitives to be incorporated as they are developed and as attacks
and vulnerabilities with incumbent primitives are discovered.

Intermediate devices such as networking equipment capable of modifying NTP
packets, for example to adjust timestamps MUST be able to do so
without compromising authentication or confidentiality. Extension fields with
separate authentication may be used to facilitate this.

Consideration must be given to how this will be incorporated into any
applicable trust model. Downgrading attacks that could lead to an adversary
disabling or removing encryption or authentication MUST NOT be possible in the
design of the protocol.

# Non-requirements

This section covers topics that are explicitly out of scope.

## Server Malfeasance Detection

Detection and reporting of server malfeasance should remain out of scope as
{{I-D.ietf-ntp-roughtime}} already provides this capability as a core
functionality of the protocol.

## Additional Time Information and Metadata

Previous versions of NTP do not transmit additional time information such as
time zone data or historical leap seconds, and NTPv5 should not explicitly add
support for it by default as existing protocols (e.g. TZDIST {{RFC7808}})
already provide mechanisms to do so. This does not prevent however, further
extensions enabling this.

## Remote Monitoring Support

Due to their misuse in DDoS amplification attacks, mode 6 messages which have
historically provided the ability for monitoring of servers SHOULD NOT be
supported in the core of the protocol. However, they may be provided in a separate
extension specification. It is likely that even with a new version of the
protocol, middleboxes may continue to block this mode in default configurations
into the future.

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
