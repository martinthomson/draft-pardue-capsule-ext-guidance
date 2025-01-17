---
title: "Guidance for HTTP Capsule Protocol Extensibility"
abbrev: "HTTP Capsule Extension Guidance"
category: std

docname: draft-pardue-capsule-ext-guidance-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "LPardue/draft-pardue-capsule-ext-guidance"
  latest: "https://LPardue.github.io/draft-pardue-capsule-ext-guidance/draft-pardue-capsule-ext-guidance.html"

author:
 -
    fullname: "Lucas Pardue"
    organization: Cloudflare
    email: "lucas@lucaspardue.com"

normative:

informative:
  RFC9110:
      display: HTTP
  RFC9112:
    display: HTTP/1.1
  RFC9113:
    display: HTTP/2
  RFC9114:
    display: HTTP/3

--- abstract

This document updates RFC 9297 with further guidance for extensibility of the
HTTP Capsule Protocol.


--- middle

# Introduction

The Capsule Protocol {{!CAPSULE=RFC9297}} is a sequence of type-length-value
tuples that definitions of new HTTP upgrade tokens can choose to use. It allows
endpoints to reliably communicate request-related information end-to-end on
HTTP request streams, even in the presence of HTTP intermediaries.

Clients can indicate in requests that they want the data stream to use the
Capsule Protocol by providing an upgrade token and/or a Capsule-Protocol header
field; see {{Section 3 of CAPSULE}}. Servers confirm Capsule Protocol usage by
returning a response in the 2xx (Successful) range, possibly including a
Capsule-Protocol header field.

The process of initiating the Capsule Protocol for any given data stream
identifies the purpose of usage and the Capsule types that endpoints can send or
receive.

This document updates RFC 9297 with further guidance for extensibility of the
Capsule Protocol.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# The Extensibility of Capsule Type Usage

In order to support extensibility, {{Section 3.2 of CAPSULE}} requires that:

> Endpoints that receive a Capsule with an unknown Capsule Type MUST silently
> drop that Capsule and skip over it to parse the next Capsule.

The first ambiguity in this text comes from the term "endpoint". It relates to
the resource that the Capsule Protocol negotiation applies to, not the general
HTTP endpoint. In other words, known or unknown types are scoped only to the
request stream Capsule Protocol usage, nothing else.

For example, proxying UDP in HTTP {{?UDP-PROXYING=RFC9298}} makes use of the
upgrade token "connect-udp" to enable the Connect Protocol. It describes how
DATAGRAM capsules ({{Section 3.5 of CAPSULE}}) can be used. Use of other capsule
types on that data stream is undefined, the expectation being that they are
ignored on receipt.

Similarly, proxying IP in HTTP {{?IP-PROXYING=RFC9484}} makes use of the upgrade
token "connect-ip" to enable the Connect Protocol. It defines ADDRESS_ASSIGN,
ADDRESS_REQUEST, and ROUTE_ADVERTISEMENT capsules ({{Section 4.7 of
IP-PROXYING}}) and describes how these can be used together with DATAGRAM
capsules.

An HTTP server could support both UDP and IP proxying and it's implementation
would be able to understand all four capsule types. However, use of
ADDRESS_ASSIGN, for example, on a "connect-udp" data stream is undefined by
{{UDP-PROXYING}}.

The {{CAPSULE}} document requires updated text to resolve this ambuguity. Should
this document be adopted, consensus on the resolution can be established.

## Negotiating Additional Capsule Type Usage

The second ambiguity with the text in {{Section 3.2 of CAPSULE}} comes from
ambiguity of intent. Silent dropping of unknown types new can be safely used by
extensions without prior arrangement or negotiation.

However, some extensions might be built on the assumption that a capsule is
processed by the recipient. For example to send a capsule that elicits some
response message or behavioural change. Such extensions can benefit from some
form of explicit negotiation.

There are several approaches to negotiating the use of new capsule types within
the scope of a request stream Capsule Protocol. This document does not mandate
any specific method but advises protocol designers to use negotiation patterns
that fit the end-to-end nature of the Capsule Protocol, where endpoints generate
and process capsules.

Specifically SETTINGS negotiation ({{Section 5.5 of RFC9113}} and {{Section 9 of
RFC9114}}) could be used to extend a connection, changing the scope of Capsule
Protocol knowledge for all request streams or a set of upgrade tokens. However,
the SETTINGS are not an end-to-end mechanism and therefore this method of
negotiation does not work when intermediaries are involved.

Negotiation of new capsule types via new upgrade tokens is an end-to-end
mechanism.  However, while HTTP/1.x clients can offer several token values in
the Upgrade mechanism ({{Section 7.8 of RFC9110}}), extended CONNECT
({{?EXT-CONNECT2=RFC8441}} and {{?EXTCONNECT3=RFC9220}}) does not support this
possibility. While HTTP/2 or HTTP/3 clients could use multiple separate requests
in order to attempt the selection of a most-preferred upgrade token, this
requires additional round trips which might introduce undesirable delays.

Header fields provide an end-to-end negotiation mechanism. The Capsule-Protocol
header field is itself extensible and parameters could, in theory, be used to
negotiate extensions. However, Capsule-Protocol requires that unknown parameters
are ignored, so extension designers ought to use an offer-echo pattern that
confirms the recipient did process the parameter. Also note that use of
Capsule-Protocol is optional and the upgrade tokens can mandate use of the
Capsule Protocol without this header field. In such cases, a new header field
can be defined to support extension negotiation.

# Generic CAPSULE_ERROR code for HTTP/2 and HTTP/3

{{Section 3.3 of CAPSULE}} describes error handling for all HTTP versions. It
defined the H3_DATAGRAM_ERROR code for HTTP/3, with the description "Datagram or
Capsule Protocol parse error". Subsequent work that relies on using the Capsule
protocol has identified that there are situations where using H3_DATAGRAM_ERROR
can be confusing, such as where DATAGRAM capsules are expressly not allowed, but
some other processing error related to capsules occurs. Furthermore, there is no
registered HTTP/2 error code to articulate capsule-related errors (instead, this
is delegated to HTTP/2 generic malformed handling defined in {{Section 8.1.1 of
RFC9113}}).

This document registers two generic error codes, CAPSULE_ERROR and
H3_CAPSULE_ERROR (values TBD), for HTTP/2 and HTTP/3 respectively. This can be
used as a general-purpose error for anything related to Capsule protocol
handling, when a more-specific error is not available or is not preferred.




# Security Considerations

The ability to send capsule types that the peer may not know, and is therefore
required to ignore, can be abused to cause a peer to expend additional
processing time. This could become a burden when used unnecessarily or to
excess.

An endpoint that does not monitor such behavior exposes itself to a risk of
denial-of-service attack. Implementations SHOULD track the use of unknown
capsule types and set limits on their use. An endpoint MAY treat activity that
is suspicious as a reason to close a connection, but false positives will result
in disrupting valid connections and requests. For guidance on closing
connections see {{Section 9.6 of RFC9112}}, {{Section 5.5 of RFC9113}}, and
{{Section 9 of RFC9114}}.


# IANA Considerations

TODO: register error codes


--- back

# Acknowledgments
{:numbered="false"}

David Schinazi and Tommy Pauly are capsule enthusiasts that suggested some ideas
leading to the genesis of this document
