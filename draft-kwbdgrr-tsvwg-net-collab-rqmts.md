---
title: "Requirements for Network Collaboration Signaling"
abbrev: "Network Collaboration Requirements"
category: info

docname: draft-kwbdgrr-tsvwg-net-collab-rqmts-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: Transport
workgroup: "Transport and Services Working Group"
keyword:
 - Media
 - Low Latency
 - Wireless Links
 - Bandwidth constraints
 - Policy
 - Operational Considerations
venue:
  group: TSVWG
  type: Working Group
  mail: tsvwg@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/tsvwg/
  github:
  latest:

author:
 -
    ins: J. Kaippallimalil
    name: John Kaippallimalil
    organization: Futurewei
    email: john.kaippallimalil@futurewei.com
 -
    ins: D. Wing
    name: Dan Wing
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    email: danwing@gmail.com
 -
    ins: S. Gundavelli
    name: Sri Gundavelli
    organization: Cisco
    email: sgundave@cisco.com
 -
    ins: S. Rajagopalan
    name: Sridharan Rajagopalan
    organization: Cloud Software Group Holdings, Inc.
    abbrev: Cloud Software Group
    email: Sridharan.girish@gmail.com
 -
    ins: S. Dawkins
    name: Spencer Dawkins
    organization: Tencent America LLC
    email: spencerdawkins.ietf@gmail.com
 -
    fullname: Mohamed Boucadair
    organization: Orange
    city: Rennes
    code: 35000
    country: France
    email: mohamed.boucadair@orange.com



normative:

informative:

  TS.23.501-3GPP:
    title:  "3rd Generation Partnership Project; Technical Specification Group Servies and System Aspects; System architecture for the 5G System (5GS); Stage 2 (Release 18)"
    date: March 2023

  TR.23.700-70-3GPP:
     title:  "Study on XR (Extended Reality) and media services (Release 19)"
     date: August 2024

  5G-Lumos:
    title: "Lumos5G: Mapping and Predicting Commercial mmWave 5G Throughput, Arvind Narayanan et al., ACM Internet Measurement Conference (IMC '20), https://dl.acm.org/doi/10.1145/3419394.3423629"
    date: October 2020

  5G-Octopus:
    title: "Octopus: In-Network Content Adaptation to Control Congestion on 5G Links, Yongzhou Chen et al., ACM/IEEE Symposium on Edge Computing (SEC '23), https://dl.acm.org/doi/10.1145/3583740.3628438"
    date: December 2023

  app-measurement:
        title: "Bandwidth measurement for QUIC"
        date: 2024
        author:
        -
          fullname: Zafer Gurel
        -
          fullname: Ali C. Begen
        target: https://datatracker.ietf.org/doc/slides-119-moq-bandwidth-measurement-for-quic/

  traffic-analysis:
        title: "Encrypted Network Traffic Analysis and Classification Utilizing Machine Learning"
        date: 2024
        author:
        -
          fullname: Ibrahim A. Alwhbi
        -
          fullname: Cliff C. Zou
        -
          fullname: Reem N. Alharbi
        target: https://www.mdpi.com/1424-8220/24/11/3509
  traffic-analysis-2:
        title: "A real-world dataset of netflix videos and user watch-behavior"
        date: 2021
        target: https://www.gnan.ece.gatech.edu/archive/ICC_2021_Netflix_Insights.pdf


--- abstract

Collaborative signaling from client-to-network and server-to-network
can improve the user experience by informing the network about the
nature and relative importance of packets (frames, streams, etc.)
without having to disclose the content of the packets. Moreover,
the collaborative signaling may be enabled so that clients and
servers are aware of the network's treatment of incoming packets.
Also, client-to-network collaboration can be put in place without
revealing the identity of the remote servers. This collaboration
allows for differentiated services at the network (e.g., packet
discard preference), the sender (e.g., adaptive transmission), or
through cooperation of server/client and the network.

This document lists some use cases that illustrate the need for a
mechanism to share metadata and outlines requirements for both
client-to-network and server-to-network. The document focuses on
signaling information about a UDP transport flow (UDP 4-tuple).

--- middle

# Introduction {#intro}

Wireless networks, including 5G and WLAN, inherently experience large
variations in link quality over sub-RTT (round-trip time) intervals and on the other
hand, applications such as interactive media demand both low latency
and high bandwidth.

Superior service during adverse network events can be achieved by the
sender conveying packet behavior preferences to the network for
packets within a single UDP 4-tuple flow.  During adverse network events
this allows the network to be informed about the least-impactful
packets to drop (or delay) in the same UDP 4-tuple flow.  Without such
signaling, the network can only indiscriminately drop (or delay)
packets.  With such capability, loss-tolerant and delay-tolerant
transport protocols such as RTP {{?RFC3550}}, QUIC {{?RFC9000}}, and Unreliable
QUIC {{?RFC9221}} can inform the network and provide a superior end
user experience.

Some of the complications that are induced by adverse network events
may be eliminated by adequate dimensioning and upgrades.
However, such upgrades may not be always (immediately) possible or
justified. Complementary mitigations are thus needed to soften these
complications by introducing some collaboration between endpoints and
networks to adjust their behaviors.

