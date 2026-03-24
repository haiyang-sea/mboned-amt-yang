---
coding: utf-8

title: A YANG Data Model for Automatic Multicast Tunneling (AMT)
abbrev: YANG Data Model for AMT
docname: draft-ietf-mboned-amt-yang-latest
category: std

standalone: yes
submissiontype: IETF
pi: [toc, sortrefs, symrefs, comments]

ipr: trust200902
area: Ops
wg: MBONED Working Group
kw: YANG, AMT

author:
  -
    ins: Y. Liu
    fullname: Yisong Liu
    organization: China Mobile
    country: China
    email: liuyisong@chinamobile.com
  -
    ins: C. Lin
    fullname: Changwang Lin
    organization: New H3C Technologies
    country: China
    email: linchangwang.04414@h3c.com
  -
    ins: Z. Zhang
    fullname: Zheng(Sandy) Zhang
    organization: ZTE Corporation
    country: China
    email: zhang.zheng@zte.com.cn
  -
    ins: X. Geng
    fullname: Xuesong Geng
    organization: Huawei Technologies
    country: China
    email:  gengxuesong@huawei.com
  -
    ins: V. Kumar Nagaraj
    fullname: Vinod Kumar Nagaraj
    organization: Juniper Networks
    email:  vinkumar@juniper.net

normative:
  RFC2119:
  RFC3688:
  RFC6020:
  RFC7450:
  RFC7950:
  RFC8174:
  RFC8294:
  RFC8341:
  RFC8343:
  RFC8349:
  RFC8777:
  RFC9911:
    author:
    - name: Jürgen Schowalder
      role: editor
    title: Common YANG Data Types
    date: December 2025
    refcontent: RFC 9911, DOI 10.17487/RFC9911
    target: https://www.rfc-editor.org/info/rfc9911
informative:
  RFC4252:
  RFC6241:
  RFC7951:
  RFC8040:
  RFC8340:
  RFC8446:
  RFC9000:
  W3C.REC-xml-20081126:
    author:
    - name: Tim Bray
    - name: Jean Paoli
    - name: Michael Sperberg-McQueen
    - name: Eve Maler
    - name: François Yergeau
    title: Extensible Markup Language (XML) 1.0 (Fifth Edition)
    date: November 2008
    refcontent: World Wide Web Consortium Recommendation REC-xml-20081126
    target: https://www.w3.org/TR/2008/REC-xml-20081126
  I-D.ietf-netmod-rfc8407bis:
  I-D.ietf-netconf-udp-client-server:
--- abstract

   This document defines a YANG data model for the management of
   Automatic Multicast Tunneling (AMT) protocol operations.

--- middle

# Introduction

   {{RFC7450}} introduces the protocol definition of the Automatic
   Multicast Tunneling (AMT) for delivering multicast traffic from
   sources in a multicast-enabled network to receivers that lack
   multicast connectivity to the source network. AMT uses UDP
   encapsulation and unicast replication to provide this functionality.

   {{RFC8777}} updates {{RFC7450}} by modifying the relay discovery process.
   It defines DNS Reverse IP AMT Discovery (DRIAD) mechanism for AMT
   gateways to discover AMT relays that are capable of forwarding
   multicast traffic from a known source IP address.

   This document defines a YANG data model for managing AMT protocol.

   RFC Ed.: Please replace all occurrences of 'XXXX' with the
   actual RFC number (and remove this note). Also, please update
   the revision date to match the publication date.

# Terminology & Notation Conventions

## Conventions

   {::boilerplate bcp14-tagged}

