---
title: "Ordering of RRSets in DNS Message Sections"
category: std
updates: 1034,1035

docname: draft-jabley-dnsop-ordered-answer-section-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Operations and Management"
workgroup: "Domain Name System Operations"
keyword:
 - dns
 - dns protocol
 - dns message
venue:
  group: "Domain Name System Operations"
  type: "Working Group"
  mail: "dnsop@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/dnsop/"
  github: "ableyjoe/draft-jabley-dnsop-ordered-sections"
  latest: "https://ableyjoe.github.io/draft-jabley-dnsop-ordered-sections/draft-jabley-dnsop-ordered-sections.html"

author:
 -
    fullname: "Joe Abley"
    organization: Cloudflare
    email: "jabley@cloudflare.com"
 -
    fullname: "Sebastiaan Neuteboom"
    organization: Cloudflare
    email: "sebastiaan@cloudflare.com"

normative:

informative:
 Neuteboom2026:
   title: "What came first: the CNAME or the A record?"
   author:
     -
       ins: S. Neuteboom
       name: Sebastiaan Neuteboom
       org: Cloudflare
   date: 2026-01-14
   target: https://blog.cloudflare.com/cname-a-record-order-dns-standards/

 Cisco2026:
   title: "Cisco Business Switches Reboot with Fatal Error from DNSC Process"
   author:
     - org: Cisco
   date: 2026-01-08
   target: https://www.cisco.com/c/en/us/support/docs/smb/switches/Catalyst-switches/kmgmt3846-cbs-reboot-with-fatal-error-from-dnsc-process.html
   seriesInfo:
     "Cisco Document ID": "1767916364268164"

--- abstract

The existing Domain Name System (DNS) specifications lack some
clarity in their description of the process by which individual
sections of a DNS message are constructed.

This document updates RFC 1034 and RFC 1035 to provide a clearer
specification, consistent with deployed implementations.


--- middle

# Introduction

{{!RFC1034}} specifies an algorithm to follow when constructing
a response to a DNS QUERY.  This algorithm in some cases can result
in multiple RRSets being included in a single section of a DNS
message, e.g. when handling CNAME resource records.

Most consumers of DNS responses, such as stub resolvers, have interpreted the
direction to copy or store particular RRSets in sections of a DNS
response to mean "append", treating each section as an ordered list
of RRSets.  In particular, many stub stub resolvers are known to rely upon
that interpretation when processing DNS responses, e.g. see {{cloudflare}}.

Some DNS implementations employ algorithms in other sections that aim
to optimise processing of responses received by initiators, e.g.
NAPTR before SRV before A/AAAA in the additional section of a
response.  This behaviour has not been observed to cause any
interoperability problems, and is explicitly permitted by this
document.

This document updates {{!RFC1035}} to specify that the answer section in a
DNS message is an ordered list of RRSets, but that other sections may be
ordered differently. This document clarifies the directions
provided in {{!RFC1034}} to match the observed behaviour and
expectations of deployed software.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document assumes familiarity with terminology specific to the Domain Name System
(DNS) as described in {{!RFC9499}}.


# Updates to RFC 1034 {#rfc1034_updates}

{{!RFC1034}} specifies the algorithms by which sections of a DNS
response are constructed.  For example, step 3 of the
algorithm described in {{!RFC1034}} section 4.3.2 contains the direction
"copy all RRs which match QTYPE into answer section".

In this case, and in all other cases where {{!RFC1034}} specifies that
particular RRSets be included in the answer section of a DNS message, the
section MUST be treated as an ordered list of RRSets.  When it is
necessary to include new RRSets in a section of a DNS message that is
under construction, those RRSets MUST be appended. The receiver of a
DNS message MAY refuse to process DNS messages that have been constructed
differently.

When constructing other sections of a DNS message, each section MAY be
treated as a non-ordered list. A receiver of a DNS message MUST NOT
reject a DNS message on the basis of the order of RRSets in those sections.


# Updates to RFC 1035 {#rfc1035_updates}

In a DNS message, the answer section MUST be considered to be an ordered
set of RRSets. All other sections in a DNS message MUST be considered to
be a non-ordered set.

DNS implementations MUST construct each section in a DNS response
according to the algorithms specified in {{!RFC1034}}, as clarified in
{{rfc1034_updates}}.


# Security Considerations

The recommendations contained in this document have no known security
implications.


# IANA Considerations

This document has no IANA actions.


--- back

# Events of 8 January 2026 {#cloudflare}

Cloudflare operates a well-known public DNS resolver known as
1.1.1.1, after one of the IPv4 addresses associated with the service.
On 8 January a software change in the 1.1.1.1 service had the
unintential side-effect of changing the order in which RRSets were
encoded in the answer section of DNS responses, in the case where
constructing the responses involved CNAME processing. The previous
ordering was as clarified in {{rfc1034_updates}} and {{rfc1035_updates}}. The change in behaviour
was not detected by a corresponding failure in a regression test,
since the ordering in the answer section was not considered to be
significant.

Following the software release, Cloudflare became aware of significant
numbers of deployed DNS client implementations that were suffering
from failure. In particular, the getanswer_r() function invoked by
the getaddrinfo() function in glibc was found to fail to function,
and some deployed ethernet switches were observed to reboot when
trying to resolve the names of configured NTP servers {{Cisco2026}}.

The impact associated with this event was particularly widespread
because of the widespread use of the 1.1.1.1 resolver. However, the
two examples of client implementations are also widely deployed in
systems that may well be upgraded only infrequently (or never
upgraded at all).

See {{Neuteboom2026}} for additional information.

# Editorial Notes (remove before publication)

## draft-jabley-dnsop-ordered-sections-00

Initial draft circulated for comment in 2015; subsequently expired.

## draft-jabley-dnsop-ordered-answer-section-00

Draft revitalised following some operational excitement.

Added competent co-author.

# Acknowledgments
{:numbered="false"}

The contributions of Mark Andrews and Paul Vixie to the original
revision of this document are acknowledged.