{{sec-rationale}} discusses the rationale for per-packet metadata.

{{uc}} outlines use cases to illustrate the issues
and the need for additional information per flow to allow the network
to optimize its handling. {{metadata-req}} describes the requirements for on-path media
collaboration signals.

{{sys-considerations}} provides operational constraints in the network.

# Definitions

The document makes use of the following terms:

Per-Flow Metadata:
: Refers to metadata that doesn't change often during the lifetime
of a connection and thus can be exchanged once or as needed. This is communicated per flow (i.e., UDP 4-tuple) between client and network.
: Examples of such metadata are client request to honor per-packet metadata and preferences.

Per-Packet Metadata:
: Refers to metadata that varies packet to packet within the same flow, often capturing
the nature and characteristics of the traffic each packet carries. This needs to be communicated on a per packet basis.
: Examples of such metadata are Packet Priority and tolerance to delay

Reactive Management:
: Network management actions that are undertaken as a reaction to unplanned overload events.
Concretely, this includes policies which react to congestion events with very short to very
long durations (e.g., varying wireless and mobile air interface conditions) or protection
policies to soften the impact of ongoing attacks.

# Rationale for Per-packet Metadata {#sec-rationale}

Maximizing network utilization and enhancing perceived end user
experience under adverse network conditions are challenging. Factors that affect
wireless networks include change in channel conditions, interference
between proximate cells, and end user mobility. These variations
in link quality can be in the order of a millisecond or less
{{5G-Lumos}} while congestion control takes several tens of
milliseconds (more than one RTT) to estimate data
rate over a specific path. Similarly, application servers that encode and serve live or
interactive content (media, typically) take time to adjust the encoding level and other
processes to match the network rate. End-to-end congestion control
algorithms are far from being optimal when the link quality is
highly variable in sub-RTT timeframes and the application demands
both low latency and high bandwidth (e.g., {{Section 2.1 of
?RFC6077}}). In these conditions, applications settle for a lower
throughput when latency is prioritized, or for higher throughput
at the expense of much higher delays.

While rate control based on feedback for a flow (UDP 4-tuple) is
evidently not able to adapt for sub-RTT changes in available wireless
channel resources, an application server can provide information
on a per-packet basis that a network shaper may use to allocate the
available resources more effectively. For example, {{5G-Octopus}} has shown for
volumetric video packets and a rate controller that errs on the
side of overestimation, that the network shaper can drop low priority
frames of a group of pictures (corresponding to transient wireless
link bandwidth drops) and still achieve significantly better
performance than state-of-the-art based on feedback. With not fully
encrypted packets, networks may use heuristics to build an "implicit
signal" gleaned from a packet to prioritize or otherwise shape
flows. However, implicit signals are not desirable as they lead to
ossification of protocols as result of introducing unintended
dependencies {{?RFC9419}}. When packet contents are encrypted, the
approach of using implicit signals is no longer viable.

Bandwidth constraints exist most predominantly at the access network
(e.g., radio access networks). Users who are serviced via these
networks use clients which run various applications; each having
different connectivity needs for an optimal user experience. These
needs are not frozen but change over time depending on the application
and even depending on how an application is used (e.g., user's
preferences). An explicit signal to the client can improve management
of available bandwidth.

Other applications like interactive media can demand both high
throughput and low latency and, in some cases, carry different
streams (e.g., audio and video) in a single transport connection
(e.g., WebRTC {{?RFC8825}}).

With RTP {{?RFC3550}}, the media type could be examined and used
as an implicit signal for determining relative priority. However,
{{?RFC9335}} defines a new mechanism that completely encrypts RTP
header extensions and Contributing sources (CSRCs). Furthermore, a
full encrypted transport (e.g., QUIC {{?RFC9000}}) does not expose
any media header information to influence on-path network elements
during a Reactive Management event.

~~~~~~~~aasvg
                     :
       3GPP/mobile network
+--------------------:----------------------+
| +------+           :   +-----+    +-----+ |
| |client+-----------B---+radio+----+  CN | |
| +------+           :   +-----+    +--+--+ |
+--------------------:-----------------|----+
                     :                 |
       Wireless home/ISP network       |
+--------------------:-----------+     |    :          :
| +------+   +----+  :  +------+ | +---+--+ : +------+ : +------+
| |client+-B-+WLAN+--B--+router+---+router+---+router+---+server|
| +------+   +----+  :  +------+ | +------+ : +------+ : +------+
+--------------------:-----------+          :          :
                     :                      :          :
                     :                      : Transit  :  Content
 User device/Network :    MNO/ISP Network   : Network  :  Network
