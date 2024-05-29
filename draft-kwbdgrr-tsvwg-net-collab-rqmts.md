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
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"


normative:

informative:

  TR.23.501-3GPP:
    title:  "3rd Generation Partnership Project; Technical Specification Group Servies and System Aspects; System architecture for the 5G System (5GS); Stage 2 (Release 18)"
    date: March 2023

  TR.23.700-60-3GPP:
    title:  "Study on XR (Extended Reality) and media services (Release 18)"
    date: August 2022

  5G-Lumos:
    title: "Lumos5G: Mapping and Predicting Commercial mmWave 5G Throughput, Arvind Narayanan et al., ACM Internet Measurement Conference (IMC '20), https://dl.acm.org/doi/10.1145/3419394.3423629"
    date: October 2020


--- abstract

Wireless networks (e.g., cellular or WLAN) experience significant but transient variations in link quality and have policy constraints (e.g., bandwidth) that affect user experience.
Collaborative signaling (e.g., host-to-network and server-to-network) can improve the user experience by informing the network about the the nature and relative importance of packets (frames, streams, etc.) without having to disclose the content of the packets. Moreover, the collaborative signalling may be enabled so that hosts are aware of the network's treatment of incoming packets. Also, host-to-network collaboration can be put in place without revealing the identity of the remote servers. This collaboration allows for differentiated services at the network (e.g., packet discard preference), the sender (e.g., adaptive transmission), or through cooperation of server / host and the network.

This document lists some use cases that demonstrate the need for a mechanism to share metadata and outlines requirements for both server-to-network (and vice versa) and host-to-network (and vice versa). The document focuses on intra-flow or flows bound to the same user.

--- middle

# Introduction {#intro}

Wireless networks inherently experience large variations in link quality due to several factors.
These include the change in wireless channel conditions, interference between proximate cells and channels or because of the end user movement.
These variations in link quality can be in the order of a millisecond or less {{5G-Lumos}} while congestion control takes several tens of milliseconds (more than one RTT) to estimate data rate.
End-to-end congestion control algorithms are far from optimal when the link quality is highly variable in sub-RTT timeframes and the application demands both low latency and high bandwidth.

It is also not practical to convey sub-RTT link changes using an end-to-end feedback signal.
As a consequence, applications settling for a lower throughput when latency is prioritized or achieving higher throughput at the expense of much higher delays.

With not fully encrypted packets, networks may used some heuristics to build an "implicit signal" derived from the contents of a packet to prioritize or otherwise shape flows.
Implicit signals are not desirable as they leads to ossification of protocols as result of introducing unintended dependencies {{?RFC9419}}.
When packet contents are encrypted, the approach of using implicit signals is no longer viable.

