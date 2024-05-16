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
author:
 -
    ins: J. Kaippallimalil
    name: John Kaippallimalil
    organization: Futurewei
    email: john.kaippallimalil@futurewei.com
 -
    ins: S. Gundavelli
    name: Sri Gundavelli
    organization: Cisco
    email: sgundave@cisco.com
 -
    ins: S. Dawkins
    name: Spencer Dawkins
    organization: Tencent America LLC
    email: spencerdawkins.ietf@gmail.com

normative:

informative:

  TR.23.501-3GPP:
    title:  "3rd Generation Partnership Project; Technical Specification Group Servies and System Aspects; System architecture for the 5G System (5GS); Stage 2 (Release 18)"
    date: March 2023

  TR.23.700-60-3GPP:
    title:  "Study on XR (Extended Reality) and media services (Release 18)"
    date: August 2022

--- abstract

Wireless networks like cellular or Wi-Fi experience significant but transient variations in link capacity, have bandwidth or other policy constraints that affect user experience.
Server-to-network and host-to-network signaling can improve the user experience by informing the network about the relative importance of media frames or streams without having to disclose the content of the packets.
The differentiated service may be provided a the network (e.g., packet discard preference), the sender (e.g., adaptive transmission) or through cooperation of server / host and the network.

This document lists some use cases that demonstrate the need for a mechanism to share metadata and outlines requirements for both server-to-network (and vice versa) and host-to-network (and vice versa).

--- middle

# Introduction {#intro}

Wireless networks inherently experience large variations in link capacity due to several factors.
These include the change in wireless channel conditions, interference between proximate cells and channels or because of the end user movement.
These variations in link capacity can be in the order of a millisecond or less.
Media frames may experience high sojourn times in the wireless network or the flow may experience lower throughput, neither of which is desirable.
The result is applications settling for a lower throughput when latency is prioritized or achieving higher throughput at the expense of much higher delays.
With unencrypted packets, networks used Deep Packet Inspection (DPI) to identify an “implicit signal” derived from the contents of a packet to prioritize or otherwise shape flows.
When packet contents are encrypted, this approach is no longer viable.