~~~~~~~~
{: #Figure-e2e title="E2E Media Transport Overview"}

{{Figure-e2e}} shows where such bandwidth and performance constraints
usually exist with a "B" (for Bottleneck) in 3GPP/mobile networks
and WLAN/ISP networks.  When a bottleneck exists temporarily, the
network has no choice but to discard or delay packets -- which can
harm certain flows and, thus, lead to suboptimal perceived experience.
In this document, this is termed 'Reactive Management'.

~~~~~~~~aasvg
            (A) Application signaling (client - server)
     +--------------------------------------------------------+
    /                                                          \
+--+---+              +------------+                         +--+---+
|      |   (C) C2N    |            |                         |      |
|      |<------------>| +--------+ |                         |      |
|      |              | | Network| |  downstream packet      |      |
|      |<=============+=+ Shaper +<+=========================+      |
|      |              | +--------+ | (B) on-path S2N metadata|      |
+------+              +------------+                         +------+
 Client                   Router                              Server
~~~~~~~~
{: #Figure-netshaper align="center" title="Metadata and Network Shaping"}

{{Figure-netshaper}} shows a bottleneck (access) router on the path
of packets from Server to Client.  A network shaper in the router
manages QoS of flows of multiple users and can buffer (delay),
discard, or apply other flow control rules. Application layer
signaling and feedback between Client - Server (A - in the figure)
adjust transmission rate over a period of several RTTs using feedback
and congestion control algorithms.  Congestion control algorithms (CCAs)
are generally conservative and settle to a steady rate that avoids
excessive packet loss.  In networks where link conditions (between
Client and Router) vary significantly at timescales well below the
RTT, this results in unused (wasted) bandwidth at short timescales.

There is some research (e.g., {{5G-Octopus}}) to indicate that media
applications can obtain better Quality of Experience (QoE) when sending at a higher rate
(less conservative than current CCA) and the media application is
willing to tolerate some packet loss or delay of low priority
packets.  Packet priority and tolerance to delay of packets in such
a case would be provided on-path in a side channel associated to
the downstream packet (B - in the figure).  The requirements for this
server-to-network (S2N) metadata are described in {{server-network}}.

The client may provide information to an (access) router to drop
lower priority marked packets of a flow (UDP 4-tuple) temporarily
which can in turn allocate available bandwidth to other flows of that network
attachment, especially during a Reactive Management event.

Network shapers
observe flows and apply policies to maximize performance but are
not aware whether  there is a high preference for one flow (UDP
4-tuple) over another flow belonging to the same user and network
attachment (e.g., a subscriber connection, a 3GPP PDU Session. See
{{net-attach}} for more details).
Clients can provide information to an (access) router
to drop 'lower priority'-marked
packets of a flow (UDP 4-tuple) during a Reactive Management event.


In summary, the rapid variation of wireless link quality and/or
bandwidth limitations in networks along with interactive applications
that demand low latency and high throughput can lead to suboptimal
user experience.

# Requirements Definition {#req-definition}

## Server-to-Network

REQ-PACKET-PRIORITY:
: Server indicates the importance of a packet within a flow. This allows the network
to prioritize based on requirement and during a Reactive Management event. This
priority value may also be used to indicate loss tolerance and the
network elements may drop loss tolerant packets during Reactive Management
events.
: This is a per-packet metadata requirement.

REQ-PACKET-DELAY:
: Metadata to indicate whether the packet can tolerate delay.
: This is a per-packet metadata requirement.

## Client-to-Network

REQ-CLIENT-DECIDES:
: User/Client indicating to the network to honor the application's
metadata signaling.
: This is a per-flow metadata requirement.

REQ-PAYLOAD-CLIENT-DECIDES:
: The ability of the receiver to change the priority by communicating
to the network to prioritize one payload(metadata) over another within
the flow -- without cooperation of the sender. Gives the sender the
ability to have same metadata for all the connections without having
to change based on the user preference, aids in scalability.
: This is a per-flow metadata requirement.

## API

REQ-API-FRAMEWORK:
: API framework to facilitate signaling for applications.
: Signaling from client to network ({{client-flow-auth}})
and server to network ({{server-network}}))
is best facilitated by Application Programming Interfaces
(APIs). Signaling and retrieval of
the signals may not be performed at a single layer (although not
encouraged). Hence, for server to network signaling, a framework is required to abstract the underlying
per-packet metadata protocol(s) and allow the application(s) to retrieve/send signals
using a single or a set of APIs independent of the channels that
are used to convey the signals. The API framework is required even
if one single channel is used so that any application on a client can
consume the signals.
: The API framework uses the medium negotiated under {{metadata-negotiation}} to send/receive the signals


## System Considerations

REQ-PRIVACY-ADDITIONAL:
: An on-path observer obtains (or gleans) no additional information about the IP
packet.

REQ-SIGNALING-AVOIDANCE:
: Leveraging previous experience {{?RFC9049}}, the following is not
required to make use of the collaborative signaling:

  * Reveal the application identity.
  * Expose the application cause (or 'reason') to signal metadata.
  * Reveal the server identity.
  * Inspect client-to-server encrypted payload by network elements.

# Use Cases {#uc}

## Media Streaming {#uc-streaming}

Streaming video contains the occasional key frame ("I-frame")
containing a full video frame.  These frames are necessary to rebuild
receiver state after loss of delta frames.  The key frames are
therefore more critical to deliver to the receiver than delta frames.

Examples: live broadcast, on-demand video streaming.

Use cases:

1. Majority of streaming traffic is Audio and Video traffic.
Video contains partial frames and full frames, which need to be
distinguished so that full frames can be indicated to the network.
Audio traffic is more critical than video for many applications.
This differences in priority needs to be indicated to the network to
ensure network (de)prioritizes (or even drop if deemed necessary)
traffic appropriately.

    Requirement: REQ-PACKET-PRIORITY.

    Impact: With the above requirement met, better quality of service
    could be maintained in resource-constrained networks and during
    Reactive Management events ensuring better user experience.

2. The server (or relay) sends the same stream to many receivers,
including the same metadata (especially with media over QUIC).
Different clients have different priorities for different types of traffic.
This would result in change in priorities for the same type of traffic
that a single server sends, based on the user/client.

    Requirement: REQ-PAYLOAD-CLIENT-DECIDES.

    Impact: With the above requirement met, each client/user preferences are
    prioritized accordingly while maintaining scalability on the server, since
    the metadata that the server sends still remains the same for all the connections.

3. In loss-prone networks or during Reactive Management events, if all packets of an application
flow (UDP 4-tuple) such as live broadcast or on-demand video streaming are treated the same,
it limits the ability to maximize network utilization and use the transiently available bandwidth.
Dropping or delaying of (media) packets randomly is likely to lower network utilization
and application performance.

    Requirement: REQ-PACKET-PRIORITY, REQ-PACKET-DELAY.

    Impact: By identifying packets that tolerate being dropped, congestion can be reduced leading
    to improved performance/quality of service.


## Interactive Media {#uc-interactive}

Interactive media includes content that a user can actively engage
with and results in input and response actions that can be highly
delay-sensitive. This can also include mixed traffic based on the
user activity and interaction.

Examples: VoIP (Peer-to-Peer (P2P), group conferencing), gaming,
Remote Desktop Virtualization, eXtended Reality (XR).

Use cases:

1. A mobile/roaming user prioritizes audio over video during a VoIP
call to have a seamless meeting experience.

    Requirement: REQ-PAYLOAD-CLIENT-DECIDES.

    Impact: With the above requirement met, each client/user preferences are
    prioritized accordingly while maintaining scalability on the server.

2. A remote desktop user prioritizes graphics updates over an on-going
file copy operation. A user types in/interacts with a document/file after triggering a
save file operation, while save operation is on-going.

    Requirement: REQ-PACKET-PRIORITY, REQ-PACKET-DELAY.

    Impact: By prioritizing graphic updates/interactive traffic, user
    interactivity is improved with lesser jitter.

3. A game or VoIP application may want to signal different metadata
for the same type of packet in each direction. One user, in a VoIP
conference call, wants to prioritize the slide deck being shared while
the other wants to prioritize audio and other wants to prioritize
video of the speaker. Each user's varied preferences can be catered
with same type of metadata originating from the server.

    Requirement: REQ-PACKET-PRIORITY, REQ-PAYLOAD-CLIENT-DECIDES.

    Impact: With the above requirement met, each client/user preferences are
    prioritized accordingly while maintaining scalability on the server.

5. A network glitch while user is in an eXtended Reality application.
The traffic comprises of haptic, video, audio, graphics update and
keystrokes. During such a Reactive Management event, some packets need to be
deprioritized/dropped to maintain interactivity.

    Requirement: REQ-PACKET-PRIORITY, REQ-PACKET-DELAY.

    Impact: By prioritizing high priority traffic, user's
    interactive experience is improved with lesser jitter.

## Metadata Negotiation Support {#metadata-negotiation}

Currently, some flows are granted higher priority over other flows
because of a contractual agreement between the ISP and the content
provider. These contracts could be extended to also allow per-packet
metadata within a single UDP 4-tuple, as desired by this
document ({{client-flow-auth}}). However,
these sorts of agreements favor large content providers and major
ISPs, disfavoring smaller providers and smaller ISPs while also
preventing other network topologies such as peer-to-peer networking
(e.g., VoIP) as that traffic does not originate from a contracted
content provider.

For all applications to benefit from per-packet prioritization within
a single UDP 4-tuple, the client needs to communicate with the ISP to determine which per-packet
markings are supported by the ISP's network (e.g., encoded into IPv6 Flow Label,
UDP Option, or DSCP). Then it can indicate to the ISP's network that a
certain UDP 4-tuple will have those markings and instruct the server
to generate those per-packet metadata markings.