## Terminology

   The terminology for describing YANG data models is found in {{RFC6020}}
   and {{RFC7950}}, including:

   *  augment

   *  data model

   *  data node

   *  identity

   *  module

   The following AMT terms are used in this document:

   *  Gateway: A functional entity that acts as an endpoint for
      AMT tunnels on the receiver's domain. Its primary role is to
      discover AMT Relays, establish AMT tunnels to them, receive
      multicast traffic over these tunnels, and then forward that
      traffic natively within its local domain.

   *  Relay: A functional entity located in the source's domain
      or a multicast-enabled part of the Internet. Its primary role is
      to listen for requests from AMT Gateways, replicate multicast
      traffic from native multicast routing domains, and encapsulate/
      forward this traffic through established AMT tunnels to requesting
      Gateways.

   *  Tunnel: A unidirectional, UDP-based encapsulation tunnel
      established between an AMT Gateway (the tunnel head) and an AMT
      Relay (the tunnel tail). It is used to transport multicast
      packets from the Relay to the Gateway over networks that do not
      support native IP multicast.

   *  Pseudo-Interface: A logical interface within the AMT Gateway or
      Relay that represents the endpoint of an AMT tunnel. Multicast
      routing protocols interact with this interface as if it were a
      physical interface receiving native multicast traffic.

   *  Gateway Service: The functional component in the AMT protocol
      architecture. It interacts downstream with local multicast
      receivers and upstream with AMT Relays. It implements the
      complete service functionality of the AMT protocol, including
      processing multicast subscription requests, establishing tunnels,
      decapsulating data, and forwarding multicast traffic.

   *  Relay Service: The central component in the AMT protocol
      architecture. It establishes and manages tunnels between
      multicast sources and AMT gateways, enabling the conversion and
      forwarding of multicast traffic from multicast networks to unicast
      networks.

   *  Secret Key Timeout: The maximum recommended validity period or
      rotation interval for the private secret (or key) used by a AMT
      Relay to compute Response Message Authentication Code (MAC) values,
      according to {{Section 5.3.6 of RFC7450}}.

## Tree Diagrams

   Tree diagrams used in this document follow the notation defined in
   {{RFC8340}}.

# Data Model Overview

   The AMT protocol mainly includes two components illustrated in
   {{fig-cpt}}. The two components are Relay and Gateway entities, each
   with their internal modules (Discovery, Tunnel Management, and Multicast
   Forwarding).

~~~~ aasvg
         +------------------------------------------------+
         |            AMT Protocol Components             |
         +------------------------------------------------+
         |  .-----------------.      .-----------------.  |
         |  |   AMT Relay     |<---->|   AMT Gateway   |  |
         |  |                 |      |                 |  |
         |  |  .-----------.  |      |  .-----------.  |  |
         |  |  | Discovery |  |      |  | Discovery |  |  |
         |  |  |  Service  |<-+------+--|  Service  |  |  |
         |  |  '-----+-----'  |      |  '-----+-----'  |  |
         |  |        |        |      |        |        |  |
         |  |  .-----+-----.  |      |  .-----+-----.  |  |
         |  |  |   Tunnel  |  |      |  |   Tunnel  |  |  |
         |  |  | Management|<-+------+--| Management|  |  |
         |  |  '-----+-----'  |      |  '-----+-----'  |  |
         |  |        |        |      |        |        |  |
         |  |  .-----+-----.  |      |  +-----+-----+  |  |
         |  |  | Multicast |  |      |  | Multicast |  |  |
         |  |  | Forwarding|  |      |  | Forwarding|  |  |
         |  |  '-----------'  |      |  +-----------+  |  |
         |  '-----------------'      '-----------------'  |
         +------------------------------------------------+