Bandwidth constraints exist most predominantly at the access network (e.g., radio access networks).
Users who are serviced via these networks use hosts which run various application clients; each having different connectivity needs for an optimal user experience.
These needs are not frozen but change over time depending on the application and even depending on how an application is used (e.g., user's preferences).
An explicit signal to the host can help to manage the use of available bandwidth better and better share it with requesting applications.

Other applications like interactive media can demand both high throughput and low latency and, in some cases, carry different media streams (e.g., audio and video) in a single transport connection (e.g., WebRTC {{?RFC8825}}).
There may be preferences that an application may wish to convey, such as a higher priority for audio over video (or the opposite) in congested networks.
With RTP {{?RFC3550}}, the type of media could be examined and used as an implicit signal for determining relative priority. However, {{?RFC9335}} defines a new mechanism that completely encrypts RTP header extensions and Contributing sources (CSRCs). Furthermore, a full encrypted transport (e.g., QUIC {{?RFC9000}}) does not expose any media header information that on-path network elements can use for forwarding.

Also, traffic patterns in some emerging applications can vary significantly during the session. For example, live media or AI-generated content can have significant dynamic variations and potentially aperiodic frames.
Information in unencrypted media packets and headers that wireless networks have used to optimize traffic shaping and scheduling are not exposed in encrypted communications.

~~~~~~~~
                     |
       3GPP/mobile network
+--------------------|----------------------+
|+-- ---+            |   +-----+    +-----+ |
||client+-----------(B)--+radio+----+ UPF | |
|+------+            |   +-----+    +--+--+ |
+--------------------|-----------------|----+
                     |                 |
       Wireless home/ISP network       |
+--------------------|-----------+     |    |          |
|+--- --+    +----+  |  +------+ | +---+--+ | +------+ | +------+
||client+-(B)+WLAN+-(B)-+router+---+router+---+router+---+server|
|+------+    +----+  |  +------+ | +------+ | +------+ | +------+
+--------------------|-----------+          |          |
                     |                      |          |
                     |                      | Transit  |  Content
 User device/Network |    MNO/ISP Network   | Network  |  Network
~~~~~~~~
{: #Figure-e2e title=”E2E Media Transport Overview”}

{{Figure-e2e}} shows where such bandwidth and performance constraints usually exist with a “B” (for Bottleneck) in 3GPP/mobile networks and WLAN/ISP networks.
When a bottleneck exists temporarily, the network has no choice but to discard or delay packets -- which can harm certain flows and, thus, lead to suboptimal perceived experience.
In this document, this is termed 'reactive policy'.

A network attachment represents the communication link between hosts (client) and network (router) over which a connection policy (including QoS) is applied to flows within that network attachment.

A network attachment may be established using control plane signaling between the client and the network (access) router and is out of scope of this document.
Transport flows over a network attachment may consist of multiple streams such as video or audio. {{Figure-conn-flow}} shows a high level view of network attachments, flows, and QoS/policy discussed in {{metadata-req}}.

The requirements in {{metadata-req}} apply to data units like frames within a flow, but not between flows. Specifically, this document does not discuss flows of distinct hosts/users.

~~~~~~~~
+----------+          +-----------------+
|+----+    |          | +-------------+ |
|| A1 |--+ |          | | QoS, Policy | |
|+-+--+A2| |          | +---+-----+---+ |          +------+
|  |+-+--+ |  Network |     |     |     |          |srv-A2|
|  |  |    |attachment|     v     |     |          +--+---+
|  |  |  *************************|**** |             |
|  |  +------------- flow-x2 -----|-------------------+   +------+
|  +------------ flow-x1 ---------|-----------------------+srv-A1|
|        *************************|**** |                 +--+---+
+-Client-1-+          |           |     |                    |
                      |           |     |                    |
+----------+  Network attachment  V     |                    |
|+----+  ****************************** |                    |
|| A1 +------------ flow-x3 ---------------------------------+
|+----+  ****************************** |
+----------+          +-----------------+
  Client-2                  Router
~~~~~~~~
{: #Figure-conn-flow title=”E2E transport flows and connection session”}

{{Figure-conn-flow}} shows "Client-1" and "Client-2" that negotiate  connection policy (e.g., QoS) and other aspects like mobility handling, charging applied to flows in that network attachment.
"Client-1" has "flow-x1" and "flow-x2" over its network attachment while "Client-2" has "flow-x3".
The requirements in this document focuses on on-path collaboration signals that apply to data units such as media frames within flows like "flow-x1/x2/x3" but not between them.

In summary, the rapid variation of wireless link quality and/or bandwidth limitations in networks along with interactive applications that demand low latency and high throughput can lead to suboptimal user experience. {{uc}} outlines use cases to illustrate the issues and the need for additional information per flow to allow the network to optimize its handling.

Some of the complications that are induced by the phenomena discussed above may be eliminated by
   adequate dimensioning and upgrades.  However, such upgrades may not
   be always (immediately) possible or justified. Complementary mitigations are thus needed to soften these complications by introducing some collaboration between endpoints and networks to adjust their behaviors.

{{operational}} provides operational constraints in the network and {{metadata-req}} describes the requirements for on-path media collaboration signals.

# Definitions

The document makes use of the following terms:

Intentional Management:
: Network policy such as (monthly) bandwidth quota or bandwidth limit, or quality (delay and/or jitter)) assurances.

Reactive Management:
: Network reactions to congestion events or protection polices under attacks, with very short to very long durations (e.g., varying wireless and mobile air interface conditions).

Traffic shaping:
: Refers in this document to QoS management at the wireless/access router to delay or discard packets of lower priority to achieve bounded latency and high throughput.

User Plane Function (UPF):
: Refers to a 3GPP element that is located between the mobile infrastructure and the Data Network (DN) as shown in {{Figure-3gpp}}.
: For a definitive description of 3GPP network architectures, the reader should refer to the 3GPP's TR 23.501 {{TR.23.501-3GPP}}.

~~~
  ┌─────┐  ┌─────┐  ┌─────┐    ┌─────┐  ┌─────┐  ┌─────┐
  │NSSF │  │ NEF │  │ NRF │    │ PCF │  │ UDM │  │ AF  │
  └──┬──┘  └──┬──┘  └──┬──┘    └──┬──┘  └──┬──┘  └──┬──┘
Nnssf│    Nnef│    Nnrf│      Npcf│    Nudm│        │Naf
  ───┴────────┴──┬─────┴──┬───────┴───┬────┴────────┴────
            Nausf│    Namf│       Nsmf│
              ┌──┴──┐  ┌──┴──┐     ┌──┴──────┐
              │AUSR │  │ AMF │     │   SMF   │
              └─────┘  └──┬──┘     └──┬──────┘
                       ╱  │           │      ╲
Control Plane      N1 ╱   │N2         │N4     ╲N4
════════════════════════════════════════════════════════════
User Plane          ╱     │           │         ╲
                ┌───┐  ┌──┴──┐  N3 ┌──┴──┐ N9 ┌─────┐ N6  .───.
                │UE ├──┤(R)AN├─────┤ UPF ├────┤ UPF ├────( DN  )
                └───┘  └─────┘     └─────┘    └─────┘     `───'
~~~
{: #Figure-3gpp title="5GS Architecture" artwork-align="center"}

# Use Cases {#uc}

## Media Streaming {#uc-streaming}

Streaming video contains the occasional key frame ("I-frame") containing a full video frame.
These frames are necessary to rebuild receiver state after loss of delta frames.
The key frames are therefore more critical to deliver to the receiver than delta frames.

Streaming video also contains audio frames which can be encoded
separately and, thus, can be signaled separately (requirement:
REQ-MEDIA-KEYFRAME).

Audio is more critical than video for many applications, but its
importance (relative to other packets in the flow) is still an
application decision (requirements: REQ-MEDIA-AV-SEPARATE,
REQ-MEDIA-CLIENT-DECIDES). For example, the ability of the receiver to change the priority by communicating to the network -- without cooperation of the sender -- gives a hearing-impaired user the ability to adjust video above audio.

Especially with media over QUIC, the server (or relay) sends the same
stream to many receivers, including the same metadata.  Thus, when a
client needs different prioritization (e.g., video over audio or the other way around), this
is only achievable by signaling that priority inversion from the
client to the ISP router (requirement: REQ-CLIENT-DECIDES).

Examples: live broadcast, on-demand video streaming

Use cases:

  1. In loss-prone networks or during reactive policy events, retransmissions cause long delays. All packets being treated the same can have challenges in efficiently handling/forwarding data. Today, there is no way to identify packets which are less important and/or loss-tolerant to prioritize packets in challenging networks and/or during reactive events.

  2. Some media frames may be able to tolerate more delay over the wire than others (e.g., live media frames require very low latency while a background image for augmented reality may be delivered with more delay tolerance).  Even when the media payload is not encrypted, the network has no means to distinguish these different requirements.

Requirements:

  REQ-MEDIA-KEYFRAME: Video contains partial frames and full frames, which need to be distinguished so that
full frames can be indicated to the network.

  REQ-MEDIA-AV-SEPARATE:  Audio can be prioritized differently than video.


## Interactive Media {#uc-interactive}

Interactive media includes content that a user can actively engage with and results in input and response actions that can be highly delay-sensitive.

Examples: VoIP (Peer-to-Peer (P2P), group conferencing), gaming, eXtended Reality (XR).

Use cases:

  1. A mobile/roaming user prioritizes audio over video during a VoIP call to have a seamless meeting experience.

Requirements:

  REQ-PACKET-NATURE: The receiver indicates that a flow is interactive and requests that the network honors the incoming flow's
per-packet signals, which prevents denial of service of mis-marked incoming flows.

## User Preferences {#uc-preferences}

A game or VoIP application may want to signal different metadata for the same type of packet in each direction.
For example, for a game, video in the server-to-client direction might be more important than audio, whereas input devices (e.g., keystrokes) might be more important than audio.  Each user can have varied preferences for the same type of data originating from the server.

Determination of such preferences is outside of the scope of this document.

Requirements:

  REQ-CLIENT-DECIDES: The receiving client determines importance of packets it receives, as the client may have changing needs over time.

Use cases:

  1. Dynamic changes to priority based on user activity is not possible today. For example, audio packets having the same priority when a user mutes the audio locally, or change in priority during times of emergency where video streaming applications share the same priority as SOS signals.

  2. A user prioritizing video over audio while playing an interactive game.

  3. User's foreground application should receive priority over background tasks. For example, a user is typing in a document while playing a media in the background within the same session.

## Mixed Traffic {#mixed-traffic}

Mixed traffic can contain multiple types of data flowing through the same 5-tuple connection. They may include digital models of the real world, multimedia content, virtualized desktop/apps, and interactive engagement.

In cases where the wireless network has to drop or delay processing, all packets of the media frame or stream are treated in the same manner.

Requirements:

  REQ-PACKET-RELIABILITY: Indicate if a packet is treated as reliable or unreliable by the application.

  REQ-PACKET-NATURE: Indicate if a packet belongs to bulk traffic or interactive traffic.

Examples: Virtual Apps and Desktops.

Use cases:
  1. Document being printed/saved while being edited. The interactive traffic has higher priority over bulk traffic).

  2. File download while intearacting with the webpage. With QUIC, this could occur with the same webserver. Interactive activity performed with the webpage higher priority over file download.

  3. In graphics remoting, Critical Glyphs (Reliable) has more priority over Smoothing Glyphs (unreliable) during reactive events.

## Honoring of Metadata for Servers Behind a Gateway

In enterprise networks and remote desktop use case, a server can host multiple connections with varying type of traffic. These servers are often exposed to the Internet through some sort of a gateway-proxy and the signaling (like DSCP bits) from these servers are often ignored by the access/transit networks.

Requirement: REQ-CLIENT-DECIDES as defined previously.

# Operational Considerations {#operational}

Traffic policing and shaping are enforced in ingress/egress network points for various reasons (protect the network against attacks, ensure conformance with a trafic profile, etc.). Out-of-profile trafic may be discarded or assigned another class (e.g., using Lower Effort Per-Domain Behavior (LE PDB) {{?RFC3662}}) a bandwidth limit among others. The exact behavior is policy-based and deployment-specific.

The entire set of operations to manage traffic is beyond the scope of this document.
This section focuses on operational constraints that impact  server – network, and host – network modes of sending metadata.

## Policy Enforcement {#policy}

Some metadata requires the network to share some hints with a host to adjust its behavior for some specific flows.
However, that metadata may have a dependency on the service offering that is subscribed by a user.

Let us consider the example of a bitrate for an optimized video delivery. *Such bitrate may not be computed system-wide* given that flows from users with distinct service offerings (and connectivity SLOs) may be serviced by the same network nodes. Instead, the network needs to dynamically adjust the bitrate based on each user's service package and connectivity SLOs to ensure optimal delivery for all users (REQ-METADATA-ACCURACY).

Requirement:  REQ-METADATA-ACCURACY.

## Redundant Functions and Classification Complications {#classification}

If distinct channels are used to share the metadata between a host and a network, a network that engages in the collaborative signaling approach will require sophisticated features to classify flows and decide which channel is used to share metadata so that it can consume that information.

Likewise, the network will require to implement redundant functions; for each signaling interface.

*As such, application- and protocol-specific signaling channels are suboptimal.* (REQ-SINGLE-CHANNEL)

Requirement:  REQ-SINGLE-CHANNEL is preferred.

## Metadata Scope {#metadata-scope}

An operational challenge for sharing resource-quota like metadata (e.g., maximum bitrate) is that the network is generally not entitled to allocate quota per-application, per-flow, per-stream, etc. that delivered as part of an Internet connectivity service.  However, the network has a visibility about the overall network attachment (e.g. inbound/outbound bandwidth discussed in {{?I-D.ietf-opsawg-teas-attachment-circuit}}).

As such, hints about resource-like metadata is bound by default to the overall network attachment, not specific to a given application or flow.

Most importantly, the metadata can be used by the network to prioritize traffic within a single 5-tuple connection and metadata cannot be leveraged for prioritization between different flows.

It is out of the scope of this document to discuss setups (e.g., 3GPP PDU Sessions) where network attachments with Guaranteed Bit Rate (GBR) for specific flows is provided.

Requirement:
  REQ-SCOPED-METADATA: Means to characterize the scope of a shared metadata for the sake of better interoperability should be supported.

## Application Interference {#app-interference}

Applications that have access to a resource-quota information may adopt an aggressive behavior (compared to those that don't have access) if they assumed that a resource-quota like metadata is for the application, not for the host that runs the applications.

This is challenging for home networks where multiple hosts may be running behind the same CPE, with each of them running a video application. The same challenge may apply when tethering is enabled.

Requirement:
  REQ-SIGNAL-EXPOSURE-FAIRNESS: Means to expose the signal independent of the application should be considered. An example of such exposure is OS APIs.

## Privacy Considerations {#privacy}

Encrypted media payloads along with temporary IPv6 addresses between a server and user (client) provide a measure of privacy for the content and the identity of the user.
It should, however, be noted that media flows (e.g., encrypted video payloads in SRTP) exhibit a pattern of bursts and intervals that amounts to a signature and is vulnerable to frequency analysis.
To avoid this kind of frequency analysis, media sent by the server would need to be scheduled or multiplexed differently to each user/recipient.
This may be possible in transports like QUIC which allows flexibility in scheduling each stream.
Transports like QUIC also fully encrypt the entire stream and, therefore, no media headers are observable on-path either.
The security aspects of the media payload / transport are not in the scope of this document and is described here only to provide context for metadata privacy.

Some metadata (e.g., the size of a burst of packets, sequence number, and timestamp) can be readily observed or inferred by entities
along the network path. However, it is essential to recognize that while sequence numbers and timestamps are typically visible in
the clear-text headers of protocols (e.g., TCP, RTP, or SRTP) they are not directly observable in encrypted protocols such as QUIC. All metadata sent
from the server to the network, including these elements and others, are vulnerable to modification while in transit. Only an
on-path attacker can modify on-path metadata. Such an attacker could engage in other malicious activities, like corrupting the checksum or
completely dropping the packet. For instance, an active attacker could alter the metadata to mislabel packets containing video key-frames
as unimportant, but such changes are detectable by the receiver. The privacy implications of revealing metadata to network elements need to
be thoroughly analyzed. This analysis should ensure that any exposure of metadata does not compromise user privacy or allow unauthorized
entities to infer sensitive information about the data being transmitted.

Requirements:

REQ-PRIVACY-ADDITIONAL:
: An on-path observer obtains no additional information about the IP packet.

REQ-SIGNALING-AVOIDANCE:
: Leveraging previous experience {{?RFC9049}}, the folliwing is not required to make use of the collaborative signaling:

   * Reveal the application identity.
   * Expose the application cause (or 'reason') to signal metadata.
   * Reveal server identity.
   * Inspect client-to-server encrypted payload by network elements.

## Scalability {#scalability}

There may be a large number of media flows handled by the server and wireless/access router.

Per flow information (state) at a wireless router for optimizing the flow can negate the advantages offered as the number of flows handled increases.

The metadata and other state information that a router has to maintain for each additional media flow it handles should be kept to a minimum or eliminated altogether.

Requirement:  REQ-ISP-SCALE: The metadata other state information that a wireless router has to maintain for each additional media flow it handles should be very low or none.

## Session Continuity {#continuity}

The general trend in wireless networks is to deploy the wireless router closer to the user.
This can help with low link latency but can result in more frequent handovers as the user moves between routers.
The frequency of handovers increases when a user moves faster or when the media session lasts longer.

During handovers, there should be minimal delay incurred during handover in configuring/setting up the metadata of a media session in progress.

Requirement: REQ-CONTINUITY: Handover from one radio or router to another should continue to provide same service level.


## Abuse and Constraints {#abuse}

It is important that not every flow be prioritized; otherwise, the network devolves into the best-effort network that existed prior to metadata signaling. It is a requirement that mechanisms exist to prevent this occurrence.

Such a mechanism might be simple, for example, a cellular network might allow one flow from a subscriber to declare itself as important; other flows with that subscriber are denied attempts to prioritize themselves. However, the network cannot identify whether the prioritized flow is legitimate or malicious.

Requirements:

  REQ-SIGNAL-VALIDATION: The network/OS needs to ensure that the user/client signaling of priority (if any) does not associate the same priority level with all traffic types within the same flow, thereby avoiding prioritizing of all the streams/traffic the same way.

  REQ-CLIENT-VALIDATION: The network needs to ensure the signal is coming from the same user/client that is part of the 5-tuple flow. This is to ensure no other application influences the priority of another application's flow.

# On-path Metadata Requirements {#metadata-req}

There are various approaches for collaborative signaling between the server/host and network including out-of-band signaling, client-centric metadata sharing and proxied connections.
The requirements here focus on proxied metadata connections on path with the data traffic.

The path signals below should follow the principles of intentional distribution, protection of information, minimization and limiting impact as described in {{?RFC9419}} and {{?RFC8558}}.
Leveraging previous experience ({{?RFC9049}}), the metadata signals does not need application identity, application cause (or 'reason'), server identity or the inspection of client(host)-to-server encrypted payload.

The metadata connections may be between server and network (in either direction) or between host and network (in either direction).

Some use cases benefit from server – network metadata exchanges ({{server-network}}) and others need client involvement ({{host-network}}).

For the requirements that follow, the assumption is that the client agrees to the exchange of metadata between the server and network, or between the client and network.

REQ-PACKET-SELF: Packet importance is indicated by the packet itself, which may need to be decrypted or de-obfuscated.


## Host-Network Metadata {#host-network}

### Priority between Flows (Inter-flow) {#interflow-priority}

Certain flows being received by a host (or by an application on a host) are less or more important than other flows of *the same host*.
For example, a host downloading a software update is generally considered less important than another host doing interactive audio/video or gaming.
By signaling the relative importance of flows to a network element, the network element can (de-)prioritize those flows to best accommodate the needs of the various applications on a same host.

Without a signaling in place between a receiving host and its network, remote peers are able to mark packets that interfere with the desires of the receiving host -- making their flows more important than what the receiving host considers more important.
This eventually causes all flows to be marked as important, or -- more likely -- such priority markings to be ignored.

However, prioritizing between flows presents challenges because the host can have both malicious and legitimate applications, and the remote peers can also be malicious and benign.

There is no requirement associated with this use-case.


### Priority within a Flow (Intra-Flow) {#intra-flow-priority}

Interactive Audio/Video has long been using {{?RFC3550}} which runs over UDP.  As described in Section 2.3.7.2 of {{?RFC7478}}, there is value in differentiating between voice, video and data.
Today's video streaming is exclusively over TCP but will migrate to QUIC and eventually is likely to support unreliable transport ({{?RFC9221}}, {{?I-D.ietf-moq-transport}}).  With unreliable transport of video in RTP or QUIC, it is beneficial to differentiate the important video keyframes from other video frames.

Other applications, as mentioned in {{uc}}, such as gaming and remote desktop also benefit from differentiating their packets to the network.

Many of these flows do not originate from a content provider's network -- rather, they originate from a peer (e.g., VoIP, interactive video, peer-to-peer gaming, Remote Desktop, and CDN). Thus, the flows originate from an IP address that is not known before connection establishment, so there needs to be a way for the client to authorize the network elements to receive and hopefully to honor the metadata of those packets from a remote peer.

Without a signaling in place between a receiving host and its network, remote peers are able to mark every packet of a flow as important, causing much the same problem as the previous use-case.
Eventually, when all packets of every flow are marked as important, there is no differentiation between packets within a flow, rendering the network unable to improve reactive policy decisions.

Requirements: The requirements vary based on use case and user preference. The requirements listed in {{uc}} are applicable here.

## Network-Host Metadata {#network-host}

### Assisted Offload

   There are cases (crisis) where "normal" network resources cannot be
   used at maximum and, thus, a network would seek to reduce or offload
   some of the traffic during these events -- often called 'reactive
   traffic policy'.  An example of such use case is cellular networks
   that are overly used (and radio resources exhausted) such as a large
   collection of people (e.g., parade, sporting event), or such as a
   partial radio network outage (e.g., tower power outage).  During such
   a condition, an alternative network attachment may be available to
   the host (e.g., Wi-Fi).

   Network-to-host signals are useful to put in place adequate traffic
   distribution policies on the host (e.g., prefer the use of alternate paths,
   offload a network) (REQ-NETWORK-SEEKS-LOAD-DOWN).

## Server-Network Metadata {#server-network}

### Identification of Media Frames and Streams {#mdu-stream-id}

Feedback provided by ECN/L4S to the server (UDP sender) is not fast enough to adjust the sending rate when available wireless capacity changes significantly in very short periods of time (~ 1 millisecond).
Differentiating using multiple DSCP codes does not provide the resolution required to classify media frames or streams and adapt to changes in coding due to dynamic content or resulting from network conditions.

Relative priority and tolerance to delay of media frames or streams can be used to optimize traffic shaping at the wireless router.
The application can provide information to detect the start, end and set of packets that belong to a media frame.
Alternatively, the application may provide information to identify one stream of the flow from another.
The application provides information to identify either media frames or streams in a flow but not both.

In cases where the wireless network has to drop or delay processing, all packets of the media frame or stream are treated in the same manner.

Requirements:

REQ-FRAME-START: Indicate packet containing start of media frame.

REQ-FRAME-MIDDLE: Indicate packet containing middle(s) of media frame.

REQ-FRAME-END: Indicate packet containing end of media frame.

### Identification of Traffic Type without Disclosure of the Application {#TrafficType}

Different nature/types of traffic can be part of the same 5-tuple flow. This could be reliable/loss-tolerant {{?RFC9221}}, bulk/interactive traffic. The type of traffic can be used to prioritize/buffer packets as needed and deprioritize/discard appropriate packets during reactive events, thereby optimizing performance. The application may provide information to identify the type of traffic in per-packet metadata.

Requirements: REQ-PACKET-RELIABILITY, REQ-PACKET-NATURE as defined in {{mixed-traffic}}.

### Relative Priority {#relative-priority}

Relative importance of a media frame provides the priority level of one media frame over another media frame within a stream.
The application server determines the importance based on the media encoded in the media frame (e.g., a base layer video I-frame has higher priority than an enhanced layer P-frame).
Importance may be used to determine drop priority of a media frame in cases of extreme congestion in the wireless network.

Relative importance of a media stream  is the priority level of one media stream over another stream in the flow (with the same IP 5-tuple).
As with media frames, importance may be used to determine drop priority in cases of extreme congestion in the wireless network.

There is no requirement associated with this use-case.

### Tolerance to Delay {#delay}

Some media frames may be able to tolerate more delay over the wire than others (e.g., live media frames require very low latency while a background image for augmented reality may be delivered with more delay tolerance).
Similarly, some media streams can tolerate more delay over the wire than others (e.g., a stream carrying a background image may tolerate more delay).
ams may be able to tolerate more delay over the wire than others (e.g., a stream carrying a background image for augmented reality may be delivered with more delay tolerance).
Even when the media payload is not encrypted, the network has no means to distinguish these different requirements.

If the application can indicate that a media frame or stream can tolerate high delay the wireless router can opt to delay packets rather than drop during transient congestion periods.

REQ-DELAY-TOLERANCE: REQ-PACKET-RELIABILITY, REQ-PACKET-NATURE as defined in {{mixed-traffic}}.

### Burst Indication {#burst}

Media flows can have large and unexpected variations in packet bursts due to dynamic changes in content, server estimation of network conditions and pacing behavior.
Encoding of live video, and multimodal media can only increase the burst size that a server has to contend with sending out in a relative smoothed out manner.
The burst size is observable on the wire, but can only be determined by the end of the burst of packets.
Wireless networks on the other hand cannot reserve resources for the maximum burst size allowed as that will likely lead to poor utilization of radio resources or tail drops.

The server may provide burst size at the beginning of the burst to allow the scheduler to reserve sufficient resources (and avoid having too few resources that may lead to a tail drop).
The server may also signal end of burst that provides information for the radio to go into sleep mode (Connected Mode Discontinuous Reception, C-DRX) if there is no paging message.

REQ-BURST-INDICATOR: Client indicates this flow's maximum burst to
ISP, and ISP agrees it can handle that burst size.  (but what does ISP
router do with the burst? Needs to be described above!)

# Non-Requirements {#non-req}

Application feedback with measurements of packets lost and delay incurred may affect the sending rate and other behavior of the application.
The requirements and specification to mitigate these aspects are not in the scope of this document.


# IANA Considerations {#iana}
None.

# Security Considerations {#sec-cons}

Security aspects for the metadata are discussed in {{privacy}}.
The principles outlined in {{?RFC8558}}, {{?RFC9049}} and {{?RFC9419}} contain security considerations and are referenced in {{metadata-req}}.

This document has no other security considerations.

# Acknowledgments
{:numbered="false"}

This document is a merge of {{?I-D.rwbr-tsvwg-signaling-use-cases}} and {{?I-D.kaippallimalil-tsvwg-media-hdr-wireless}}.

Acknowledgments from {{?I-D.kaippallimalil-tsvwg-media-hdr-wireless}}:
: Xavier De Foy and the authors of this draft have discussed the similarities and differences of this draft with the MoQ draft for carrying media metadata.
: The authors wish to thank Mike Heard, Sebastian Moeller and Tom Herbert for discussions on metadata fields, fragmentation and various transport aspects.
: The authors appreciate input from Marcus Ilhar and Magnus Westerlund on the need to address privacy in general and Dan Druta to consider a common transport across various host to network signaling when possible.
Ruediger Geib suggested that limiting the amount of state information that a wireless router has to keep for a flow should be minimized.
: Ingemar Johansson's suggestions on fast fading (which L4S handles) and dramatic drops in wireless accesses have been helpful to identify the issues.
Thanks to Hang Shi for the review and comments on host-to-network signaling.

--- back