There might be many channels to signal the Server-to-Network per-packet metadata such as
(non-exhaustive list):

  * TCP options {{?RFC9293}}
  * UDP Options {{?I-D.ietf-tsvwg-udp-options}}
  * IPv6 Hop-by-Hop Options ({{Section 4.3 of ?RFC8200}})
  * QUIC CID mapping
  * ICMP messages

Requirements: REQ-API-FRAMEWORK and REQ-CLIENT-DECIDES.

Impact: By signaling ISPs to honor the metadata for a particular flow, the client
  facilitates identifying important packets to the ISP enhancing packet delay or
  drop decisions during Reactive Management events.


~~~~~aasvg
     Client                                           ISP router
       |                                                   |
       |-------------------------------------------------->|
       | Network Collaboration Capabilities?               |
       |                                                   |
       |<--------------------------------------------------|
       | my Network Collaboration capabilities             |
       |                                                   |
       |-------------------------------------------------->|
       | Server will mark packets using "method #4"        |
       |                                                   |
       |<--------------------------------------------------|
       | ok                                                |

~~~~~
{: #client-learn artwork-align="center" title="API Framework: Client
learns ISP capabilities and signals how incoming IP packets will
signal network collaboration"}


# On-path Metadata Requirements {#metadata-req}

There are various approaches for collaborative signaling between
the server/client and network including out-of-band signaling,
client-centric metadata sharing and proxied connections. The
requirements here focus on proxied metadata connections on path
with the data traffic.

The path signals below should follow the principles of intentional
distribution, protection of information, minimization and limiting
impact as described in {{?RFC9419}} and {{?RFC8558}}. Leveraging
previous experience ({{?RFC9049}}), the metadata signals do not
need application identity, application cause (or 'reason'), server
identity or the inspection of client-to-server encrypted payload.

The metadata connections may be between server and network (in
either direction) or between client and network (in either direction).

Some use cases benefit from server-network metadata exchanges
({{server-network}}) after first performing a client-network metadata
exchange ({{client-flow-auth}}).

For the requirements that follow, the assumption is that the client
agrees to the exchange of metadata between the server and network,
or between the client and network.

## Client-Network Flow Authorization and Negotiation {#client-flow-auth}

By signaling the ISP, a client can authorize the ISP to honor
incoming per-packet metadata for a certain flow (UDP 4-tuple).

This same signal also allows negotiating capabilities
discussed in {{metadata-negotiation}}) and sharing the keys
necessary for encrypting or obfuscating server-to-network per-
packet metadata recommended in {{privacy}}.