~~~~
{: #fig-cpt title="AMT Protocol Components"}

   The AMT data model provides methods for managing AMT protocol,
   covering all its core functional components as illustrated in
   Figure 1. It includes:

   *  Parameters of AMT relay service, such as Relay Discovery Address
      ({{Section 4.1.5 of RFC7450}}), Relay Address ({{Section 4.1.5 of RFC7450}}),
      the maximum number of tunnels, and secret key timeout.

   *  Parameters of AMT gateway service, such as Relay Discovery Address
      ({{Section 4.1.5 of RFC7450}}), Relay Address ({{Section 4.1.5 of RFC7450}}),
      Discovery Timeout ({{Section 5.2.2.4 of RFC7450}}),
      Request Timeout ({{Section 5.2.2.4 of RFC7450}}), and Maximum
      Retransmission Count ({{Section 5.2.2.4 of RFC7450}}).

   *  AMT tunnel information, such as endpoint IP address and UDP port
      number, local IP address and UDP port number.

   *  DNS Resource Record (RR) used by an AMT relay service.

# AMT YANG Module

## Prefixes

   {{tab-prefixes}} summarizes the prefixes used in this document.

| Prefix   | YANG module        | Reference   |
|----------|--------------------|-------------|
| inet     | ietf-inet-types    | {{RFC9911}} |
| rt-types | ietf-routing-types | {{RFC8294}} |
| rt       | ietf-routing       | {{RFC8349}} |
| yang     | ietf-yang-types    | {{RFC9911}} |
| if       | ietf-interfaces    | {{RFC8343}} |
{: #tab-prefixes title="Prefixes and Corresponding YANG Modules"}

## Tree View

   The full tree diagram of the "ietf-amt" YANG module is provided in
   {{full-tree}}. The following subsections list the subtree structures.

### Overall Structure

   The overall tree structure of the AMT YANG module is shown in
   {{fig-overall-tree}}.

   The AMT YANG module augments the core routing YANG module "ietf-
   routing" specified in {{RFC8349}}. Specifically, the AMT YANG module
   augments "/rt:routing/rt:control-plane-protocols".

~~~~ ascii-art
module: ietf-amt
  augment /rt:routing/rt:control-plane-protocols:
    +--rw amt!
       +--rw relay {amt-relay}?
       |  ...
       +--rw gateway {amt-gateway}?
          ...
~~~~
{: #fig-overall-tree title="Overall AMT Tree Structure"}

   The 'amt' container encapsulates all AMT functionality and serves as
   the primary entry point for its configuration and state. The 'amt'
   container consists of two functional components: the relay and the
   gateway. Support of relay or gateway is indicated by dedicated YANG features.

   The 'relay' container manages the AMT Relay function on the multicast
   source side. It provides the configuration and operational state for
   AMT Relay devices that receive multicast traffic, tunnel it to AMT
   Gateways over unicast, and act as tunnel termination points. This
   container is conditionally present only if the device implements the
   'amt-relay' feature, typically on service provider edge routers or data
   center gateways.

   The 'gateway' container manages the AMT Gateway function on the
   multicast receiver side. It provides the configuration and
   operational state for AMT Gateway devices that discover AMT Relays,
   establish tunnels to receive multicast traffic, and forward it to
   local receivers. This container is conditionally present only if the
   device implements the 'amt-gateway' feature, typically on enterprise
   edge routers or Customer Premises Equipment (CPE).

### Relay

   The overall structure of 'relay' is shown in {{fig-relay-subtree}}.

~~~~ ascii-art
module: ietf-amt
  augment /rt:routing/rt:control-plane-protocols:
    +--rw amt!
       +--rw relay {amt-relay}?
       |  +--rw addresses
       |  |  +--rw address* [family]
       |  |     +--rw family             identityref
       |  |     +--rw anycast-prefix     inet:ip-prefix
       |  |     +--rw local-address      inet:ip-address
       |  +--rw tunnel-limit?            uint32
       |  +--rw secret-key-timeout?      uint32
       |  +--rw relay-dns-resource-records
       |  |  +--rw record* [source-address]
       |  |     +--rw source-address         inet:ip-address
       |  |     +--rw precedence?            uint32
       |  |     +--rw d-bit?                 boolean
       |  |     +--rw relay-type?            enumeration
       |  |     +--rw discovery-address?     inet:ip-address
       |  |     +--rw domain-name?           inet:domain-name
       |  +--ro tunnels
       |  |  ...
       |  +--ro relay-message-statistics
       |     ...
       +--rw gateway {amt-gateway}?
          ...
~~~~
{: #fig-relay-subtree title="AMT Relay Subtree Structure"}

   The 'relay' data nodes are described as follows:

   'addresses':
   : Indicates the core address configurations for AMT Relay.
   : This data node includes 'family', 'anycast-prefix', and 'local-
   address'. The 'family' indicates the address family (IPv4 or IPv6).
   The 'anycast-prefix' indicates the address prefix used by the Gateway
   to discover the Relay. The 'local-address' indicates the local
   interface address the Relay actually listens on and sends AMT
   messages on, or the actual communication address after the tunnel is
   established.

   'tunnel-limit':
   : Indicates the maximum number of endpoint Gateways
   that a Relay can serve simultaneously.

   'secret-key-timeout':
   : Indicates the maximum recommended validity
   period or rotation interval for the private secret (or key) used by a
   AMT Relay to compute Response MAC values. In addition, the private
   secret (or key) is known only to the AMT relay, and the provisioning
   of the private secret (or key) is out of scope.

   'relay-dns-resource-records':
   : Indicates the DNS RR configuration for
   AMT relay discovery. Each DNS RR configuration ('record') includes
   the specific multicast source IP address to which this DNS RR applies
   ('source-address'), the priority value of this DNS RR ('precedence'),
   the discovery optional flag ('d-bit'), the type of AMT relay address
   ('relay-type'), the directly specified IP address of AMT relay
   discovery address ('discovery-address'), and the wire-encoded domain
   name of AMT relay ('domain-name').

   'tunnels' ({{fig-tunnel-subtree}}):
   : Indicates tunnel information from various AMT gateways
   connected to this AMT relay.
   : Each tunnel entry ('tunnel') includes
   the IP address and port number of the tunnel opposite end ('gateway')
   ('gateway-address' and 'gateway-port'), the local IP address and UDP port number
   used by the local ('relay') end for this tunnel ('local-address' and 'local-
   port'), the tunnel status ('state'), the multicast flow information
   ('multicast-flows') carried by the tunnel, the number of different
   multicast groups currently carried by this tunnel ('multicast-group-
   num'), the message counter carried by the tunnel ('request-message-
   count', 'membership-query-message-count', and 'membership-update-
   message-count'), and the time on the most recent occasion at which
   any one or more of the tunnel's counters suffered a discontinuity
   ('discontinuity-time').
   : Each multicast flow information ('flow') has
   multicast source address ('source-address') and multicast group
   address ('group-address').

   > Design note: The four data nodes ('gateway-address',
   > 'gateway-port', 'local-address', and 'local-port) do not reuse
   > the standard "udp-client" grouping defined in {{I-D.ietf-netconf-udp-client-server}}
   > because AMT requires the gateway to be a specific IP address (inet:ip-address),
   > while the "udp-client" grouping allows the use of domain names (inet:host).
   > Reuse could lead to configuration errors or runtime risks, so a custom structure
   > must be defined to enforce this constraint.

~~~~ ascii-art
module: ietf-amt
  augment /rt:routing/rt:control-plane-protocols:
    +--rw amt!
       +--rw relay {amt-relay}?
       |  ...
       |  +--ro tunnels
       |  |  +--ro tunnel* [gateway-address gateway-port]
       |  |     +--ro gateway-address     inet:ip-address
       |  |     +--ro gateway-port        inet:port-number
       |  |     +--ro local-address       inet:ip-address
       |  |     +--ro local-port          inet:port-number
       |  |     +--ro state               identityref
       |  |     +--ro multicast-flows
       |  |     |  +--ro flow* [source-address
       |  |     |     |         group-address]
       |  |     |     +--ro source-address
       |  |     |     |         ip-multicast-source-address
       |  |     |     +--ro group-address
       |  |     |               rt-types:ip-multicast-group-address
       |  |     +--ro multicast-group-num        yang:gauge32
       |  |     +--ro request-message-count
       |  |     |              yang:zero-based-counter64
       |  |     +--ro membership-query-message-count
       |  |     |              yang:zero-based-counter64
       |  |     +--ro membership-update-message-count
       |  |     |              yang:zero-based-counter64
       |  |     +--ro discontinuity-time           yang:date-and-time
       |  +--ro relay-message-statistics
       |     ...
       +--rw gateway {amt-gateway}?
          ...
~~~~
{: #fig-tunnel-subtree title="AMT Relay Tunnel Subtree Structure"}

   'relay-message-statistics':
   : Indicates various messages and error
   statistics handled by AMT relay as shown in {{fig-stats-subtree}}.

~~~~ ascii-art
  augment /rt:routing/rt:control-plane-protocols:
    +--rw amt!
       +--rw relay {amt-relay}?
       |  ...
       |  +--ro tunnels
       |  |  ...
       |  +--ro relay-message-statistics
       |     +--ro received
       |     |  +--ro relay-discovery       yang:zero-based-counter64
       |     |  +--ro request               yang:zero-based-counter64
       |     |  +--ro membership-update     yang:zero-based-counter64
       |     |  +--ro teardown              yang:zero-based-counter64
       |     +--ro sent
       |     |  +--ro relay-advertisement yang:zero-based-counter64
       |     |  +--ro membership-query      yang:zero-based-counter64
       |     +--ro error
       |     |  +--ro incomplete-packet     yang:zero-based-counter64
       |     |  +--ro invalid-mac           yang:zero-based-counter64
       |     |  +--ro unexpected-type       yang:zero-based-counter64
       |     |  +--ro invalid-relay-discovery-address
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro invalid-membership-request-address
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro invalid-membership-update-address
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro incomplete-relay-discovery-messages
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro incomplete-membership-request-messages
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro incomplete-membership-update-messages
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro no-active-gateway     yang:zero-based-counter64
       |     |  +--ro invalid-inner-header-checksum
       |     |  |                           yang:zero-based-counter64
       |     |  +--ro gateways-timed-out                 yang:gauge64
       |     +--ro discontinuity-time       yang:date-and-time
       +--rw gateway {amt-gateway}?
          ...
~~~~
{: #fig-stats-subtree title="AMT Relay Statistics Subtree Structure"}


### Gateway

   The structure of 'gateway' is shown in {{fig-gateway-subtree}}.

~~~~ ascii-art
module: ietf-amt
  augment /rt:routing/rt:control-plane-protocols:
    +--rw amt!
       +--rw relay {amt-relay}?
       |  ...
       +--rw gateway {amt-gateway}?
          +--rw pseudo-interfaces
          |  +--rw interface* [interface]
          |     +--rw name                      if:interface-ref
          |     +--rw discovery-method          identityref
          |     +--rw relay-discovery-address?  inet:ip-address
          |     +--rw relay-address?            inet:ip-address
          |     +--rw relay-port?               inet:port-number
          |     +--ro local-address?            inet:ip-address
          |     +--ro local-port?               inet:port-number
          |     +--rw upstream-interface?       if:interface-ref
          |     +--rw discovery-timeout?        uint32
          |     +--rw discovery-retrans-count?  uint32
          |     +--rw request-timeout?          uint32
          |     +--rw request-retrans-count?    uint32
          |     +--rw dest-unreach-retry-count? uint32
          |     +--ro tunnel-state              identityref
          |     +--ro relay-discovery-message-count
          |     |                      yang:zero-based-counter64
          |     +--ro relay-advertisement-message-count
          |     |                      yang:zero-based-counter64
          |     +--ro request-message-count
          |     |                      yang:zero-based-counter64
          |     +--ro membership-query-message-count
          |     |                      yang:zero-based-counter64
          |     +--ro membership-update-message-count
          |                            yang:zero-based-counter64
          +--ro gateway-message-statistics
             +--ro discontinuity-time         yang:date-and-time
             +--ro received
             |  +--ro relay-advertisement yang:zero-based-counter64
             |  +--ro membership-query yang:zero-based-counter64
             +--ro sent
                +--ro relay-discovery yang:zero-based-counter64
                +--ro request           yang:zero-based-counter64
                +--ro membership-update yang:zero-based-counter64
                +--ro teardown          yang:zero-based-counter64
~~~~
{: #fig-gateway-subtree title="AMT Gateway Subtree Structure"}

   The 'gateway' data nodes are described as follows:

   'pseudo-interfaces':
   : Indicates the configuration and operational
   state of pseudo interfaces used to establish AMT tunnels between
   gateways and relays.

   'gateway-message-statistics':
   : Indicates the message statistics of the
   AMT gateway. It has the time on the most recent occasion at which
   any one or more of the AMT gateway message counters suffered a discontinuity
   ('discontinuity-time'), the received message statistics of AMT gateway
   ('received'), and the sent message statistics of AMT gateway ('sent').
   : 'received' container includes the number of AMT relay advertisement
   messages received ('relay-advertisement') and the number of AMT
   membership query messages received ('membership-query').
   : 'sent' container includes the number of AMT relay discovery messages sent ('relay-
   discovery'), the number of AMT membership request messages sent
   ('request'), the number of AMT membership update messages sent
   ('membership-update'), and the number of AMT teardown messages sent
   ('teardown').

## YANG Module

   This document imports modules defined in {{RFC9911}}, {{RFC8294}},
   {{RFC8343}}, and {{RFC8349}}.

~~~~ yang
{::include-fold ./yang/ietf-amt@2026-03-10.yang}
~~~~
{: sourcecode-markers="true" sourcecode-name="ietf-amt@2026-03-10.yang"}

# Operational Considerations

   This document specifies a YANG data model for AMT that configures and monitors
   address parameters for both Relay and Gateway functions. Operators
   MUST monitor for address family mismatches between associated address parameters
   to ensure correct protocol operation, tunnel establishment, and forwarding behavior.

   The following address pairs and combinations are critical and MUST be validated
   for address family consistency:

   * On the AMT Relay:

     Within the 'relay/addresses/address' list entry indexed by a given address
     family ('family'):

     * The 'anycast-prefix' (discovery anycast prefix)
     * The 'local-address' (unicast IP address)

     These IP addresses MUST belong to the same address family indicated by the 'family'
     leaf (either both IPv4 or both IPv6). A mismatch (e.g., IPv4 'anycast-prefix' paired
     with IPv6 'local-address' under the same IPv4 'family' entry) indicates a configuration
     anomaly that can prevent Relay discovery, Advertisement responses, and tunnel setup.

   * On the AMT Gateway:

     Within each 'gateway/pseudo-interfaces/interface' entry:

     * The 'relay-discovery-address'
     * The 'relay-address'
     * The 'local-address' (operational state)

     These IP addresses MUST all belong to the same address family. A mismatch can lead to
     failure in Relay discovery, tunnel establishment, or traffic decapsulation.

   Network operators SHOULD implement configuration validation and operational monitoring
   to detect such address family mismatches. When detected, the device MUST log an appropriate
   error or alarm, and MAY prevent the inconsistent configuration from being applied.
   Corrective actions include reconfiguring the affected addresses to match the intended address
   family and verifying routing reachability for the configured addresses.

# Security Considerations

   This section is modeled after the template described in Section 3.7.1
   of {{I-D.ietf-netmod-rfc8407bis}}.

   The "ietf-amt" YANG module defines a data model that is designed to
   be accessed via YANG-based management protocols, such as Network Configuration Protocol (NETCONF)
   {{RFC6241}} and RESTCONF {{RFC8040}}. These YANG-based management
   protocols (1) have to use a secure transport layer (e.g., Secure Shell (SSH)
   {{RFC4252}}, TLS {{RFC8446}}, and QUIC {{RFC9000}}) and (2) have to use
   mutual authentication.

   The Network Configuration Access Control Model (NACM) {{RFC8341}}
   provides the means to restrict access for particular NETCONF or
   RESTCONF users to a preconfigured subset of all available NETCONF or
   RESTCONF protocol operations and content.

   There are a number of data nodes defined in this YANG module that are
   writable/creatable/deletable (i.e., config true, which is the
   default). All writable data nodes are likely to be sensitive or
   vulnerable in some network environments. Write operations (e.g.,
   edit-config) and delete operations to these data nodes without proper
   protection or authentication can have a negative effect on network
   operations. The following subtrees and data nodes have particular
   sensitivities/vulnerabilities:

      Under /rt:routing/rt:control-plane-protocols/rt:control-plane-
      protocol/: Unauthorized access to any data nodes in these subtrees
      can adversely affect the AMT subsystem of both the local device and
      the network. This may lead to network malfunctions, delivery of
      packets to inappropriate destinations, and other problems.

         amt/relay/addresses/address:
         : This subtree specifies the IPv4 or IPv6 address information
           for an AMT relay. Modifying the configuration may cause the
           AMT tunnel to be torn down or established.

         amt/relay/relay-dns-resource-records/record:
         : This subtree specifies the DNS RR configuration used to
           discover AMT relays. Modifying this configuration may cause
           the AMT gateway to discover new AMT relay devices, or fail to
           discover AMT relay devices.

         amt/gateway/pseudo-interfaces/interface:
         : This subtree specifies the parameters of AMT pseudo-interface
           for an AMT gateway. Modifying this configuration may cause the
           AMT gateway to establish or tear down tunnels with multiple
           AMT relays.

   Some of the readable data nodes in this YANG module may be considered
   sensitive or vulnerable in some network environments. It is thus
   important to control read access (e.g., via get, get-config, or
   notification) to these data nodes. Specifically, the following
   subtrees and data nodes have particular sensitivities/
   vulnerabilities:

       Under /rt:routing/rt:control-plane-protocols/rt:control-plane-
       protocol/: amt/relay and amt/gateway. Unauthorized access to any
       data nodes in these subtrees can disclose operational state
       information about the AMT relay or AMT gateway on this device.

# IANA Considerations

## IETF XML Registry

   IANA is requested to register the following URI in the "ns" registry
   within the "IETF XML Registry" group {{RFC3688}}:

~~~~ ascii-art
   URI:  urn:ietf:params:xml:ns:yang:ietf-amt
   Registrant Contact:  The IESG.
   XML:  N/A; the requested URI is an XML namespace.
~~~~

## YANG Module Names Registry

   IANA is requested to register the following YANG module in the "YANG
   Module Names" registry {{RFC6020}} within the "YANG Parameters"
   registry group:

~~~~ ascii-art
   Name:  ietf-amt
   Maintained by IANA?  N
   Namespace:  urn:ietf:params:xml:ns:yang:ietf-amt
   Prefix:  amt
   Reference:  RFC XXXX
~~~~

# Acknowledgments

Thanks to Mohamed Boucadair for review and comments.

--- back

# Full Tree {#full-tree}

~~~~ ascii-art
{::include-fold ./yang/trees/ietf-amt@2026-03-10.tree}
~~~~

# Data Model Example

   This section presents a simple and illustrative example of how to
   configure AMT.

   The example is represented in both XML {{W3C.REC-xml-20081126}} and
   JSON {{RFC7951}} formats.

   {{fig-example-xml}} shows a sample configuration for an AMT relay service in
   XML format. This example configures the protocol address family
   (IPv4 or IPv6), secret key timeout (120 minutes), and tunnel limit
   (10) for AMT relay function. In addition, the AMT anycast prefix is
   set to 192.0.2.1/32 for IPv4 and 2001:db8::1/128 for IPv6, and the
   AMT local address is configured to 198.51.100.42 for IPv4 and
   2001:db8:abcd:12::42 for IPv6.

~~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
  <routing xmlns="urn:ietf:params:xml:ns:yang:ietf-routing">
    <control-plane-protocols>
      <amt xmlns="urn:ietf:params:xml:ns:yang:ietf-amt">
        <relay>
          <addresses>
            <address>
              <family>ipv4</family>
              <anycast-prefix>192.0.2.1/32</anycast-prefix>
              <local-address>198.51.100.42</local-address>
            </address>
            <address>
              <family>ipv6</family>
              <anycast-prefix>2001:db8::1/128</anycast-prefix>
              <local-address>2001:db8:abcd:12::42</local-address>
            </address>
          </addresses>
          <tunnel-limit>10</tunnel-limit>
          <secret-key-timeout>120</secret-key-timeout>
        </relay>
      </amt>
    </control-plane-protocols>
  </routing>
</config>
~~~~
{: #fig-example-xml title="Data Model Example in XML"}

   {{fig-example-json}} shows the same sample configuration for an AMT relay
   service in JSON format.

~~~~ json
{
  "ietf-routing:routing": {
    "control-plane-protocols": {
      "ietf-amt:amt": {
        "relay": {
          "addresses": {
            "address": [
              {
                "family": "ipv4",
                "anycast-prefix": "192.0.2.1/32",
                "local-address": "198.51.100.42"
              },
              {
                "family": "ipv6",
                "anycast-prefix": "2001:db8::1/128",
                "local-address": "2001:db8:abcd:12::42"
              }
            ]
          },
          "tunnel-limit": 10,
          "secret-key-timeout": 120
        }
      }
    }
  }
}
~~~~
{: #fig-example-json title="Data Model Example in JSON"}