Bandwidth constraints exist most predominantly at the access network (e.g., radio access networks).
Users who are serviced via these networks use various hosts which run various applications; each having different connectivity needs for an optimal user experience.
These needs are not frozen but change over time depending on the application and even depending on how an application is used (e.g., user's preferences).
An explicit signal to the user can help to manage the use of available bandwidth better.

Applications like interactive media on the other hand can demand both high throughput and low latency, and in some cases carry different media streams in a single transport connection (e.g., WebRTC).
There may be preferences that an application may wish to convey, such as a higher priority for audio over video (or the opposite) in congested networks.
With RTP, the type of media could be examined and used as an implicit signal for relative priority.
However, a full encrypted transport such as QUIC does not expose any media header information that networks on path can use.

Traffic patterns in some emerging applications can vary significantly during the session.
For example, live media or AI generated content can have significant dynamic variations and potentially aperiodic frames.
Information in unencrypted media packets and headers that wireless networks have used to optimize traffic shaping and scheduling are not exposed in encrypted communications.

~~~~~~~~
                    |
      3GPP/mobile network
+-------------------|----------------------+
|+----+             |   +-----+    +-----+ |
||host+------------(B)--+radio+----+ UPF | |
|+----+             |   +-----+    +--+--+ |
+-------------------|-----------------|----+
                    |                 |
      Wireless home/ISP network       |
+-------------------|-----------+     |    |          |
|+----+    +------+ |  +------+ | +---+--+ | +------+ | +------+
||host+-(B)+ WLAN +(B)-+router+---+router+---+router+---+server|
|+----+    +------+ |  +------+ | +------+ | +------+ | +------+
+-------------------|-----------+          |          |
                    |                      |          |
                    |                      | Transit  |  Content
 User device/Network|    MNO/ISP Network   | Network  |  Network

~~~~~~~~
{: #Figure-e2e title=”E2E Media transport overview”}

{{Figure-e2e}} shows where such bandwidth and performance constraints usually exist with a “B” (for Bottleneck) in 3GPP/mobile networks and WLAN/ISP networks.
When a bottleneck exists temporarily, the network has no choice but to discard or delay packets -- which can harm certain flows and thus lead to suboptimal perceived experience.  In this document, this is termed 'reactive policy'.

The rapid variation of wireless link capacity and/or bandwidth limitations in networks along with interactive applications that demand low latency and high throughput can lead to poor user experience.
{{uc}} outlines use cases to illustrate the issues.
{{operational}} provides operational constraints in the network and {{metadata-req}} describes the requirements for on-path media collaboration signals.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Definitions

ISP:
: Internet Service Provider

UPF:
: User Plane Function (3GPP); first router in the 5G network that provides session/connectivity service to a host (user).

WLAN:
: Wireless Local Area Network

Intentional Management:
: network policy such as (monthly) bandwidth quota or bandwidth limit, or quality (delay and/or jitter)) assurances.

Reactive Management:
: network reactions to congestion events, with very short to very long durations (e.g., varying wireless and mobile air interface conditions).

Traffic shaping in this document refers to QoS management at the wireless/access router to delay or discard packets (or groups of packets) of lower priority to achieve bounded latency and high throughput.


# Use Cases {#uc}

## Media Streaming {#uc-streaming}

Streaming video contains the occasional key frame ("I-frame") containing a full video frame.
These are necessary to rebuild receiver state after loss of delta frames.
The key frames are therefore more critical to deliver to the receiver than delta frames.

Streaming video also contains audio frames which can be encoded separately and thus can be signaled separately.
Audio is more critical than video for almost all applications, but its importance (relative to other packets in the flow) is still an application decision.

Examples: Super bowl, On-Demand Streaming

## Interactive Media {#uc-interactive}

Interactive media includes content that a user can actively engage with and results in input and response actions that can be highly delay sensitive.
They may include digital models of the real world, multimedia content and interactive engagement.

Examples: VoIP (peer-to-peer (P2P), group conferencing), gaming, eXtended Reality (XR).

## User Preferences {#uc-preferences}

A game or VoIP application may want to signal different metadata for the same type of packet in each direction.
For example, for a game, video in the server-to-client direction might be more important than audio, whereas input devices (e.g., keystrokes) might be more important than audio.  Each user can have varied preferences for the same type of data originating from the server.
Determination of such preferences is outside of the scope of this document.

## Mixed Traffic {#uc-mixed}

Desktop virtualization is an example where a server can host multiple connections with varying types of traffic to a remote desktop.
In some cases, the signaling (like DSCP) from the servers are ignored by the ISP.

In a remote desktop, a streaming video application may be playing in the background while the user is editing a document.
The user’s keystrokes and those glyphs may need priority over the video in such cases.

Determination of such preferences is up to the user or application(s) and out of scope of this document.
However, it illustrates the need for explicit signaling of preferences to the network.

# Operational Considerations {#operational}

Traffic policing and shaping are enforced in ingress/egress network points for various reasons (protect the network against attacks, ensure conformance with a trafic profile, etc.). Out-of-profile trafic may be discarded, or assigned another class (e.g., using Lower Effort Per-Domain Behavior (LE PDB) {{?RFC3662}}), or delayed to meet a bandwidth limit among others. The exact behavior is policy-based and deployment-specific.

The entire set of operations to manage traffic is beyond the scope of this document.
This section focuses on operational constraints that impact  server – network, and host – network modes of sending metadata.


## Policy Enforcement {#policy}

Some metadata requires the network to share some hints with a host to adjust its behavior for some specific flows.
However, that metadata may have a dependency on the service offering that is subscribed by a user.

Let us consider the example of a bitrate for an optimized video delivery. *Such bitrate may not be computed system-wide* given that flows from users with distinct service offerings (and connectivity SLOs) may be serviced by the same network nodes.

## Redundant Functions and Classification Complications {#classification}

If distinct channels are used to share the metadata between a host and a network, a network that engages in the collaborative signaling approach will require sophisticated features to classify flows and decide which channel is used to share metadata so that it can consume that information.

Likewise, the network will require to implement redundant functions; for each signaling interface.

*As such, application- and protocol-specific signaling channels are suboptimal.*


## Metadata Scope {#metadata-scope}

An operational challenge for sharing resource-quota like metadata (e.g., maximum bitrate) is that the network is generally not entitled to allocate quota per-application, per-flow, per-stream, etc. that delivered as part of an Internet connectivity service.  However, the network has a visibility about the overall network attachment (e.g. inbound/outbound bandwidth discussed in {{!I-D.ietf-opsawg-teas-attachment-circuit}}).

As such, hints about resource-like metadata is bound by default to the overall network attachment, not specific to a given application or flow.

It is out of the scope of this document to discuss setups (e.g., 3GPP PDU Sessions) where network attachments with Guaranteed Bit Rate (GBR) for specific flows is provided.

## Application Interference {#app-interference}

Applications that have access to a resource-quota information may adopt an aggressive behavior (compared to those that don't have access) if they assumed that a resource-quota like metadata is for the application, not for the host that runs the applications.

This is challenging for home networks where multiple hosts may be running behind the same CPE, with each of them running a video application. The same challenge may apply when tethering is enabled.

## Privacy Considerations {#privacy}

Encrypted media payloads along with temporary IP addresses between a server and user (client) provide a measure of privacy for the content and the identity of the user.
It should however be noted that media flows (e.g., encrypted video payloads in SRTP) exhibit a pattern of bursts and intervals that amounts to a signature and is vulnerable to frequency analysis.
To avoid this kind of frequency analysis, media sent by the server would need to be scheduled or multiplexed differently to each user/recipient.
This may be possible in transports like QUIC which allows flexibility in scheduling each stream.
Transports like QUIC also fully encrypt the entire stream and therefore no media headers are observable on path either.
The security aspects of the media payload/ transport are not in the scope of these requirements and is described here only to provide context for metadata privacy.
Privacy considerations for the metadata itself should ensure that no additional information about either the content or the user of the content is revealed.

Some of the metadata like the size of a burst of packets, sequence number and timestamp are information that can be plainly observed or inferred by an entity on path.
These and all other metadata sent from server to the wireless router are vulnerable to modification on path.
All metadata should therefore have secure integrity protection (e.g., a secure message digest) to detect any modification or tampering on path.


## Key Establishment {#key-establishment}

Various proposals have suggested establishing a key to validate per-packet metadata or to decrypt per-packet metadata.
However, most proposals have not specified how this key would be established.
A signaling protocol from the receiving host to its ISP could establish such a key.
The host can then convey the key to the sending host to use to integrity protect or encrypt the per-packet metadata.

Note: The CPU overhead of validating or decrypting such per-packet metadata needs to be carefully considered (and further assessed via experiments) by the signaling protocol proposing such keying.
Also, the required operational setup should be documented.


## Scalability {#scalability}

There may be a large number of media flows handled by the server and wireless/access router.

Per flow information (state) at a wireless router for optimizing the flow can negate the advantages offered as the number of flows handled increase.
The metadata other state information that a wireless router has to maintain for each additional media flow it handles should be very low or none.


## Session Continuity {#continuity}

The general trend in wireless networks is to distribute the wireless router closer to the user.
This can help with low link latency but can result in more handovers from one router to another as the user moves.
The number of handovers also increase as a user moves faster or the media session lasts longer.

There should no additional delay incurred during handover in configuring/setting up the metadata of a media session in progress.


## Abuse and Constraints {#abuse}

It is important that not every flow be prioritized; otherwise, the network devolves into the best-effort network that existed prior to metadata signaling.
It is a requirement that mechanisms exist to prevent this occurrence.

Such a mechanism might be simple, for example, a cellular network might allow one flow from a subscriber to declare itself as important; other flows with that subscriber are denied attempts to prioritize themselves.
The mechanism might be more complex where authentication and authorization is performed by an enterprise network which, itself, decides which flows are important based on its policy and only the enterprise network communicates flow priorities to the ISP network.  The enterprise might prioritize certain users (e.g., IT staff), certain equipment (audio/video equipment in a conference room), or whatever its policies it might want.


# On-path Metadata Requirements {#metadata-req}

There are various approaches for collaborative signaling between the server/host and network including out-of-band signaling, client-centric metadata sharing and proxied connections.
The requirements here focus on proxied metadata connections on path with the data traffic.

The path signals below should follow the principles of intentional distribution, protection of information, minimization and limiting impact as described in {{?RFC9419}} and {{?RFC8558}}.
Leveraging previous experience ({{?RFC9049}}), the metadata signals does not need application identity, application cause (or 'reason'), server identity or the inspection of client(host)-to-server encrypted payload.

The metadata connections may be between server and network (in either direction) or between host and network (in either direction).

Some use cases benefit from server – network metadata exchanges ({{server-network}}) and others need client involvement ({{host-network}})


## Server-Network Metadata {#server-network}

In many scenarios that require low latency media delivery, the server and wireless router have established relationships and contractual agreements to optimize the delivery of media flows.
For example, the Content Provider (CP) and Mobile Network Operator (MNO) may have a limited domain ({{?RFC8799}}) that manages trust and configures policies related to the delivery of content.
The server and wireless router (3GPP UPF) are located in networks operated by CP and MNO respectively.
The transit (IP) network in between is either a trusted network that is managed and operated by the CP/MNO, or the CP/MNO use security gateways at the boundaries of their network to encrypt all traffic flowing between them.
Thus, the requirements in this section are not expected to satisfy sending metadata between arbitrary servers and wireless routers located across a wide area network.


### Identification of Media Frames {#frame-id}

Feedback provided by ECN/L4S to the server (UDP sender) is not fast enough to adjust the sending rate when available wireless capacity changes significantly in very short periods of time (~ 1 millisecond).
Differentiating using multiple DSCP codes does not provide the resolution required to classify media frames and adapt to changes in coding due to dynamic content or resulting from network conditions.

Relative priority of media frames, tolerance to delay, identification of media frame boundaries are provided by the application to optimize traffic shaping at the wireless router.
Alternatively, an application may prefer to provide only the information about streams and their relative priority (see {{stream-id}}).
In such cases it does not provide any information to classify media frames.

A wireless router should treat all packets of an media frame in the same manner for optimal application performance.
Random, or tail drops that span media frame boundaries may result in the decoder being unable to decode many more media frames.

The application should provide information that allows the wireless router to detect the start, end and set of packets of a media frame.
In cases where the wireless network has to drop or delay processing, all packets of the media frame are treated in the same manner.


### Relative Priority of Media Frames {#frame-priority}

Relative importance of a packet (or media frame) provides the priority level of one media frame over another media frame in a stream.
The application server determines the importance based on the media encoded in the media frame (e.g., a base layer video I-frame has higher priority than an enhanced layer P-frame).
Importance may be used to determine drop priority in cases of extreme congestion in the wireless network.


### Tolerance to Delay of Media Frames {#frame-delay}

Some media frames may be able to tolerate more delay over the wire than others (e.g., live media frames require very low latency while a background image for augmented reality may be delivered with more delay tolerance).
Even when the media payload is not encrypted, the network has no means to distinguish these different requirements.

If the application can indicate that a media frame can tolerate high delay the wireless router can opt to delay packets rather than drop during transient congestion periods.
The application should also be able to convey that in some cases it has no specific information about relative delays, or expects the same delay tolerance for all media frames.


### Identification of Media Streams {#stream-id}

In some deployments, multiple media streams with different priorities are sent in a single transport connection (e.g., WebRTC {{?RFC8854}}, QUIC transports for multimodal media, audio, video, control, haptics).
In such cases, an application may want to prioritize one stream over another in the event of extreme congestion (e.g., audio stream prioritized over video stream).

The application may prefer to provide only the information about streams and their relative priority, and in such cases it does not provide any information to classify media frames (see {{frame-id}}).
In this case, the application conveys to the wireless network the relative priority of media streams in a single transport connection.

Multiple streams of media are sometimes delivered over a single transport connection where the payload and stream headers are encrypted (e.g., QUIC).
A wireless router cannot identify the streams by inspecting the header fields (e.g., connection identifier in QUIC).

The application should provide a means to identify one encrypted media stream from another to allow the wireless router to provide differentiated QoS treatment for each stream.


### Relative Priority of Media Streams {#stream-priority}

Relative importance of a media stream  is the priority level of one media stream over another stream in the transport connection.
The application server determines the importance based on the media encoded in the stream.
Importance may be used to determine drop priority in cases of extreme congestion in the wireless network.


### Tolerance to Delay of Media Streams {#stream-delay}

Some media streams may be able to tolerate more delay over the wire than others (e.g., a stream carrying a background image for augmented reality may be delivered with more delay tolerance).
Even when the media payload is not encrypted, the network has no means to distinguish these different requirements.

If the application can indicate that a stream can tolerate high delay the wireless router can opt to delay packets rather than drop during transient congestion periods.
The application should also be able to convey that in some cases it has no specific information about relative delays, or expects the same delay tolerance for all media streams.


### Burst Indication {#burst}

Media flows can have large and unexpected variations in packet bursts due to dynamic changes in content, server estimation of network conditions and pacing behavior.
Encoding of live video, and multimodal media can only increase the burst size that a server has to contend with sending out in a relative smoothed out manner.
The burst size is observable on the wire, but can only be determined by the end of the burst of packets.
Wireless networks on the other hand cannot reserve resources for the maximum burst size allowed as that will likely lead to poor utilization of radio resources or tail drops.


The server may also signal end of burst that provides information for the radio to go into sleep mode (Connected Mode Discontinuous Reception, C-DRX) if there is no paging message.

The server should provide burst size at the beginning of the burst to allow the scheduler to reserve sufficient resources (and avoid having too few resources that may lead to a tail drop).
Application servers that are not able to reliably estimate the burst size at the beginning of a burst should not provide any information about the burst.


### Packet Loss Detection {#loss}

The wireless router should be aware of any loss of packets belonging to a media frame.
For example, the loss of a packet that is a start or end of a media frame can cause confusion in estimating media frame boundaries.
However, the wireless router does not re-order packets arriving out of order at the wireless router as this will increase latency experienced by the flow.

The server should provide a sequence number that identifies the sequence of packets for a transport connection carrying the media flows.


## Host-Network Metadata {#host-network}

### Priority between Flows (Inter-flow) {#interflow-priority}

Certain flows being received by a host (or by an application on a host) are less or more important than other flows of *the same host*.
For example, a host downloading a software update is generally considered less important than another host doing interactive audio/video or gaming.
By signaling the relative importance of flows to a network element, the network element can (de-)prioritize those flows to best accommodate the needs of the various applications on a same host.

Without a signaling in place between a receiving host and its network, remote peers are able to mark packets that interfere with the desires of the receiving host -- making their flows more important than what the receiving host considers more important.
This eventually causes all flows to be marked as important, or -- more likely -- such priority markings to be ignored.


### Priority within a Flow (Intra-Flow) {#intra-flow-priority}

Interactive Audio/Video has long been using {{?RFC3550}} which runs over UDP.  As described in Section 2.3.7.2 of {{?RFC7478}}, there is value in differentiating between voice, video and data.
Today's video streaming is exclusively over TCP but will migrate to QUIC and eventually is likely to support unreliable transport ({{?RFC9221}}, {{!I-D.ietf-moq-transport}}).  With unreliable transport of video in RTP or QUIC, it is beneficial to differentiate the important video keyframes from other video frames.
Other applications such as gaming and remote desktop also benefit from differentiating their packets to the network.

Many of these flows do not originate from a content provider's network -- rather, they originate from a peer (e.g., VoIP, interactive video, peer-to-peer gaming, Remote Desktop).
Thus, the flows originate from an IP address that is not known before connection establishment, so there needs to be a way for the client to authorize the network elements to receive and hopefully to honor the metadata of those packets.

Without a signaling in place between a receiving host and its network, remote peers are able to mark every packet of a flow as important, causing much the same problem as the previous use-case.
Eventually, when all packets of every flow are marked as important, there is no differentiation between packets within a flow, rendering the network unable to improve reactive policy decisions.



# Non-Requirements {#non-req}

Application feedback with measurements of packets lost and delay incurred may affect the sending rate and other behavior of the application.
The requirements and specification to mitigate these aspects are not in the scope of this document.


# IANA Considerations {#iana}
None.

# Security Considerations {#sec-cons}

Security aspects for the metadata are discussed in {{privacy}} and {{key-establishment}}.
The principles outlined in {{?RFC8558}}, {{?RFC9049}} and {{?RFC9419}} contain security considerations and are referenced in {{metadata-req}}.

This document has no other security considerations.

# Acknowledgments
{:numbered="false"}

Xavier De Foy and the authors of this draft have discussed the similarities and differences of this draft with the MoQ draft for carrying media metadata.

The authors wish to thank Mike Heard, Sebastian Moeller and Tom Herbert for discussions on metadata fields, fragmentation and various transport aspects.

The authors appreciate input from Marcus Ilhar and Magnus Westerlund on the need to address privacy in general and Dan Druta to consider a common transport across various host to network signaling when possible.
Ruediger Geib suggested that limiting the amount of state information that a wireless router has to keep for a flow should be minimized.

Ingemar Johansson's suggestions on fast fading (which L4S handles) and dramatic drops in wireless accesses have been helpful to identify the issues.
Thanks to Hang Shi for the review and comments on host-to-network signaling.

--- back