REQ-CLIENT-DECIDES is satisfied by signaling Client Flow Authorization as part of client-to-network signal.

## Server-Network Metadata {#server-network}

Application flows (UDP 4-tuple) for live media, eXtended Reality
(XR) and gaming require both high bandwidth and low latency and
would as such benefit from being able to use the bandwidth available
for the flow. In wireless networks, some of the bandwidth available
for the flow is not possible to schedule using the feedback based
rate control (due to the significant link variations at sub-RTT
timescales). In such networks where variations in link quality is
well below RTT, congestion control algorithms settle to a steady
rate that avoids excessive packet loss. Feedback via ECN/L4S
{{?RFC9331}} provides an accurate signal but is also on an RTT
timescale and thus does not provide finer resolution information
of instantaneous bandwidth available.

If application packets can either tolerate delay or some loss of
lower priority packets, the network traffic shaper and scheduler
can use this information to provide a higher application quality
of service. There is some research {{5G-Octopus}} to indicate that media
applications can obtain better measured application quality when
sending at a higher rate (less conservative than current CCA) and
allowing the network to delay or drop low priority packets.

The metadata in {{relative-priority}} should also satisfy constraints
identified in {{sys-considerations}}. Privacy ({{privacy}}) requires that
metadata should not provide additional information to identify the
application or the user. The application server can decide on the
metadata values that provide the best handling for the packets of
the flow.
This
metadata is advisory in nature and network traffic policy
that restricts its use would not result in additional issues. Other
constraints including scale ({{scalability}}) and continuity
({{continuity}}) are required for {{relative-priority}}.

Realizing the additional bandwidth potential with these metadata
may require a higher sending rate for the transport flow. This
requires work that is not specified in this document. Similarly,
the assumption is that network shapers and schedulers can use the
metadata in {{relative-priority}} but further details are out of
scope.

Previous work in {{TR.23.700-70-3GPP}} has identified the general
problem in this section. However, the solution in {{TS.23.501-3GPP}}
is specific to a 5G network. The metadata sent from a (dedicated
5G) application server identifies PDU set information and end-of-burst
signals which are not understood by non-3GPP systems such as Wi-Fi
or DOCSIS. Further, 3GPP functions and policy configurations are
required since this is a 5G specific solution. The metadata disclosed
in the 5G solution also identifies frame boundaries and does not
fully conform to the constraints for privacy or minimality of data
identified in {{sys-considerations}}.

### Packet Priority {#relative-priority}

Per-packet priority information provides the priority level of one
packet relative to other packets within a transport flow (UDP
4-tuple). When a packet is marked with high priority, the expectation
is that during a Reactive Management event, the network will give high importance to forwarding the
packet compared to a packet marked with low priority.
The application server can decide on the priority or
importance values that provide the best handling for the packets
of the transport flow.
When more than one application stream (e.g., video, audio)
is sent on the same transport flow, the application server decides
the best allocation of priority values across the different streams
of the flow.

Per-packet priority or importance determines the drop priority of
a packet.

REQ-PACKET-PRIORITY is satisfied by signaling Packet Priority as part of server-to-network metadata.

### Tolerance to Delay {#delay}

Some packets of a media flow (UDP 4-tuple) can tolerate more delay
over the wire than others (e.g., packets carrying live media frames
require very low latency while packets carrying a background image
for augmented reality can tolerate more delay). The objective of
this metadata is to indicate that these packets can tolerate a
limited amount of delay when there is severe congestion or limited
bandwidth. Similar to the LE PHB {{?RFC8622}} for flows, the
expectation is that in this case, each packet marked with this
metadata is dropped during a Reactive Management event. As with
per-packet priority in {{relative-priority}}, the application server
can decide on the metadata values that provide the best handling
for the packets of the transport flow.

REQ-PACKET-DELAY is satisfied by signaling Tolerance to Delay as part of server-to-network metadata.

# System Considerations {#sys-considerations}

Traffic policing and shaping are enforced in ingress/egress network
points for various reasons (protect the network against attacks,
ensure conformance with a traffic profile, etc.). Out-of-profile
traffic may be discarded or assigned another class (e.g., using Lower
Effort Per-Domain Behavior (LE PDB) {{?RFC3662}}) a bandwidth limit
among others. The exact behavior is policy-based and deployment-specific.

The entire set of operations to manage traffic is beyond the scope
of this document.  This section focuses on operational constraints
that impact  server-network, and client-network modes of sending
metadata.

## Privacy Considerations {#privacy}

Media flows are vulnerable to traffic analysis even without per-packet
metadata (see, e.g., {{traffic-analysis}}). The security aspects of the
media payload / transport are not in the scope of this document; these
are mentioned here only to provide context for metadata privacy.

Protocols such as TLS, SRTP, and QUIC offer some mitigations (like padding)
but are vulnerable to traffic analysis ({{traffic-analysis-2}}).


Per-packet metadata can aid in traffic analysis. Hence, it is recommended to
encrypt or obfuscate the metadata information so it is only available to the
server, client, and authorized network elements. However, encryption/obfuscation
of per-packet metadata is ineffective if the threat resides in the same network
entity with keys to decrypt the metadata. The method of encryption or
obfuscation is out of scope.  To best preserve privacy, implementations might
also consider less granular per-packet marking, for example marking all
audio and video packets the same and only marking a background data transfer
with different metadata.

Analysis to ensure that metadata exposure does not compromise user privacy
or allow unauthorized entities to infer sensitive information, while
maintaining minimal resource consumption is crucial. There is a tension
between resource consumption of such encryption and the user's privacy
({{Section 7.4 of ?RFC6973}}).


REQ-PRIVACY-ADDITIONAL and REQ-SIGNALING-AVOIDANCE are satisfied by
not revealing any information that could identify the application's identity, reason to signal,
server identity, and securing the metadata.

# Non-Requirements {#non-req}

Application feedback with measurements of packets lost and delay
incurred may affect the sending rate and other behavior of the
application.  The requirements and specification to mitigate these
aspects are not in the scope of this document.


# IANA Considerations {#iana}

None.

# Security Considerations {#sec-cons}

Security aspects for the metadata are discussed in {{privacy}}.
The principles outlined in {{?RFC8558}}, {{?RFC9049}}, and {{?RFC9419}}
contain security considerations and are referenced in {{metadata-req}}.

Per-packet metadata can be vulnerable to modification in transit by
on-path attackers, who can corrupt checksums, drop packets, or modify
metadata. Such changes can be detected by the receiver.

Since the document focuses only on priorities within a flow
(not specifying inter-flow priority), the document does not induce concerns related to a specific
user or client declaring all flows or a subset of them as being more important. Such abuse concerns are thus not applicable.

Privacy-related considerations are discussed in {{privacy}}.

--- back


# Network Attachment {#net-attach}

A network attachment represents the communication link between
clients and network (router) over which a connection policy (including
QoS) is applied to flows within that network attachment.

A network attachment may be established using control plane signaling
between the client and the network (access) router and is out of
scope of this document.  Transport flows over a network attachment
may consist of multiple streams such as video or audio.
{{Figure-conn-flow}} shows a high level view of network attachments,
flows, and QoS/policy discussed in {{metadata-req}}.

The requirements in {{metadata-req}} apply to data units like frames
within a flow, but not between flows. Specifically, this document
does not discuss flows of distinct clients/users.

~~~~~~~~aasvg
+--------------+          +-----------------+
| +---+ +---+  |          | +-------------+ |
| |A1 | |A2 |  |          | | QoS, Policy | |
| +-+-+ +-+-+  |          | +---+----+----+ |       +------+
|   |     |    |  Network |     |    |      |       |srv-A2|
|   |     |    |attachment|     v    |      |       +--+---+
|   |     |  .------------------+-.  |      |          |
|   |     +-+------- flow-x2 ------+-|------|----------+
|   +-------+------- flow-x1 ------+-|------|------------+
|            '--------------------'  |      |            |
|              |          |          |      |            |
+--------------+          |          |      |         +--+---+
  Client-1                |          |      |         |srv-A1|
                          |          |      |         +--+---+
+--------------+  Network |          |      |            |
|              |attachment|          v      |            |
| +----+  .--------------------------+-.    |            |
| | A1 +-+----------- flow-x3 ----------+---|------------+
| +----+  '----------------------------'    |
|              |          |                 |
+--------------+          +-----------------+
  Client-2                  Router
~~~~~~~~
{: #Figure-conn-flow title="E2E transport flows and connection session"}


{{Figure-conn-flow}} shows "Client-1" and "Client-2" which negotiate
connection policy (e.g., QoS) and other aspects like mobility
handling, charging applied to flows in that network attachment.
"Client-1" has "flow-x1" and "flow-x2" over its network attachment
while "Client-2" has "flow-x3".  **The requirements in this document
focus on on-path collaboration signals that apply to data units
such as media frames within flows like "flow-x1/x2/x3" but not
between them.**

# Extended Requirements Definition {#x-req-def}

REQ-MEDIA-AV-SEPARATE:
: Audio can be prioritized differently than video.
: This requirement may be generalized to non-media packet types.
: This is an enhanced requirement that requires e2e application layer
signaling (out of scope here) to identify of frame boundaries and may
not be suitable in cases which are sensitive to traffic analysis
(see REQ-SIGNALING-AVOIDANCE and {{RFC9049}}). If the application
provides frame boundaries, the client signals the enhanced application
priority values in REQ-PAYLOAD-CLIENT-DECIDES.
: This is a per-flow metadata requirement.

REQ-MEDIA-KEYFRAME:
: Video contains partial frames and full frames, which need to be
distinguished so that full frames can be indicated to the
network.
: This is an enhanced requirement that requires e2e application layer
signaling (out of scope here) to identify of frame boundaries and may
not be suitable in cases which are sensitive to traffic analysis
(see REQ-SIGNALING-AVOIDANCE and {{RFC9049}}). If the application
provides frame boundaries, the client signals the enhanced application
priority values in REQ-PAYLOAD-CLIENT-DECIDES.
: This is a per-packet metadata requirement.

REQ-NETWORK-THROUGHPUT:
: A mechanism to signal the available network throughput to interested
clients, including changes to throughput.

REQ-NRLP:
: The network shall inform the endpoint of the Rate limiting policies.

REQ-SIGNAL-EXPOSURE-FAIRNESS:
: Means to expose the signal independent of the application should be
considered. An example of such exposure is OS APIs.

REQ-NETWORK-SEEKS-LOAD-DOWN:
: During detected Reactive Management events, the network implements a
reactive traffic policy to reduce or offload some of the traffic.
: This may involve utilizing alternative network attachments
available to the client (e.g., Wi-Fi).

REQ-CONTINUITY:
: Handover from one radio or router to another should continue to
provide same service level.

REQ-MULTIPLE-BOTTLENECKS:
: Signaling should support Multiple bottlenecks.
: The network must identify multiple bottlenecks, including those
within the ISP and subscriber networks, ensuring all bottlenecks
benefit from network/client collaboration to enhance overall performance.

REQ-SCOPED-METADATA:
: Means to characterize the scope of a shared metadata for the sake of
better interoperability should be supported.
: The metadata can be used by the network to prioritize traffic
within a single 5-tuple connection and metadata cannot be leveraged
for prioritization between different flows.

REQ-SINGLE-CHANNEL:
: The network should use a single channel for sharing metadata
to simplify the process and avoid the need for redundant functions.

REQ-ISP-SCALE:
: The metadata and other state information that a router has to
maintain for each additional flow it handles should be kept
to a minimum or eliminated altogether.

# Extended Use-Cases

## Media Streaming Extended

Streaming video contains the occasional key frame ("I-frame")
containing a full video frame.  These frames are necessary to rebuild
receiver state after loss of delta frames.  The key frames are
therefore more critical to deliver to the receiver than delta frames.

Examples:

1. Audio is more critical than video for many applications and should
be prioritized differently than video.

Requirement: REQ-MEDIA-AV-SEPARATE.

2. Video contains partial frames and full frames, which need to be
distinguished so that full frames can be indicated to the network.

Requirement: REQ-MEDIA-KEYFRAME.

## Assisted Offload

There are cases (crisis) where "normal" network resources cannot
be used at maximum and, thus, a network would seek to reduce or
offload some of the traffic during these events -- called
'Reactive Management' policy.  An example of such use case is cellular
networks that are overly used (and radio resources exhausted) such
as a large collection of people (e.g., parade, sporting event), or
such as a partial radio network outage (e.g., tower power outage).
During such a condition, an alternative network attachment may be
available to the client (e.g., Wi-Fi).

Network-to-client signals are useful to put in place adequate traffic
distribution policies on the client (e.g., prefer the use of alternate
paths, offload a network).

Requirement: REQ-NETWORK-SEEKS-LOAD-DOWN.

## Network Bandwidth & Network Rate Limiting Policies {#nrlp}

Bandwidth constraints exist most predominantly at the access network.
This can be constraints in the network itself or a result of rate
limiting due to various reasons.

Also, traffic exchanged over a network attachment may be subject
to rate-limit policies. These policies may be intentional policies
(e.g., enforced as part of the activation of the network attachment
and typically agreed upon service subscription) or be Reactive Management
policies (e.g., enforced temporarily to manage an overload or during
a DDoS attack mitigation).

Requirements: REQ-NETWORK-THROUGHPUT, REQ-NRLP.

Use cases:

  1. Performance Optimization: Some applications support some forms
  of bandwidth measurements (e.g., {{app-measurement}}) which feed
  how the content is accessed to using ABR. Complementing or replacing
  these measurements with explicit signals will improve overall
  network performance and can help optimize the data transfer.
  Signaling bandwidth availability allows endpoints to avoid
  contributing to network congestion. When the network informs the
  endpoint about available bandwidth, the endpoint can dynamically
  adjust its data transmission rate. Knowing available bandwidth
  helps the endpoint allocate resources efficiently. Cloud-based
  applications can auto-scale based on available bandwidth.

  2. Rate Limiting: Monthly data quotas on cellular networks can
  be easily exceeded by video streaming, in particular, if the
  client chooses excessively high quality or routinely abandons
  watching videos that were downloaded. The network can assist the
  client by informing the client of the network's bandwidth policy.


# Extended System Considerations

## Application Interference {#app-interference}

Applications that have access to a resource-quota information may
adopt an aggressive behavior (compared to those that don't have
access) if they assumed that a resource-quota like metadata is for
the application, not for the client that runs the applications.

This is challenging for home networks where multiple clients may
be running behind the same CPE, with each of them running a video
application. The same challenge may apply when tethering is enabled.

Requirement: REQ-SIGNAL-EXPOSURE-FAIRNESS.

## Redundant Functions and Classification Complications {#classification}

If distinct channels are used to share the metadata between a client
and a network, a network that engages in the collaborative signaling
approach will require sophisticated features to classify flows and
decide which channel is used to share metadata so that it can consume
that information.

Likewise, the network will require to implement redundant functions;
for each signaling interface.

*As such, application- and protocol-specific signaling channels are
suboptimal.* (REQ-SINGLE-CHANNEL)

Requirement:  REQ-SINGLE-CHANNEL is preferred.

## Session Continuity {#continuity}

The frequency of handovers increases when a user moves
faster or when the media session lasts longer. During handovers,
there should be minimal delay incurred during handover in
configuring/setting up the metadata of a media session in progress.

Requirement: REQ-CONTINUITY.

## Multiple Bottlenecks

Whereas models often show a single bottleneck, there are frequently
two (or more) bottlenecks: the ISP network and within the subscriber
network (e.g., Wi-Fi link). As such, all bottlenecks near the
subscriber should be able to benefit from network/client collaboration.

Requirement: REQ-MULTIPLE-BOTTLENECKS.

## Metadata Scope {#metadata-scope}

An operational challenge for sharing resource-quota like metadata
(e.g., maximum bitrate) is that the network is generally not entitled
to allocate quota per-application, per-flow, per-stream, etc. that
delivered as part of an Internet connectivity service. However, the
network has a visibility about the overall network attachment (e.g.
inbound/outbound bandwidth discussed in
{{?I-D.ietf-opsawg-teas-attachment-circuit}}).

Hints about resource-like metadata is bound by default to
the overall network attachment, not specific to a given application
or flow.

It is out of the scope of this document to discuss setups (e.g.,
3GPP PDU Sessions) where network attachments with Guaranteed Bit
Rate (GBR) for specific flows is provided.

Requirement: REQ-SCOPED-METADATA.

## Scalability {#scalability}

There may be a large number of flows handled by the server
and wireless/access router. Per flow information (state) at a
wireless router for optimizing the flow can negate the advantages
offered as the number of flows handled increases.

Requirement: REQ-ISP-SCALE.

# Acknowledgments
{:numbered="false"}

This document is a merge of {{?I-D.rwbr-tsvwg-signaling-use-cases}}
and {{?I-D.kaippallimalil-tsvwg-media-hdr-wireless}}.

T. Reddy contributed text and ideas to this document.

Acknowledgments from {{?I-D.kaippallimalil-tsvwg-media-hdr-wireless}}:
: Xavier De Foy and the authors of this draft have discussed the
similarities and differences of this draft with the MoQ draft for
carrying media metadata.
: The authors wish to thank Mike Heard,
Sebastian Moeller and Tom Herbert for discussions on metadata fields,
fragmentation and various transport aspects.
: The authors appreciate
input from Marcus Ilhar and Magnus Westerlund on the need to address
privacy in general and Dan Druta to consider a common transport
across various client/server to network signaling when possible.
Ruediger Geib suggested that limiting the amount of state information
that a wireless router has to keep for a flow should be minimized.
: Ingemar Johansson's suggestions on fast fading (which L4S handles)
and dramatic drops in wireless accesses have been helpful to identify
the issues.  Thanks to Hang Shi for the review and comments on
client-to-network signaling.  Thanks to Luis Miguel Contreras, Colin
Kahn, Marcus Ilhar and Tianji Jiang for their review and comments.
